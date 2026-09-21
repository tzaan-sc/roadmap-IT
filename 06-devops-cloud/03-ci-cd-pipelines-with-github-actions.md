# 03 - Tự Động Hóa CI/CD Với GitHub Actions (CI/CD Pipelines)

> **Mục tiêu bài học:** Chuyển đổi quy trình phát hành phần mềm từ việc "deploy thủ công bằng tay đầy rủi ro lúc nửa đêm" sang quy trình tự động hóa 100%: hiểu rõ bản chất của CI và CD, làm chủ cấu trúc workflow của GitHub Actions, quản lý bí mật an toàn với GitHub Secrets, và xây dựng một đường ống (Pipeline) tự động kiểm thử, đóng gói Docker và deploy lên máy chủ sản xuất.

---

## 1. Bản Chất Của CI/CD: Tại Sao Không Thể Thiếu?

Trước khi có CI/CD, quy trình phát hành phần mềm là một cơn ác mộng:
1. Lập trình viên viết xong code trên máy mình, nén file zip gửi cho quản trị viên (Ops).
2. Quản trị viên SSH vào server, gõ lệnh `git pull`, build code và restart ứng dụng thủ công.
3. Nếu code bị lỗi hoặc thiếu biến môi trường: Server sập, khách hàng phàn nàn, cả đội phải thức trắng đêm để tìm nguyên nhân và rollback!

```text
       QUY TRÌNH THỦ CÔNG CŨ                           QUY TRÌNH TỰ ĐỘNG HÓA CI/CD
┌──────────────────────────────────────┐       ┌──────────────────────────────────────┐
│ Dev commit code ──► Gửi file nén zip │       │ Dev chỉ cần gõ: git push origin main │
│ Ops nhận file ──► SSH vào server     │       │                │                     │
│ Ops tự build và restart app thủ công │       │                ▼ TỰ ĐỘNG HOÀN TOÀN:  │
│ ❌ Dễ sai sót con người, sập hệ thống!│       │ 1. Chạy Linter & Kiểm tra cú pháp    │
└──────────────────────────────────────┘       │ 2. Chạy toàn bộ bộ Unit Tests        │
                                               │ 3. Đóng gói Docker Image siêu nhẹ    │
                                               │ 4. Đẩy Image lên Registry & Deploy!  │
                                               │ ✅ Nhanh chóng, chuẩn xác, an toàn!  │
                                               └──────────────────────────────────────┘
```

---

## 2. Phân Biệt: CI vs CD

```text
┌───────────────────────────────────────┐
│     CONTINUOUS INTEGRATION (CI)       │
│  (Tự động hóa kiểm tra & tích hợp)    │
│  Code ──► Build ──► Lint ──► Test     │
└──────────────────┬────────────────────┘
                   │ Vượt qua toàn bộ bài kiểm tra!
                   ▼
┌───────────────────────────────────────┐
│        CONTINUOUS DELIVERY (CD)       │
│  Đóng gói bản phát hành sẵn sàng,     │
│  chờ con người bấm nút phê duyệt      │
│  ┌─────────────────────────────────┐  │
│  │     CONTINUOUS DEPLOYMENT (CD)  │  │
│  │  Tự động đẩy thẳng lên máy chủ  │  │
│  │  Production không cần bấm nút!  │  │
│  └─────────────────────────────────┘  │
└───────────────────────────────────────┘
```

- **CI (Continuous Integration - Tích hợp liên tục):** Đảm bảo mã nguồn mới từ nhiều lập trình viên khi gộp vào nhánh chính (`main`) không làm hỏng ứng dụng. Máy chủ tự động chạy kiểm thử (Automated Tests). Nếu test tạch (Fail), hệ thống lập tức chặn không cho merge!
- **CD (Continuous Delivery):** Tự động đóng gói ứng dụng (ví dụ tạo Docker Image hoặc file cài đặt) sẵn sàng để release. Việc triển khai thực tế chỉ cần 1 cú click chuột phê duyệt của trưởng nhóm.
- **CD (Continuous Deployment - Triển khai liên tục):** Mọi commit vượt qua toàn bộ các bước kiểm thử CI sẽ được tự động triển khai ngay lập tức lên môi trường thật (Production) mà không cần sự can thiệp thủ công của con người.

---

## 3. Kiến Trúc Của GitHub Actions

GitHub Actions là nền tảng CI/CD tích hợp sẵn ngay trong kho mã nguồn GitHub của bạn. Tất cả các đường ống tự động hóa được định nghĩa trong thư mục bắt buộc: **`.github/workflows/<tên-file>.yml`**.

```text
                         CẤU TRÚC GITHUB ACTIONS WORKFLOW
┌────────────────────────────────────────────────────────────────────────┐
│ WORKFLOW (Quy trình làm việc tổng thể)                                 │
│                                                                        │
│ 1. EVENT / TRIGGER (Sự kiện kích hoạt: on push nhánh main, pull_request)│
│                                                                        │
│ 2. JOBS (Các công việc lớn - Mặc định chạy song song hoặc tuần tự)     │
│    ┌──────────────────────────────────┐ ┌───────────────────────────┐  │
│    │ JOB 1: Test & Lint               │ │ JOB 2: Build & Deploy     │  │
│    │ Runner: ubuntu-latest (Máy ảo)   │ │ Runner: ubuntu-latest     │  │
│    │                                  │ │ (Chờ Job 1 pass mới chạy!)│  │
│    │  STEPS (Các bước nhỏ tuần tự):   │ │                           │  │
│    │  - Step 1: Checkout mã nguồn     │ │  STEPS:                   │  │
│    │  - Step 2: Cài đặt Node.js       │ │  - Step 1: Build Docker   │  │
│    │  - Step 3: npm run test          │ │  - Step 2: Deploy to VPS  │  │
│    └──────────────────────────────────┘ └───────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### 5 Thành phần cốt lõi:
1. **Workflow:** Tệp tin cấu hình YAML định nghĩa quy trình tự động.
2. **Events (Triggers):** Sự kiện kích hoạt chạy workflow (ví dụ: `push`, `pull_request`, `schedule` theo giờ).
3. **Jobs:** Nhóm các bước thực thi trên một máy ảo độc lập. Các job có thể phụ thuộc nhau bằng từ khóa `needs: [job_truoc]`.
4. **Runners:** Máy ảo do GitHub cấp phát miễn phí (`ubuntu-latest`, `windows-latest`) để chạy các lệnh của bạn.
5. **Steps & Actions:** Các bước lệnh cụ thể. Bạn có thể tự viết lệnh bash (`run: npm test`) hoặc tái sử dụng các **Actions** chuẩn từ GitHub Marketplace (`uses: actions/checkout@v4`).

---

## 4. Quản Lý Bảo Mật Với GitHub Secrets

> [!CAUTION]
> **Quy tắc an ninh mạng sống còn:**
> **TUYỆT ĐỐI KHÔNG BAO GIỜ** ghi mật khẩu, API Token, thông tin CSDL hay Private SSH Key trực tiếp vào mã nguồn hoặc file YAML. Hacker liên tục quét toàn bộ GitHub để đánh cắp các thông tin này chỉ trong vài giây!

### Cách thiết lập GitHub Secrets:
1. Truy cập Repository trên GitHub ──► Bấm tab **Settings**.
2. Chọn mục **Secrets and variables** ──► Bấm **Actions**.
3. Bấm **New repository secret** để thêm các biến bí mật (ví dụ: `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `SSH_PRIVATE_KEY`, `SERVER_IP`).
4. Trong file YAML, bạn truy cập các bí mật này thông qua cú pháp:
   ```yaml
   ${{ secrets.DOCKER_USERNAME }}
   ${{ secrets.SSH_PRIVATE_KEY }}
   ```
   GitHub sẽ tự động ẩn giấu (che dấu sao `***`) các giá trị này trong log hiển thị để đảm bảo an toàn tuyệt đối.

---

## 5. Xây Dựng Pipeline Hoàn Chỉnh Từ A Đến Z

Dưới đây là một file workflow mẫu chuẩn mực công nghiệp đặt tại `.github/workflows/ci-cd.yml`. Pipeline này sẽ tự động:
1. Kiểm tra mã nguồn và chạy toàn bộ Unit Tests.
2. Nếu test thành công: Đóng gói Docker Image và đẩy lên Docker Hub.
3. Tự động SSH vào máy chủ VPS để cập nhật phiên bản mới:

```yaml
name: CI/CD Production Pipeline

# 1. Chỉ chạy khi có commit được push vào nhánh main hoặc có Pull Request
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # -------------------------------------------------------------
  # GIAI ĐOẠN 1: KIỂM THỬ MÃ NGUỒN (CI)
  # -------------------------------------------------------------
  test:
    name: Run Automated Tests
    runs-on: ubuntu-latest

    steps:
      - name: 1. Tải mã nguồn về máy ảo runner
        uses: actions/checkout@v4

      - name: 2. Thiết lập môi trường Node.js v20
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm' # Tự động cache thư viện npm để chạy siêu tốc

      - name: 3. Cài đặt dependencies
        run: npm ci

      - name: 4. Chạy kiểm tra cú pháp và Unit Tests
        run: npm run test

  # -------------------------------------------------------------
  # GIAI ĐOẠN 2: BUILD DOCKER & TỰ ĐỘNG DEPLOY (CD)
  # -------------------------------------------------------------
  build-and-deploy:
    name: Build Docker Image & Deploy
    needs: test # BẮT BUỘC: Job 'test' phải PASS thành công thì job này mới được chạy!
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push' # Chỉ deploy khi push vào main

    steps:
      - name: 1. Checkout mã nguồn
        uses: actions/checkout@v4

      - name: 2. Đăng nhập vào Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: 3. Build và Push Docker Image lên Docker Hub
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/my-app:latest

      - name: 4. Tự động SSH vào máy chủ Linux để triển khai bản mới
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/my-app
            docker compose pull
            docker compose up -d --remove-orphans
            docker system prune -f
            echo "🚀 Ứng dụng đã được cập nhật thành công lên Production!"
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **CI** tự động hóa kiểm tra (Test & Lint) ngay khi có code mới; **CD** tự động hóa đóng gói và triển khai ứng dụng lên máy chủ.
2. File cấu hình GitHub Actions bắt buộc đặt tại thư mục **`.github/workflows/<tên-file>.yml`**.
3. Hiểu rõ các thành phần: **Trigger (`on:`)** ──► **Jobs (chạy trên Runners)** ──► **Steps (thực hiện Actions hoặc script bash)**.
4. Dùng từ khóa **`needs: [job_id]`** để quy định thứ tự các công việc (ví dụ: Test pass mới được phép Deploy).
5. Luôn lưu trữ khóa bảo mật, mật khẩu và API token trong **GitHub Secrets**, không bao giờ đưa vào mã nguồn Git công khai.
