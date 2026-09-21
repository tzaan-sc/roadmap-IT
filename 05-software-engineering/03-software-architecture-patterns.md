# 03 - Các Mẫu Kiến Trúc Phần Mềm (Software Architecture Patterns)

> *"Một kiến trúc phần mềm tốt cho phép bạn trì hoãn các quyết định lớn (chọn CSDL nào, dùng framework gì) mà không làm ảnh hưởng đến lõi nghiệp vụ của hệ thống."* — **Uncle Bob**

---

## 1. Kiến Trúc Phần Mềm Là Gì?

Nếu **Design Patterns** là cách bạn thiết kế từng căn phòng, viên gạch hay cánh cửa, thì **Kiến Trúc Phần Mềm (Software Architecture)** là bản quy hoạch tổng thể của cả một tòa nhà chọc trời:
- Xác định các thành phần lớn của hệ thống.
- Thiết lập ranh giới trách nhiệm giữa các thành phần.
- Định nghĩa cách các thành phần giao tiếp với nhau để đảm bảo hiệu năng, độ tin cậy và khả năng mở rộng.

---

## 2. Sự Tiến Hóa: Monolith vs Modular Monolith vs Microservices

Nhiều lập trình viên trẻ thường bị cuốn theo xu hướng: *"Dự án mới là phải chia Microservices ngay!"* — Đây là nguyên nhân khiến hàng ngàn dự án khởi nghiệp sụp đổ vì sự phức tạp không đáng có.

```text
       MONOLITH                    MODULAR MONOLITH                     MICROSERVICES
┌─────────────────────┐        ┌───────────────────────┐        ┌─────────┐   ┌─────────┐
│     ỨNG DỤNG        │        │   ┌─────┐   ┌─────┐   │        │ Auth    │   │ Order   │
│  Tất cả code nằm    │        │   │Auth │   │Order│   │        │ Service │   │ Service │
│  chung trong 1 khối,│        │   └─────┘   └─────┘   │        └────┬────┘   └────┬────┘
│  dùng chung 1 DB.   │        │   ┌───────────────┐   │             │ Mạng HTTP   │
│                     │        │   │    Payment    │   │             ▼ / gRPC      ▼
│                     │        │   └───────────────┘   │        ┌─────────┐   ┌─────────┐
│                     │        │ Ranh giới module sạch,│        │ Auth DB │   │Order DB │
│                     │        │ deploy chung 1 khối!  │        └─────────┘   └─────────┘
└─────────────────────┘        └───────────────────────┘      (Độ phức tạp mạng cực cao!)
```

### Bảng so sánh 3 phong cách kiến trúc:

| Tiêu chí | Traditional Monolith | Modular Monolith (Khuyến nghị) | Microservices |
| :--- | :--- | :--- | :--- |
| **Triển khai (Deploy)** | 1 file duy nhất (Dễ nhất) | 1 file duy nhất (Rất dễ) | Hàng chục file/container độc lập (Phức tạp) |
| **Độ trễ giao tiếp** | Siêu tốc (Gọi hàm trong RAM) | Siêu tốc (Gọi hàm trong RAM) | Chậm hơn (Gọi qua mạng HTTP/gRPC) |
| **Toàn vẹn dữ liệu** | Giao dịch ACID CSDL cục bộ | Giao dịch ACID CSDL cục bộ | **Distributed Transaction** (Cực khó, cần Saga) |
| **Gỡ lỗi (Debug)** | Rất dễ, chạy được trên máy cá nhân | Rất dễ | Rất khó, cần Distributed Tracing |
| **Khi nào nên dùng?** | Dự án nhỏ, ít người làm | **90% các dự án từ nhỏ đến quy mô lớn** | Khi công ty có hàng trăm kỹ sư chia nhiều team độc lập |

---

## 3. Kiến Trúc Phân Tầng (Layered / N-Tier Architecture)

Đây là kiến trúc truyền thống và phổ biến nhất trong các ứng dụng Web:

```text
┌────────────────────────────────────────────────────────┐
│ 1. PRESENTATION LAYER (Tầng giao diện / Controller)    │
│ Nhận HTTP request, parse JSON, validate đầu vào.       │
└───────────────────────────┬────────────────────────────┘
                            │ Phụ thuộc chiều xuống
                            ▼
┌────────────────────────────────────────────────────────┐
│ 2. BUSINESS LOGIC LAYER (Tầng nghiệp vụ / Service)     │
│ Chứa toàn bộ logic tính toán, quy tắc doanh nghiệp.    │
└───────────────────────────┬────────────────────────────┘
                            │ Phụ thuộc chiều xuống
                            ▼
┌────────────────────────────────────────────────────────┐
│ 3. DATA ACCESS LAYER (Tầng truy cập dữ liệu / Repo)    │
│ Tương tác CSDL, gọi câu lệnh SQL / ORM.                │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ 4. DATABASE LAYER (PostgreSQL, MySQL, MongoDB)         │
└────────────────────────────────────────────────────────┘
```
- **Ưu điểm:** Cực kỳ dễ hiểu, chia tách trách nhiệm rõ ràng theo chiều dọc.
- **Hạn chế:** Tầng Business Logic vô tình bị phụ thuộc vào Tầng Data Access (nếu đổi CSDL có thể ảnh hưởng tới logic nghiệp vụ).

---

## 4. Clean Architecture & Hexagonal Architecture (Ports and Adapters)

Để giải quyết nhược điểm của kiến trúc phân tầng, Robert C. Martin (Uncle Bob) đề xuất **Clean Architecture**, đảo ngược hoàn toàn hướng phụ thuộc:

```text
                           CÁC VÒNG TRÒN CỦA CLEAN ARCHITECTURE
      ┌────────────────────────────────────────────────────────────────────────┐
      │ FRAMEWORKS & DRIVERS (Vòng ngoài cùng: Web, DB, UI, Devices, Express)   │
      │   ┌────────────────────────────────────────────────────────────────┐   │
      │   │ INTERFACE ADAPTERS (Controllers, Gateways, Presenters)         │   │
      │   │   ┌────────────────────────────────────────────────────────┐   │   │
      │   │   │ APPLICATION BUSINESS RULES (Use Cases / Services)      │   │   │
      │   │   │   ┌────────────────────────────────────────────────┐   │   │   │
      │   │   │   │ ENTERPRISE BUSINESS RULES                      │   │   │   │
      │   │   │   │ (ENTITIES / DOMAIN CORE)                       │   │   │   │
      │   │   │   │   - Trái tim thuần khiết của hệ thống          │   │   │   │
      │   │   │   │   - KHÔNG chứa bất kỳ thư viện bên ngoài nào!  │   │   │   │
      │   │   │   └────────────────────────────────────────────────┘   │   │   │
      │   │   └──────────────────────────▲─────────────────────────────┘   │   │
      │   └──────────────────────────────┼─────────────────────────────────┘   │
      └──────────────────────────────────┼─────────────────────────────────────┘
                             QUY TẮC PHỤ THUỘC (THE DEPENDENCY RULE)
                       Mọi mũi tên phụ thuộc CHỈ ĐƯỢC PHÉP HƯỚNG VÀO TRONG!
```

### 4.1. Quy Tắc Phụ Thuộc (The Dependency Rule)
- **Tầng bên trong hoàn toàn không biết gì về tầng bên ngoài:**
  - `Entities` và `Use Cases` nằm ở lõi trung tâm. Chúng là code TypeScript/Java/Python thuần túy, **không phụ thuộc vào Express, không phụ thuộc vào Prisma/Postgres, không phụ thuộc vào React**!
  - Nếu ngày mai bạn đổi Web framework từ Express sang Fastify, hoặc đổi Database từ Postgres sang MongoDB: **100% mã nguồn lõi nghiệp vụ (Use Cases) không cần sửa đổi một dòng nào!**

---

### 4.2. Kiến Trúc Lục Giác (Hexagonal Architecture / Ports & Adapters)

Mô hình Lục giác xem ứng dụng như một hòn đảo trung tâm độc lập, giao tiếp với thế giới bên ngoài thông qua các **Ports (Giao diện Interface)** và **Adapters (Hiện thực cụ thể)**:

```text
               PHÍA ĐIỀU KHIỂN (DRIVING)                    PHÍA ĐƯỢC ĐIỀU KHIỂN (DRIVEN)
             (Ai gửi lệnh vào ứng dụng?)                     (Ứng dụng cần ai để làm việc?)

      [ Web REST API ] ──► [ HTTP Adapter ]                  [ Postgres Adapter ] ──► [ CSDL ]
                                    │                                  ▲
                                    ▼                                  │
                            ┌───────────────┐                  ┌───────────────┐
                            │ DRIVING PORT  │                  │  DRIVEN PORT  │
                            │  (Inbound)    │                  │  (Outbound)   │
                            └───────┬───────┘                  └───────▲───────┘
                                    │                                  │
                                    ▼                                  │
                            ┌──────────────────────────────────────────┴───┐
                            │            LÕI NGHIỆP VỤ (DOMAIN CORE)       │
                            │           (Pure Business Logic Entities)     │
                            └──────────────────────────────────────────────┘
```

- **Port (Cổng):** Là các Interface định nghĩa cách giao tiếp (ví dụ: `IUserRepository`, `IPaymentGateway`).
- **Adapter (Bộ điều hợp):** Là mã cụ thể gắn vào cổng đó (ví dụ: `PostgresUserAdapter` cắm vào `IUserRepository`).

---

## 5. Tóm Tắt & Ghi Nhớ Nhanh

1. Đừng vội vàng chia **Microservices** khi quy mô còn nhỏ; hãy bắt đầu bằng **Modular Monolith** để giữ tốc độ phát triển cao nhất và tránh độ phức tạp của mạng.
2. **Layered Architecture** phân chia theo 3 tầng (Controller - Service - Repository), rất trực quan và dễ tiếp cận.
3. **Clean Architecture & Hexagonal Architecture** đặt **Lõi nghiệp vụ (Domain)** làm trung tâm tối thượng; các chi tiết như Database, Web Framework, Third-party APIs chỉ là các công cụ bên ngoài cắm vào qua **Ports & Adapters**.
4. Luôn tuân thủ **Quy tắc phụ thuộc hướng vào trong**: Code nghiệp vụ lõi không bao giờ được phép `import` các thư viện framework của tầng bên ngoài.
