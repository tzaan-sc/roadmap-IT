# 05 - Thực Hành Triển Khai Thực Chiến & Bộ Câu Hỏi Phỏng Vấn DevOps / Cloud (Deployment & Interview)

> **Mục tiêu bài học:** Hiện thực hóa kiến thức về Docker, Docker Compose, CI/CD và Cloud thông qua 2 bài lab triển khai hệ thống chuẩn cấp độ sản xuất (Production-ready), cùng bộ 15 câu hỏi phỏng vấn kỹ thuật DevOps & Cloud cốt lõi có lời giải chi tiết và Checklist tự đánh giá năng lực.

---

## PHẦN 1: CÁC BÀI LAB THỰC CHIẾN

### LAB 1: Đóng Gói Toàn Diện Hệ Thống Đa Dịch Vụ Với Docker Compose & Nginx

#### Đề bài:
Xây dựng một hệ thống hoàn chỉnh gồm:
1. **API Server:** Ứng dụng Backend (Node.js hoặc Python).
2. **PostgreSQL Database:** Lưu trữ dữ liệu bền vững qua Volume.
3. **Redis Cache:** Tăng tốc độ truy xuất.
4. **Nginx Reverse Proxy:** Đóng vai trò là cửa ngõ duy nhất đón nhận request từ bên ngoài cổng 80 và chuyển tiếp (forward) vào Backend bên trong mạng nội bộ.

```text
INTERNET (Người dùng gọi cổng 80)
           │
           ▼
┌────────────────────────────────────────────────────────────────────────┐
│ DOCKER HOST (Hệ thống Docker Compose)                                 │
│                                                                        │
│   ┌────────────────────────────────┐                                   │
│   │ CONTAINER: NGINX REVERSE PROXY │                                   │
│   │ (Cổng 80 - Cửa ngõ duy nhất)   │                                   │
│   └───────────────┬────────────────┘                                   │
│                   │ Chuyển tiếp tới http://api_server:8080             │
│                   ▼                                                    │
│   ┌────────────────────────────────┐      ┌─────────────────────────┐  │
│   │ CONTAINER: API SERVER          │ ───► │ CONTAINER: REDIS_CACHE  │  │
│   │ (Chạy cổng nội bộ 8080)        │      │ (Cổng nội bộ 6379)      │  │
│   └───────────────┬────────────────┘      └─────────────────────────┘  │
│                   │ Đọc ghi dữ liệu                                    │
│                   ▼                                                    │
│   ┌────────────────────────────────┐                                   │
│   │ CONTAINER: POSTGRES_DB         │ ◄── Gắn Volume: postgres_data     │
│   │ (Cổng nội bộ 5432)             │     (Lưu bền vững trên ổ đĩa host)│
│   └────────────────────────────────┘                                   │
└────────────────────────────────────────────────────────────────────────┘
```

#### Bước 1: Tạo cấu hình Nginx (`nginx/nginx.conf`)
```nginx
events { worker_connections 1024; }

http {
    server {
        listen 80;

        location / {
            # Chuyển tiếp toàn bộ request tới container có tên 'api_server'
            proxy_pass http://api_server:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

#### Bước 2: Tạo file `docker-compose.yml` hoàn chỉnh
```yaml
services:
  nginx_proxy:
    image: nginx:alpine
    container_name: production_nginx
    restart: always
    ports:
      - "80:80" # Chỉ mở duy nhất cổng 80 ra ngoài Internet!
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api_server
    networks:
      - internal_network

  api_server:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: production_api
    restart: always
    # KHÔNG CẦN mở ports ra ngoài host! Nginx sẽ nói chuyện nội bộ qua port 8080!
    expose:
      - "8080"
    environment:
      DATABASE_URL: "postgresql://postgres:secret123@postgres_db:5432/app_db"
      REDIS_URL: "redis://redis_cache:6379"
    depends_on:
      postgres_db:
        condition: service_healthy
      redis_cache:
        condition: service_started
    networks:
      - internal_network

  postgres_db:
    image: postgres:16-alpine
    container_name: production_db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret123
      POSTGRES_DB: app_db
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d app_db"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - internal_network

  redis_cache:
    image: redis:7-alpine
    container_name: production_redis
    restart: always
    networks:
      - internal_network

volumes:
  db_data:

networks:
  internal_network:
    driver: bridge
```

#### Bước 3: Khởi chạy và kiểm tra
```bash
# Khởi chạy toàn bộ hệ thống
docker compose up -d

# Kiểm tra tất cả các container đều xanh
docker compose ps

# Gửi request kiểm tra tới Nginx cổng 80
curl http://localhost/api/health
```

---

### LAB 2: Thiết Lập Pipeline Tự Động CI/CD Với GitHub Actions

#### Đề bài:
Thiết lập đường ống tự động hóa hoàn toàn:
1. Khi có bất kỳ commit nào được push lên nhánh `main`, GitHub Actions tự động kiểm thử.
2. Nếu test pass, tự động build Docker Image đa tầng (Multi-stage) và đẩy lên **Docker Hub**.
3. Tự động dùng SSH kết nối vào máy chủ Cloud (VPS Ubuntu), tải Image mới về và khởi động lại container không gián đoạn.

#### File cấu hình `.github/workflows/deploy.yml`:
```yaml
name: Continuous Deployment Pipeline

on:
  push:
    branches: [ main ]

jobs:
  test-and-build:
    name: Build, Test & Push to Docker Hub
    runs-on: ubuntu-latest

    steps:
      - name: 1. Tải mã nguồn về máy ảo runner
        uses: actions/checkout@v4

      - name: 2. Thiết lập môi trường Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: 3. Chạy Unit Test
        run: |
          npm ci
          npm run test

      - name: 4. Đăng nhập vào Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: 5. Build và Push Docker Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/shop-api:latest

  deploy-to-vps:
    name: Deploy to Cloud Server
    needs: test-and-build
    runs-on: ubuntu-latest

    steps:
      - name: SSH vào máy chủ và triển khai phiên bản mới
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/shop-project
            # 1. Đăng nhập Docker Hub trên server
            echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
            # 2. Tải bản image mới nhất vừa build
            docker compose pull
            # 3. Khởi động lại ứng dụng
            docker compose up -d --remove-orphans
            # 4. Dọn dẹp image cũ
            docker image prune -f
            echo "✅ Cập nhật ứng dụng thành công lúc: $(date)"
```

---

## PHẦN 2: BỘ 15 CÂU HỎI PHỎNG VẤN DEVOPS & CLOUD CỐT LÕI (CÓ LỜI GIẢI)

### Câu 1: Phân biệt sự khác nhau giữa Container (Docker) và Máy ảo (Virtual Machine)?
- **Trả lời:**
  - **Virtual Machine (VM):** Sử dụng một phần mềm Hypervisor để giả lập toàn bộ phần cứng ảo và **chạy một hệ điều hành khách riêng biệt (Guest OS)**. VM rất nặng (vài GB), tốn nhiều tài nguyên RAM cố định và mất từ 1-3 phút để khởi động.
  - **Docker Container:** Chạy trực tiếp trên máy chủ và **chia sẻ chung Nhân Kernel của hệ điều hành Linux**. Container được cô lập nhờ 2 tính năng của Linux Kernel là **Namespaces** (cô lập tiến trình, mạng, thư mục) và **Cgroups** (giới hạn CPU, RAM). Container siêu nhẹ (vài chục MB) và khởi động tức thì trong vài phần trăm giây.

### Câu 2: Kỹ thuật Multi-stage build trong Dockerfile là gì và mang lại lợi ích gì?
- **Trả lời:**
  - Multi-stage build cho phép chia `Dockerfile` thành nhiều giai đoạn độc lập bằng các câu lệnh `FROM ... AS <stage_name>`.
  - **Lợi ích:** Ta có thể dùng một Image đầy đủ công cụ biên dịch nặng hàng Gigabytes ở giai đoạn đầu (Build Stage) để compile mã nguồn, sau đó ở giai đoạn cuối (Runtime Stage), ta chỉ dùng một Image tối giản siêu nhẹ (như `alpine` hoặc `scratch`) và **chỉ copy các file thành phẩm sang**. Giúp giảm kích thước Image cuối cùng từ hàng GB xuống còn vài chục MB, rút ngắn thời gian deploy và tăng cường bảo mật (loại bỏ trình biên dịch khỏi môi trường production).

### Câu 3: Tại sao cần sử dụng Docker Volumes? Phân biệt Volume và Bind Mount?
- **Trả lời:**
  - Mặc định, container có tính chất **Ephemeral (phù du)**: khi container bị xóa, toàn bộ dữ liệu sinh ra bên trong nó sẽ biến mất vĩnh viễn. Docker Volumes giúp lưu trữ dữ liệu bền vững trên ổ đĩa máy chủ host bên ngoài vòng đời của container.
  - **Docker Volume:** Được lưu trữ tại một thư mục riêng do Docker toàn quyền quản lý trên máy chủ (`/var/lib/docker/volumes/`). Đây là lựa chọn chuẩn mực cho Database trong môi trường Production vì an toàn và có hiệu năng I/O cao.
  - **Bind Mount:** Ánh xạ trực tiếp một đường dẫn thư mục bất kỳ trên máy host vào bên trong container (ví dụ: `./src:/app/src`). Thường dùng trong môi trường phát triển (Local Development) để hỗ trợ tính năng Live-reload (sửa code ngoài máy host thì container cập nhật ngay).

### Câu 4: Làm thế nào để hai container trong cùng một file `docker-compose.yml` có thể giao tiếp được với nhau?
- **Trả lời:**
  - Khi khởi chạy với Docker Compose, Docker sẽ tự động tạo một mạng ảo dạng **Bridge Network** và đưa tất cả các container vào mạng này.
  - Docker tích hợp sẵn một máy chủ **Embedded DNS** tại IP ảo `127.0.0.11`. Nhờ cơ chế **Service Discovery**, các container có thể giao tiếp trực tiếp với nhau bằng chính **tên của service** được định nghĩa trong file YAML (ví dụ `http://api_server:8080` hoặc `postgres_db:5432`) mà không cần biết địa chỉ IP cụ thể của container.

### Câu 5: Sự khác nhau giữa lệnh `CMD` và `ENTRYPOINT` trong Dockerfile là gì?
- **Trả lời:**
  - **`ENTRYPOINT`:** Định nghĩa lệnh thực thi chính cố định của container. Nó không thể bị ghi đè một cách dễ dàng khi người dùng chạy `docker run` (trừ khi dùng cờ `--entrypoint`).
  - **`CMD`:** Cung cấp các tham số mặc định cho `ENTRYPOINT` (hoặc lệnh thực thi mặc định nếu không có `ENTRYPOINT`). Các giá trị trong `CMD` sẽ bị ghi đè hoàn toàn nếu người dùng truyền thêm tham số lúc chạy `docker run`.

### Câu 6: Phân biệt sự khác biệt giữa Continuous Delivery và Continuous Deployment?
- **Trả lời:**
  - **Continuous Delivery:** Mã nguồn sau khi vượt qua tất cả các bài kiểm tra tự động (CI) sẽ được đóng gói sẵn sàng thành bản phát hành (Release artifact/Docker image). Tuy nhiên, bước triển khai thực tế lên môi trường Production vẫn cần **một cú click chuột phê duyệt thủ công** của con người (Tech Lead/Release Manager).
  - **Continuous Deployment:** Tự động hóa 100% từ đầu đến cuối. Bất kỳ commit nào vượt qua thành công toàn bộ bài test CI sẽ **tự động được deploy thẳng lên máy chủ Production** ngay lập tức mà không cần bất kỳ sự can thiệp hay phê duyệt thủ công nào.

### Câu 7: Tại sao tuyệt đối không được commit file `.env` hoặc API token vào GitHub? GitHub Secrets hoạt động thế nào?
- **Trả lời:**
  - Nếu commit file `.env` chứa mật khẩu hay API Key lên GitHub, các bot độc hại của hacker liên tục quét mã nguồn công khai để đánh cắp thông tin, gây thiệt hại tài chính nghiêm trọng hoặc làm lộ dữ liệu khách hàng.
  - **GitHub Secrets:** Cho phép mã hóa và lưu trữ an toàn các biến môi trường nhạy cảm trong hệ thống của GitHub. Trong quá trình chạy workflow CI/CD, chỉ có máy ảo Runner mới được giải mã bí mật để sử dụng, và GitHub sẽ tự động che dấu sao (`***`) mọi giá trị bí mật trong log đầu ra để chống rò rỉ.

### Câu 8: Phân biệt 3 mô hình dịch vụ đám mây: IaaS, PaaS, và SaaS?
- **Trả lời:**
  - **IaaS (Infrastructure as a Service):** Cho thuê hạ tầng thô (CPU, RAM, Ổ cứng, Mạng). Khách hàng tự cài đặt và quản trị hệ điều hành cùng toàn bộ phần mềm (Ví dụ: AWS EC2, Google Compute Engine).
  - **PaaS (Platform as a Service):** Nhà cung cấp lo toàn bộ phần cứng, hệ điều hành và môi trường chạy runtime. Khách hàng chỉ việc đưa mã nguồn hoặc container lên để chạy (Ví dụ: Vercel, Render, Heroku).
  - **SaaS (Software as a Service):** Ứng dụng phần mềm hoàn chỉnh cho người dùng cuối sử dụng qua trình duyệt web (Ví dụ: Google Drive, Slack, Office 365).

### Câu 9: Serverless (FaaS - AWS Lambda) là gì? Hiện tượng "Cold Start" là gì?
- **Trả lời:**
  - **Serverless (FaaS):** Mô hình cho phép lập trình viên chỉ viết mã hàm logic xử lý mà không cần quản trị bất kỳ máy chủ nào. Hệ thống chỉ cấp phát tài nguyên khi có sự kiện/request gửi tới, và tính tiền chính xác theo từng mili-giây hàm chạy.
  - **Cold Start (Khởi động lạnh):** Khi một hàm Serverless đã lâu không được gọi hoặc khi lưu lượng truy cập tăng đột biến đòi hỏi tạo thêm bản sao mới, nhà cung cấp Cloud phải khởi tạo một container mới và nạp runtime từ đầu. Quá trình này gây ra độ trễ (delay) từ vài trăm mili-giây đến vài giây cho lần request đầu tiên đó.

### Câu 10: Amazon S3 là gì? Tại sao không nên lưu ảnh đại diện của người dùng trên ổ cứng của server EC2?
- **Trả lời:**
  - **Amazon S3** là dịch vụ lưu trữ đối tượng tĩnh (Object Storage) không giới hạn dung lượng với độ bền cam kết 99.999999999% (11 số 9).
  - **Không nên lưu ảnh trên ổ cứng EC2 vì:**
    1. Ổ cứng EC2 có dung lượng hữu hạn, lưu nhiều ảnh sẽ làm đầy ổ đĩa khiến máy chủ bị sập.
    2. Nếu máy chủ EC2 bị hỏng hoặc khi áp dụng Auto-scaling (tự động bật tắt thêm máy ảo mới), các máy ảo mới sẽ không có các file ảnh này!
    3. Lưu trên S3 kết hợp CDN (CloudFront) giúp người dùng trên toàn cầu tải ảnh với tốc độ tối đa mà không tốn băng thông và CPU của máy chủ chính.

### Câu 11: Multi-AZ trong Amazon RDS có ý nghĩa gì đối với tính sẵn sàng cao (High Availability)?
- **Trả lời:**
  - Multi-AZ (Multiple Availability Zones) là tính năng tự động tạo và duy trì một bản sao dự phòng đồng bộ (Synchronous Standby Replica) của Cơ sở dữ liệu tại một trung tâm dữ liệu (Availability Zone) độc lập khác về mặt địa lý.
  - Nếu trung tâm dữ liệu chính gặp thảm họa (mất điện toàn vùng, lũ lụt, hỏng hóc phần cứng), RDS sẽ tự động kích hoạt cơ chế chuyển đổi dự phòng (**Failover**) sang bản sao chỉ trong vòng 60 đến 120 giây mà không cần can thiệp thủ công và không làm thay đổi chuỗi kết nối (Endpoint) của ứng dụng.

### Câu 12: IAM Role trong AWS khác gì so với IAM User? Tại sao nên dùng IAM Role cho máy chủ EC2?
- **Trả lời:**
  - **IAM User:** Đại diện cho một con người cụ thể, có tên đăng nhập, mật khẩu và cặp khóa API (`AccessKeyId` / `SecretAccessKey`) có thời hạn vĩnh viễn.
  - **IAM Role:** Là một danh tính bảo mật được gán cho một thực thể (như máy chủ EC2 hoặc hàm Lambda) với các quyền hạn cụ thể, sử dụng **khóa xác thực tạm thời tự động xoay vòng (Temporary Credentials)**.
  - **Nên dùng IAM Role cho EC2 vì:** Bạn hoàn toàn không cần phải lưu hardcode bất kỳ Access Key nào trong file code hay file `.env` trên server. Kể cả khi hacker đột nhập vào đọc mã nguồn của máy chủ, chúng cũng không thể đánh cắp được cặp khóa vĩnh viễn của tài khoản AWS.

### Câu 13: Blue/Green Deployment và Rolling Update là gì? Chúng giúp giảm Downtime như thế nào?
- **Trả lời:**
  - **Rolling Update:** Cập nhật dần dần từng server một. Ví dụ có 4 server: tắt server 1 để cập nhật bản mới trong khi 3 server kia vẫn phục vụ khách, sau đó lặp lại cho tới hết. Người dùng không bao giờ thấy trang web bị gián đoạn (Zero-downtime).
  - **Blue/Green Deployment:** Duy trì 2 môi trường giống hệt nhau: **Blue** (đang chạy phiên bản hiện tại) và **Green** (cài sẵn phiên bản mới hoàn chỉnh và đã kiểm thử xong). Khi phát hành, bộ cân bằng tải (Load Balancer) chỉ cần chuyển hướng 100% lưu lượng từ Blue sang Green trong 1 giây. Nếu có lỗi phát sinh, hệ thống có thể chuyển ngược lại Blue ngay lập tức (**Instant Rollback**).

### Câu 14: Kubernetes (K8s) giải quyết bài toán gì mà Docker Compose không thể giải quyết được?
- **Trả lời:**
  - Docker Compose chỉ chạy trên **một máy chủ vật lý duy nhất (Single-node)**, không có khả năng tự phục hồi khi máy chủ đó chết phần cứng, và không tự động tăng giảm số lượng container theo lượng truy cập thực tế.
  - Kubernetes là hệ thống điều phối container trên **cụm hàng trăm máy chủ (Cluster Orchestration)**. K8s tự động dời container từ máy hỏng sang máy khỏe (Self-healing), tự động nhân bản số lượng Pods khi CPU tăng cao (Horizontal Pod Auto-scaler - HPA), và quản lý định tuyến mạng phức tạp ở quy mô hàng triệu người dùng.

### Câu 15: Làm thế nào để dọn dẹp dung lượng ổ cứng bị đầy do Docker sinh ra trên máy chủ Linux?
- **Trả lời:**
  - Docker thường tích lũy các Image cũ, Container đã dừng và các lớp đệm build cache làm đầy ổ cứng. Lệnh dọn dẹp thần thánh:
  ```bash
  # Xóa sạch các container đã dừng, mạng ảo thừa và toàn bộ image rác không sử dụng:
  docker system prune -a

  # Nếu muốn xóa cả các volume rác không gắn vào container nào (CẨN TRỌNG với dữ liệu CSDL):
  docker system prune -a --volumes
  ```

---

## PHẦN 3: CHECKLIST TỰ ĐÁNH GIÁ NĂNG LỰC (LEVEL 6 COMPETENCY)

Hãy tự kiểm tra xem bạn đã thành thạo các kỹ năng của Level 6 chưa:

- [ ] Tôi hiểu rõ sự khác biệt giữa Container và Máy ảo (VM).
- [ ] Tôi tự tay viết được `Dockerfile` chuẩn công nghiệp áp dụng kỹ thuật Multi-stage build.
- [ ] Tôi biết cách dùng Docker Volumes để bảo vệ dữ liệu CSDL không bị mất khi xóa container.
- [ ] Tôi sử dụng thành thạo Docker Compose để khởi chạy hệ thống đa dịch vụ và hiểu cơ chế DNS Service Discovery.
- [ ] Tôi hiểu rõ sự khác biệt giữa CI (Continuous Integration) và CD (Continuous Delivery/Deployment).
- [ ] Tôi tự thiết lập được một pipeline GitHub Actions tự động kiểm thử và đóng gói Docker Image.
- [ ] Tôi biết cách quản lý thông tin bảo mật an toàn bằng GitHub Secrets.
- [ ] Tôi phân biệt được các mô hình đám mây IaaS, PaaS, và Serverless.
- [ ] Tôi hiểu vai trò của 5 dịch vụ AWS cốt lõi: EC2, S3, RDS, IAM, và VPC.
- [ ] Tôi biết cách dùng lệnh `docker system prune` để giải phóng dung lượng đĩa cho máy chủ.
