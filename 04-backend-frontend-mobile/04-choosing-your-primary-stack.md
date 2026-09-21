# 04 - Chiến Lược Chọn 1 Stack Chính & Lộ Trình 90 Ngày (Choosing Your Primary Stack)

> *"Tôi không sợ người biết 10,000 cú đá khác nhau. Tôi chỉ sợ người luyện một cú đá 10,000 lần."* — **Lý Tiểu Long**

---

## 1. Triết Lý Phát Triển Nghề Nghiệp: Kỹ Sư Chữ T (T-Shaped Engineer)

Một trong những sai lầm chết người của những người học lập trình là **cố gắng học tất cả mọi thứ cùng một lúc**: hôm nay học Spring Boot, ngày mai học Flutter, tuần sau lại nhảy sang Next.js rồi AI. Hậu quả là bạn biết mỗi thứ một chút ở mức bề nổi, nhưng khi đi phỏng vấn hay giải quyết bài toán hóc búa trong công ty thì **không thể làm độc lập được bất kỳ việc gì**.

Mô hình **Kỹ Sư Chữ T (T-Shaped Engineer)** là tiêu chuẩn vàng mà mọi công ty công nghệ săn đón:

```text
               KIẾN THỨC NỀN TẢNG RỘNG (THANH NGANG CỦA CHỮ T)
 ┌─────────────────────────────────────────────────────────────────────────────┐
 │ Phần cứng ──► Linux/OS ──► Git ──► SQL ──► Mạng HTTP ──► HTML/CSS/JS cơ bản │
 └──────────────────────────────────────┬──────────────────────────────────────┘
                                        │
                                        │   CHUYÊN MÔN SÂU (THANH DỌC CỦA CHỮ T)
                                        │   Chọn DUY NHẤT 1 hướng mũi nhọn:
                                        │   ┌───────────────────────────────┐
                                        │   │   BACKEND (NestJS / Java)     │
                                        ├──►│             HOẶC              │
                                        │   │   FRONTEND (Next.js / React)  │
                                        │   │             HOẶC              │
                                        │   │   MOBILE (React Native/Flutter│
                                        │   └───────────────────────────────┘
                                        ▼
                          Đạt trình độ Chuyên gia / Độc lập giải quyết vấn đề!
```

- **Thanh ngang:** Bạn đã tích lũy từ Level 0 đến Level 3 (hiểu hệ điều hành, biết gõ dòng lệnh, hiểu cấu trúc dữ liệu, biết gọi API và thao tác CSDL).
- **Thanh dọc:** Giờ là lúc bạn phải **chọn ĐÚNG 1 hướng đi chính** và đào sâu đến mức trở thành điểm tựa kỹ thuật vững chắc.

---

## 2. Phân Tích Thị Trường Tuyển Dụng & Tiềm Năng Nghề Nghiệp

| Hướng đi | Mức độ cạnh tranh | Nhu cầu tuyển dụng | Đặc thù công việc | Độ bền sự nghiệp |
| :--- | :--- | :--- | :--- | :--- |
| **Backend Engineer** | Trung bình | **Rất cao và ổn định** | Xử lý logic, CSDL, kiến trúc, bảo mật; ít thay đổi công nghệ | **Cực kỳ bền vững** (Càng nhiều năm kinh nghiệm càng đắt giá) |
| **Frontend Engineer** | Khá cao ở mức Junior | Cao (ở các công ty Product) | Chú trọng trải nghiệm người dùng, UI/UX, tối ưu Web Vitals, công nghệ đổi mới nhanh | Rất tốt nếu nắm vững TypeScript và kiến trúc phần mềm sạch |
| **Mobile Engineer** | Trung bình (Ít cạnh tranh hơn Web) | Ổn định, đãi ngộ cao | Tập trung vào người dùng di động, tối ưu hiệu năng thiết bị, có thể tự làm app kiếm tiền thụ động | Rất cao nhờ sự bùng nổ của Smartphone và thiết bị thông minh |

---

## 3. Bốn Combo Stack Vàng Được Săn Đón Nhất Hiện Nay

Thay vì tự phối hợp công nghệ rời rạc, dưới đây là 4 bộ Combo công nghệ chuẩn mực được thị trường việc làm săn đón nhất:

### COMBO 1: Fullstack Web Hiện Đại (TypeScript-Centric)
- **Frontend:** Next.js (App Router, Server Components) + TailwindCSS + shadcn/ui.
- **Backend:** NestJS (Node.js) hoặc Next.js Server Actions.
- **Database:** PostgreSQL + Prisma ORM.
- **Cache & Queue:** Redis + BullMQ.
- **Thế mạnh:** Dùng **duy nhất một ngôn ngữ TypeScript** xuyên suốt từ Client đến Database. Tốc độ làm sản phẩm của 1 kỹ sư bằng 2-3 người khác!

### COMBO 2: Kỹ Sư Backend & Trí Tuệ Nhân Tạo (Python-Centric)
- **Framework:** FastAPI (Asynchronous Python).
- **Database:** PostgreSQL + SQLAlchemy / Alembic.
- **Tác vụ nền:** Celery + Redis.
- **Hệ sinh thái:** Docker + Tích hợp các thư viện AI (LangChain, OpenAI API, PyTorch).
- **Thế mạnh:** Đón đầu làn sóng bùng nổ Trí tuệ nhân tạo (AI), rất dễ chuyển dịch sang làm Kỹ sư AI / AI Engineer sau này.

### COMBO 3: Enterprise Heavyweight (Java-Centric)
- **Framework:** Java 17/21 + Spring Boot 3 (Spring Security, Spring Data JPA).
- **Database:** PostgreSQL hoặc Oracle.
- **Message Broker:** Apache Kafka.
- **Hạ tầng:** Docker, Kubernetes, Microservices.
- **Thế mạnh:** Lương thưởng và chế độ cực kỳ hậu hĩnh tại các Ngân hàng, Tập đoàn Viễn thông, Công ty Chứng khoán và Fintech lớn.

### COMBO 4: Kỹ Sư Ứng Dụng Di Động Toàn Diện (Cross-Platform)
- **Nền tảng:** **React Native** (nếu đã vững React/TS) HOẶC **Flutter** (nếu yêu thích ngôn ngữ Dart của Google).
- **Backend as a Service:** Firebase / Supabase hoặc kết hợp Backend NestJS/FastAPI.
- **Thế mạnh:** Một mình có thể xây dựng và xuất bản cả ứng dụng iOS lẫn Android lên 2 chợ ứng dụng toàn cầu.

---

## 4. Kế Hoạch Bứt Phá 90 Ngày: Từ Người Học Đến Ứng Viên Sáng Giá

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        KẾ HOẠCH HÀNH ĐỘNG 90 NGÀY                      │
└──────────────┬─────────────────────────┬───────────────────────────────┘
               │                         │
               ▼                         ▼
      THÁNG 1 (NGÀY 1 - 30)     THÁNG 2 (NGÀY 31 - 60)   THÁNG 3 (NGÀY 61 - 90)
       LÀM CHỦ FRAMEWORK         XÂY DỰNG PORTFOLIO       LUYỆN PHỎNG VẤN & XIN VIỆC
      - Đọc hết Docs chính thức - Xây 1 dự án Full     - Tối ưu hóa GitHub & CV
      - Hiểu sâu Best Practices   thực tế (E-commerce)  - Luyện thuật toán LeetCode
      - Viết code sạch chuẩn mực - Deploy lên môi trường  - Phỏng vấn thử Mock Interview
                                  thật (AWS/Vercel)
```

### Tháng 1: Đào sâu vào Framework cốt lõi
- Chọn đúng 1 Stack trong 4 Combo ở trên.
- Đọc kỹ tài liệu chính thức (Official Documentation).
- Không chỉ học cú pháp, hãy học cách tổ chức thư mục dự án theo chuẩn module và cách viết kiểm thử (Unit Testing).

### Tháng 2: Xây dựng 1 Đồ án "Đinh" (Signature Project)
- Đừng làm các bài tập đơn giản như ứng dụng TodoList nữa!
- Hãy xây dựng một dự án thực tế có đầy đủ:
  - Hệ thống xác thực đăng nhập phân quyền (JWT, Refresh Token).
  - Tích hợp thanh toán thật hoặc cổng gửi email tự động.
  - Caching bằng Redis để tăng tốc độ.
  - Deploy lên Cloud thật (Vercel, Render, AWS Lightsail) có domain riêng.

### Tháng 3: Chuẩn bị CV & Chinh phục phỏng vấn
- Chăm chút GitHub cá nhân: README dự án rõ ràng, có ảnh chụp màn hình, sơ đồ kiến trúc và link demo sống.
- Ôn tập kỹ bộ câu hỏi phỏng vấn trong bài học tiếp theo.
- Tự tin nộp hồ sơ vào các vị trí thực tập sinh (Intern) hoặc Junior.
