# 01 - Chuyên Sâu Kỹ Sư Backend (Backend Engineering Specialization)

> **Mục tiêu bài học:** Nâng cấp tư duy từ việc chỉ biết viết API CRUD đơn giản lên trình độ một Kỹ sư Backend thực thụ: làm chủ các framework kiến trúc chuẩn công nghiệp (NestJS, FastAPI, Spring Boot), tối ưu hóa hiệu năng bằng Redis Cache, xử lý tác vụ nền bất đồng bộ với Message Queue và hiểu sâu các giao thức hiện đại (REST, GraphQL, gRPC).

---

## 1. Chân Dung Của Một Kỹ Sư Backend Chuyên Nghiệp

Một người mới học thường nghĩ: *"Backend chỉ là viết vài endpoint nhận dữ liệu rồi lưu vào Database."*
Tuy nhiên, khi ứng dụng có **100,000 người dùng đồng thời**, sự khác biệt giữa "người biết code" và "kỹ sư backend" nằm ở 4 tiêu chí sống còn:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                      4 TRỤ CỘT CỦA KỸ SƯ BACKEND                       │
└───────┬──────────────────┬───────────────────┬──────────────────┬──────┘
        │                  │                   │                  │
        ▼                  ▼                   ▼                  ▼
  1. SCALABILITY     2. RELIABILITY      3. SECURITY        4. MAINTAINABILITY
  (Khả năng mở rộng) (Độ tin cậy)        (Bảo mật)          (Dễ bảo trì)
  Hệ thống vẫn chạy  Không bao giờ mất   Chống tấn công,    Code phân tầng sạch,
  mượt khi lượng     dữ liệu tiền tệ,    phân quyền chặt    dễ dàng mở rộng và
  truy cập tăng gấp  tự phục hồi khi     chẽ, mã hóa an     viết Unit/Integration
  100 lần.           server crash.       toàn.              Tests.
```

---

## 2. So Sánh Bộ Ba Framework Chuẩn Doanh Nghiệp (Enterprise Frameworks)

Khi làm các dự án lớn, người ta không dùng Express hay Flask trần trụi vì chúng quá tự do, dễ dẫn đến cấu trúc code lộn xộn. Các doanh nghiệp chuẩn mực ưu tiên 3 framework có tính kỷ luật kiến trúc cao:

```text
       NODE.JS (NestJS)             PYTHON (FastAPI)            JAVA (Spring Boot)
┌─────────────────────────────┐ ┌─────────────────────┐ ┌──────────────────────────────┐
│ - Cấu trúc Module / DI      │ │ - Hiệu năng Async   │ │ - Chuẩn mực bảo mật tuyệt đối│
│ - Typescript 100%           │ │ - Pydantic schemas  │ │ - Xử lý đa luồng & Microserv │
│ - Giống tư duy Spring Boot  │ │ - Tự sinh Swagger   │ │ - Vua của hệ thống Ngân hàng │
└─────────────────────────────┘ └─────────────────────┘ └──────────────────────────────┘
```

### 2.1. NestJS (Node.js + TypeScript)
NestJS mang toàn bộ tư duy kiến trúc hướng đối tượng của Java Spring Boot và Angular vào thế giới Node.js:
- **Module:** Chia nhỏ ứng dụng thành các khối chức năng độc lập (`UserModule`, `OrderModule`, `PaymentModule`).
- **Dependency Injection (DI) & Inversion of Control (IoC):** Controller không tự khởi tạo Service bằng `new Service()`, mà framework sẽ tự động "tiêm" (inject) instance vào constructor, giúp việc thay thế Mock Service khi viết Unit Test trở nên siêu dễ dàng.
- **Pipes & DTO (Data Transfer Object):** Tự động validate dữ liệu đầu vào bằng `class-validator` trước khi nó kịp chạm tới Controller.
- **Guards:** Chặn và kiểm tra quyền hạn (Role-based access control) trước khi route handler được kích hoạt.

### 2.2. FastAPI (Python)
FastAPI là framework Python hiện đại nhất hiện nay, được các công ty AI và Data Science tin dùng tuyệt đối:
- **Tốc độ ngang ngửa NodeJS và Go:** Xây dựng trên nền tảng Starlette và Uvicorn với kiến trúc bất đồng bộ `async/await`.
- **Pydantic Data Validation:** Định nghĩa schema dữ liệu bằng Python Type Hints, tự động ép kiểu và ném lỗi 422 chi tiết nếu dữ liệu gửi lên sai.
- **Tự động sinh tài liệu API (Swagger UI / OpenAPI):** Chỉ cần viết code xong, truy cập đường dẫn `/docs` là bạn có ngay một trang web tương tác API chuyên nghiệp mà không cần dùng Postman!

### 2.3. Spring Boot (Java)
Vua của các hệ sinh thái tài chính, chứng khoán, viễn thông và thương mại điện tử khổng lồ:
- **Xử lý giao dịch mạnh mẽ:** Quản lý Transaction phức tạp chỉ bằng annotation `@Transactional`.
- **Spring Security:** Hệ thống bảo mật toàn diện bậc nhất thế giới (OAuth2, LDAP, JWT, Session).
- **Hệ sinh thái Microservices:** Tích hợp sẵn sàng với Spring Cloud, Eureka Discovery, Config Server, Circuit Breaker (Resilience4j).

---

## 3. Bộ Đệm Dữ Liệu Tốc Độ Cao Với Redis (Caching Strategy)

Database (PostgreSQL / MySQL) đọc ghi dữ liệu từ ổ đĩa cứng (SSD/NVMe). Khi có 10,000 người cùng xem trang chủ sản phẩm trong 1 giây, Database sẽ quá tải 100% CPU!
**Redis** là một CSDL lưu trữ hoàn toàn trên **bộ nhớ RAM** (In-memory Data Store) với tốc độ đọc ghi dưới **1 mili-giây**.

### Mô hình Cache-Aside Pattern (Chiến lược phổ biến nhất):

```text
[ CLIENT ] ── 1. Gọi GET /products ──► [ BACKEND API ]
                                              │
                                  2. Kiểm tra trong REDIS?
                                       /              \
                                (ĐÃ CÓ)                (CHƯA CÓ)
                               CACHE HIT               CACHE MISS
                                  │                        │
               3. Trả về ngay lập tức! (0.5ms)      3. Query xuống Database (50ms)
                                                           │
                                                    4. Lưu kết quả vào Redis
                                                       kèm thời gian hết hạn (TTL = 5m)
                                                           │
                                                    5. Trả về cho Client
```

### Các thách thức lớn khi dùng Cache trong thực tế:
1. **Cache Invalidation:** *"Làm sao để khi Admin sửa giá sản phẩm, Cache được xóa ngay lập tức để người dùng không nhìn thấy giá cũ?"*
2. **Cache Stampede / Breakdown:** Khi một Key rất "hot" (ví dụ sản phẩm flash sale) vừa hết hạn (expired), hàng vạn request cùng lúc đổ dồn xuống Database thật làm sập server! (Khắc phục bằng Mutex Lock).

---

## 4. Xử Lý Tác Vụ Nền Bất Đồng Bộ Với Message Queues

Trong thiết kế Backend, **tuyệt đối không để người dùng phải chờ đợi các tác vụ tốn thời gian** (ví dụ: tạo file PDF hóa đơn mất 5 giây, nén ảnh đại diện mất 3 giây, gửi email kích hoạt mất 2 giây).

```text
[ CLIENT ] ── 1. Bấm Đặt Hàng ──► [ BACKEND SERVER ]
                                          │
                                  2. Lưu đơn hàng vào DB
                                  3. Đẩy việc "Gửi Email" vào QUEUE (1ms)
                                          │
[ CLIENT ] ◄── 4. Báo "Đặt thành công!" ──┘ (Phản hồi cực nhanh trong 50ms!)

                                  [ MESSAGE QUEUE ]
                                  (RabbitMQ / BullMQ)
                                          │
                                          ▼
                                  [ WORKER SERVER ]
                                  (Lấy task ra xử lý ngầm,
                                   tự retry nếu gửi email lỗi)
```

- **Công cụ hàng đầu:**
  - **RabbitMQ:** Hàng đợi tin nhắn chuẩn AMQP, độ tin cậy cực cao, định tuyến linh hoạt.
  - **BullMQ:** Hàng đợi tin nhắn xây dựng trên nền tảng Redis dành riêng cho hệ sinh thái Node.js/TypeScript.
  - **Celery:** Hệ thống Distributed Task Queue tiêu chuẩn của thế giới Python.
  - **Apache Kafka:** Xử lý luồng sự kiện (Event Streaming) khổng lồ hàng triệu message mỗi giây.

---

## 5. Các Giao Thức Giao Tiếp: REST vs GraphQL vs gRPC

| Tiêu chí | RESTful API | GraphQL | gRPC |
| :--- | :--- | :--- | :--- |
| **Định dạng dữ liệu**| JSON (Văn bản) | JSON (Văn bản) | **Protocol Buffers (Nhị phân)** |
| **Giao thức mạng** | HTTP/1.1 hoặc HTTP/2 | HTTP/1.1 hoặc HTTP/2 | **HTTP/2 (Ghép kênh, Streaming)** |
| **Bản chất truy vấn**| Nhiều endpoints (`/users`, `/orders`) | 1 endpoint duy nhất (`/graphql`), Client tự chọn trường | Gọi hàm từ xa (Remote Procedure Call) như hàm nội bộ |
| **Ưu điểm** | Đơn giản, phổ biến nhất, dễ cache | Giải quyết triệt để lỗi Over-fetching / Under-fetching | **Tốc độ siêu nhanh, siêu nhẹ** (nhanh hơn REST gấp 7-10 lần) |
| **Ứng dụng tối ưu** | Public API cho bên thứ 3, Web cơ bản | Mobile App cần tiết kiệm băng thông 4G | **Giao tiếp nội bộ giữa các Microservices** trong hệ thống lớn |

---

## 6. Bảo Vệ & Giám Sát API Hệ Thống

1. **Rate Limiting (Giới hạn tần suất gọi API):**
   - Ngăn chặn tấn công từ chối dịch vụ (DDoS) hoặc cào dữ liệu (Web scraping).
   - Ví dụ: Mỗi IP chỉ được phép gọi tối đa 100 requests trong vòng 1 phút (sử dụng thuật toán Token Bucket hoặc Leaky Bucket với Redis).
2. **API Gateway:**
   - Đóng vai trò là cửa ngõ duy nhất đón nhận mọi request từ Internet: chịu trách nhiệm giải mã SSL/TLS, xác thực JWT tập trung, phân luồng tải (Load Balancing) tới các service con.
3. **Giám Sát & Truy Vết (Observability):**
   - **Metrics:** Dùng **Prometheus** thu thập số liệu (CPU, RAM, số request/giây) và vẽ biểu đồ cảnh báo trực quan với **Grafana**.
   - **Centralized Logging:** Gom toàn bộ log từ hàng chục server về một chỗ bằng **ELK Stack** (Elasticsearch, Logstash, Kibana) hoặc **Loki**.
   - **Distributed Tracing:** Dùng **OpenTelemetry / Jaeger** để theo dõi một request đi qua những service nào và bị chậm ở bước nào.

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. Kỹ sư Backend hiện đại cần làm chủ framework có tính cấu trúc cao: **NestJS** (TypeScript), **FastAPI** (Python), hoặc **Spring Boot** (Java).
2. Áp dụng **Dependency Injection (DI)** để tách rời sự phụ thuộc và giúp code dễ kiểm thử (Unit Testing).
3. Luôn dùng **Redis (Cache-Aside Pattern)** để bảo vệ Database khỏi các đợt bùng nổ truy cập.
4. Đẩy toàn bộ tác vụ tốn thời gian ra xử lý nền bằng **Message Queue (RabbitMQ / BullMQ / Celery)** để giữ độ trễ của API luôn dưới 100ms.
5. Hiểu rõ khi nào dùng **REST** (chuẩn mực), **GraphQL** (tối ưu dữ liệu cho mobile), và **gRPC** (siêu tốc cho Microservices nội bộ).
