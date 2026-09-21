# Level 6 — DevOps + Cloud (Vận Hành & Điện Toán Đám Mây)

> *"Một dòng code chưa được triển khai lên môi trường sản xuất (Production) một cách an toàn và tự động thì vẫn chỉ là một giả thuyết chưa được chứng minh."*

Chào mừng bạn đến với **Level 6** trong lộ trình [Roadmap IT](../README.md). Đây là cấp độ chuyển dịch từ một "người chỉ biết viết code ứng dụng" thành một "kỹ sư hoàn chỉnh" làm chủ toàn bộ chu trình sống của phần mềm: **Đóng gói container (Docker), điều phối đa dịch vụ (Docker Compose), tự động hóa tích hợp và triển khai liên tục (CI/CD với GitHub Actions), cùng với việc làm chủ hạ tầng đám mây toàn cầu (AWS, GCP, Azure)**.

---

## ♾️ Vòng Lặp Vô Tận DevOps (The DevOps Infinity Loop)

DevOps là sự kết hợp nhuần nhuyễn giữa **Phát triển (Development)** và **Vận hành hệ thống (Operations)** thành một chu trình khép kín tự động:

```text
                  ┌─────────────── PLAN (Lên kế hoạch) ──────────────┐
                  │                                                  │
                  ▼                                                  │
             [  CODE  ] (Viết mã nguồn)                              │
                  │                                                  │
                  ▼                                                  │
             [  BUILD ] (Đóng gói Docker Multi-stage)                │
                  │                                                  │
                  ▼                                                  │
             [  TEST  ] (Chạy tự động Unit / Integration Tests)      │
                  │                                                  │
                  ▼                                                  │
             [RELEASE ] (Tạo Docker Image đẩy lên Registry)          │
                  │                                                  │
                  ▼                                                  │
             [ DEPLOY ] (Tự động triển khai lên AWS Cloud / VPS)     │
                  │                                                  │
                  ▼                                                  │
             [OPERATE ] (Hệ thống phục vụ người dùng 24/7)           │
                  │                                                  │
                  ▼                                                  │
             [MONITOR ] (Thu thập logs, giám sát độ trễ & CPU) ──────┘
```

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là 5 chuyên đề chuyên sâu đã được biên soạn hoàn chỉnh trong thư mục này:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Docker & Ảo Hóa Container** | Vấn đề "Chạy trên máy tôi nhưng lỗi trên server", Bản chất Container vs Máy ảo VM (Linux Namespaces & Cgroups), Kiến trúc Docker (Image, Container, Daemon, Registry), Kỹ nghệ viết `Dockerfile` Multi-stage build giảm 90% dung lượng, Quản lý lưu trữ bền vững với **Docker Volumes**, Mạng Bridge Network. | [Xem bài viết](./01-docker-and-containerization.md) |
| **02** | **Điều Phối Với Docker Compose & K8s** | Quản lý hệ thống đa container bằng file `docker-compose.yml`, Khởi chạy toàn bộ hệ sinh thái chỉ bằng 1 lệnh duy nhất (`docker compose up -d`), Cơ chế **Service Discovery** qua DNS nội bộ Docker, Thiết lập `healthcheck` và thứ tự phụ thuộc, Nhập môn **Kubernetes (K8s)** và thời điểm nâng cấp lên cụm máy chủ. | [Xem bài viết](./02-docker-compose-and-orchestration.md) |
| **03** | **Tự Động Hóa CI/CD Với GitHub Actions** | Bản chất CI (Tích hợp liên tục) vs CD (Triển khai liên tục), Giải phẫu cấu trúc Workflow YAML (`.github/workflows`), Quản lý bí mật an toàn tuyệt đối với **GitHub Secrets**, Xây dựng đường ống Pipeline chuẩn: Tự động chạy Lint/Test ──► Đóng gói Docker ──► Tự động SSH vào máy chủ VPS để deploy bản mới. | [Xem bài viết](./03-ci-cd-pipelines-with-github-actions.md) |
| **04** | **Điện Toán Đám Mây: AWS, GCP & Azure** | Bản chất Cloud Computing (Pay-as-you-go & Elasticity), Phân biệt các mô hình: **IaaS, PaaS, SaaS, Serverless (AWS Lambda)**, Bộ 5 dịch vụ AWS cốt lõi nền móng: **EC2** (Máy chủ), **S3** (Lưu trữ tệp vô hạn 11 số 9), **RDS** (CSDL tự quản Multi-AZ), **VPC** (Mạng ảo bảo mật), **IAM** (Phân quyền Roles), So sánh AWS vs GCP vs Azure. | [Xem bài viết](./04-cloud-computing-aws-gcp-azure.md) |
| **05** | **Thực Hành Triển Khai & Phỏng Vấn** | **2 Bài Lab Thực Chiến:** Lab 1 đóng gói và điều phối hệ sinh thái Fullstack (API + Postgres + Redis + Nginx Reverse Proxy) bằng Docker Compose, Lab 2 thiết lập đường ống CI/CD GitHub Actions tự động hóa 100%. **Bộ 15 câu hỏi phỏng vấn kỹ thuật DevOps & Cloud cốt lõi** có lời giải chi tiết và Checklist tự đánh giá. | [Xem bài viết](./05-practice-deployment-and-interview.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 6

1. **Nói lời tạm biệt vĩnh viễn với lỗi môi trường:** Mọi ứng dụng của bạn đều được "container hóa" bằng Docker, chạy giống hệt nhau 100% trên máy phát triển và trên máy chủ sản xuất.
2. **Khởi chạy hệ thống phức tạp trong nháy mắt:** Không còn phải cài đặt thủ công hàng giờ đồng hồ, chỉ cần `docker compose up -d` là toàn bộ cơ sở dữ liệu, cache và web app đều hoạt động trơn tru.
3. **Tự động hóa quy trình phát hành (Zero-touch Deployment):** Chỉ cần `git push` là hệ thống CI/CD tự động kiểm thử, đóng gói và triển khai lên máy chủ mà không cần thao tác bằng tay.
4. **Làm chủ các khái niệm điện toán đám mây (Cloud):** Tự tin thiết kế kiến trúc hạ tầng an toàn trên AWS/GCP, biết cách tối ưu chi phí và mở rộng hệ thống theo lưu lượng người dùng.

---

## ⏭️ Bước Tiếp Theo
Sau khi đã làm chủ hạ tầng đám mây và quy trình triển khai tự động, bạn đã sẵn sàng bước lên đỉnh cao của kiến trúc hệ thống quy mô lớn:
👉 **[Level 7: System Design + Distributed Systems](../07-system-design-distributed-systems/README.md)**
