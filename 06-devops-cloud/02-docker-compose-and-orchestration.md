# 02 - Điều Phối Đa Container Với Docker Compose & Nhập Môn Kubernetes (Docker Compose & Orchestration)

> **Mục tiêu bài học:** Giải quyết bài toán vận hành hệ thống đa dịch vụ phức tạp (Frontend + Backend API + PostgreSQL + Redis + Nginx) chỉ bằng một câu lệnh duy nhất với Docker Compose, làm chủ cơ chế Service Discovery qua DNS nội bộ, và hiểu rõ thời điểm vàng để nâng cấp từ Docker Compose lên hệ thống điều phối cụm quy mô lớn: Kubernetes (K8s).

---

## 1. Tại Sao Cần Docker Compose?

Hãy tưởng tượng bạn phát triển một ứng dụng thương mại điện tử thực tế gồm 4 thành phần:
1. **Frontend:** React / Next.js (chạy cổng 3000).
2. **Backend API:** Node.js / FastAPI (chạy cổng 8080).
3. **Database:** PostgreSQL (chạy cổng 5432).
4. **Cache:** Redis (chạy cổng 6379).

Nếu sử dụng các câu lệnh Docker CLI thông thường, bạn sẽ phải gõ lần lượt 4 câu lệnh dài dằng dặc:
```bash
# Gõ thủ công rất dễ sai sót và mất thời gian:
docker run -d --name my-redis -p 6379:6379 redis:alpine
docker run -d --name my-postgres -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data -p 5432:5432 postgres:16
docker run -d --name my-backend -p 8080:8080 --link my-postgres --link my-redis my-backend-img
docker run -d --name my-frontend -p 3000:3000 my-frontend-img
```
Khi bạn tuyển thêm một thành viên mới vào nhóm, người đó sẽ mất cả ngày chỉ để thiết lập môi trường chạy thử trên máy của họ!

**Docker Compose** sinh ra để giải quyết triệt để vấn đề này: Bạn gom toàn bộ định nghĩa cấu hình của cả 4 dịch vụ vào **duy nhất một file `docker-compose.yml`**. Bất kỳ ai trong team chỉ cần gõ đúng một câu lệnh:
```bash
docker compose up -d
```
Chỉ sau 30 giây, toàn bộ hệ sinh thái của dự án sẽ tự động được tải về, build, tạo mạng, gắn volume và khởi chạy trơn tru!

---

## 2. Giải Phẫu Cấu Trúc File `docker-compose.yml`

File cấu hình được viết bằng định dạng **YAML** với các khối cấu trúc rõ ràng:

```yaml
services:
  # -------------------------------------------------------------
  # DỊCH VỤ 1: CƠ SỞ DỮ LIỆU POSTGRESQL
  # -------------------------------------------------------------
  postgres_db:
    image: postgres:16-alpine # Sử dụng image chính thức từ Docker Hub
    container_name: ecom_postgres
    restart: always # Tự khởi động lại nếu bị crash
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secretpassword123
      POSTGRES_DB: ecommerce_db
    ports:
      - "5432:5432" # Ánh xạ cổng: "Port_Máy_Thật:Port_Container"
    volumes:
      - postgres_data:/var/lib/postgresql/data # Gắn volume lưu dữ liệu bền vững
    networks:
      - backend_network
    healthcheck: # Kiểm tra xem DB đã thực sự sẵn sàng nhận kết nối chưa
      test: ["CMD-SHELL", "pg_isready -U admin -d ecommerce_db"]
      interval: 5s
      timeout: 5s
      retries: 5

  # -------------------------------------------------------------
  # DỊCH VỤ 2: BỘ NHỚ ĐỆM REDIS
  # -------------------------------------------------------------
  redis_cache:
    image: redis:7-alpine
    container_name: ecom_redis
    restart: always
    ports:
      - "6379:6379"
    networks:
      - backend_network

  # -------------------------------------------------------------
  # DỊCH VỤ 3: BACKEND API
  # -------------------------------------------------------------
  api_server:
    build:
      context: ./backend # Tự động build Dockerfile nằm trong thư mục ./backend
      dockerfile: Dockerfile
    container_name: ecom_api
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      PORT: 8080
      # PHÉP MÀU SERVICE DISCOVERY:
      # Thay vì dùng IP 'localhost', ta dùng chính tên service 'postgres_db' và 'redis_cache'!
      DATABASE_URL: "postgresql://admin:secretpassword123@postgres_db:5432/ecommerce_db"
      REDIS_URL: "redis://redis_cache:6379"
    depends_on:
      postgres_db:
        condition: service_healthy # Đợi Database vượt qua healthcheck xong mới khởi động API!
      redis_cache:
        condition: service_started
    networks:
      - backend_network

# -------------------------------------------------------------
# KHAI BÁO CÁC NGUYÊN TÀI NGUYÊN CHUNG
# -------------------------------------------------------------
volumes:
  postgres_data: # Tạo volume tên postgres_data

networks:
  backend_network: # Tạo mạng nội bộ Bridge kết nối 3 container
    driver: bridge
```

---

## 3. Bản Chất Cơ Chế Service Discovery Qua DNS Nội Bộ

Tại sao trong cấu hình Backend ở trên, ta không viết `localhost:5432` mà lại viết `@postgres_db:5432`?

```text
MÁY CHỦ VẬT LÝ HOST (127.0.0.1)
┌────────────────────────────────────────────────────────────────────────┐
│ DOCKER BRIDGE NETWORK: backend_network                                 │
│                                                                        │
│   ┌──────────────────────┐              ┌──────────────────────┐       │
│   │ CONTAINER: api_server│              │CONTAINER: postgres_db│       │
│   │                      │ ───────────► │                      │       │
│   │ Gửi request tới:     │              │ Lắng nghe cổng 5432  │       │
│   │ "postgres_db:5432"   │              │                      │       │
│   └──────────────────────┘              └──────────────────────┘       │
│              ▲                                     ▲                   │
│              │       MÁY CHỦ DNS CỦA DOCKER        │                   │
│              └─────► (127.0.0.11 tự động phân ─────┘                   │
│                       giải tên thành IP ảo của container)              │
└────────────────────────────────────────────────────────────────────────┘
```

- Mỗi container là một chiếc máy độc lập. Với container Backend, `localhost` chính là bản thân nó (nơi không có PostgreSQL nào đang chạy).
- Docker tích hợp sẵn một **Embedded DNS Server** (tại địa chỉ ảo `127.0.0.11`). Khi `api_server` gửi gói tin tới tên miền `postgres_db`, DNS của Docker sẽ tự động chuyển đổi tên này thành địa chỉ IP ảo nội bộ của container `postgres_db`.

---

## 4. Các Lệnh Thao Tác Với Docker Compose

| Lệnh | Ý nghĩa thực tế |
| :--- | :--- |
| `docker compose up -d` | Tự động tải image, build và khởi chạy toàn bộ các dịch vụ dưới nền ngầm (`-d`) |
| `docker compose down` | Dừng và xóa bỏ toàn bộ các container và mạng ảo đã tạo (vẫn giữ lại volume dữ liệu) |
| `docker compose down -v` | Dừng và **XÓA SẠCH CẢ VOLUME DỮ LIỆU** (Dùng khi muốn reset sạch CSDL về ban đầu) |
| `docker compose ps` | Xem trạng thái hoạt động của tất cả các container trong file compose |
| `docker compose logs -f api_server` | Theo dõi log thời gian thực của riêng dịch vụ `api_server` |
| `docker compose build --no-cache` | Ép Docker phải build lại mã nguồn mới mà không dùng cache cũ |
| `docker compose exec api_server sh` | Chui thẳng vào bên trong terminal của container `api_server` |

---

## 5. Khi Nào Cần Kubernetes (K8s)?

Docker Compose là công cụ tuyệt vời cho môi trường phát triển (Local Development) và các máy chủ đơn lẻ quy mô vừa và nhỏ. Tuy nhiên, khi hệ thống của bạn phát triển thành quy mô doanh nghiệp lớn:

```text
       GIỚI HẠN CỦA DOCKER COMPOSE                     SỨC MẠNH CỦA KUBERNETES (K8S)
┌────────────────────────────────────────┐       ┌────────────────────────────────────────┐
│ 1 MÁY CHỦ DUY NHẤT (Single-Node Server)│       │ CỤM CÁC MÁY CHỦ (Multi-Node Cluster)   │
│ - Nếu máy chủ này bị sập nguồn điện    │       │ ┌────────────┐┌────────────┐┌─────────┐│
│   hoặc hỏng ổ cứng:                    │       │ │ Node 1     ││ Node 2     ││ Node 3  ││
│   TOÀN BỘ WEBSITE SẬP!                 │       │ └────────────┘└────────────┘└─────────┘│
│ - Muốn tăng tải chỉ có thể nâng cấp CPU│       │ - Nếu Node 1 chết, K8s tự động dời     │
│   cho máy này (Vertical Scaling).      │       │   container sang Node 2 trong 1 giây!  │
│                                        │       │ - Tự động nhân bản Pods theo tải       │
│                                        │       │   (Auto-scaling: từ 2 lên 50 bản sao). │
└────────────────────────────────────────┘       └────────────────────────────────────────┘
```

### Các khái niệm cốt lõi của Kubernetes (K8s):
- **Node:** Một máy chủ vật lý hoặc máy chủ ảo trong cụm máy tính.
- **Pod:** Đơn vị nhỏ nhất trong K8s (chứa một hoặc nhiều container dùng chung mạng và ổ đĩa).
- **Deployment:** Quản lý số lượng bản sao (Replicas) của Pods và thực hiện cập nhật mã mới không gián đoạn (**Zero-downtime Rolling Update**).
- **Service:** Đóng vai trò là Load Balancer nội bộ phân phối lưu lượng truy cập tới các Pods.
- **Ingress:** Cửa ngõ định tuyến lưu lượng từ bên ngoài Internet vào các Service bên trong.

> [!TIP]
> **Khuyến nghị thực tế:** Hãy làm chủ tuyệt đối **Docker** và **Docker Compose** trước. Chỉ học và ứng dụng Kubernetes khi sản phẩm của bạn thực sự đòi hỏi kiến trúc chịu lỗi cao (High Availability) trên cụm nhiều máy chủ.

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **Docker Compose** cho phép quản lý và khởi chạy toàn bộ hệ thống đa dịch vụ chỉ bằng 1 file `docker-compose.yml`.
2. Khởi chạy toàn bộ hệ thống bằng `docker compose up -d`, tắt bằng `docker compose down`.
3. Tận dụng **Service Discovery**: Các container trong cùng mạng Docker tự động giao tiếp với nhau bằng chính tên của `service` thay vì dùng địa chỉ IP cố định.
4. Dùng **`depends_on` kết hợp `healthcheck`** để đảm bảo CSDL đã sẵn sàng trước khi Backend khởi động.
5. Docker Compose tối ưu cho 1 máy chủ (Single-node); khi cần mở rộng quy mô trên cụm hàng chục máy chủ (Cluster), ta nâng cấp lên **Kubernetes (K8s)**.
