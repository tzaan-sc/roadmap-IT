# Level 5 — Software Engineering (Kỹ Nghệ Chế Tác Phần Mềm)

> *"Biết code chỉ giúp bạn tạo ra một chương trình chạy được. Nhưng để xây dựng một hệ thống phần mềm tồn tại và mở rộng qua 10 năm mà không bị sụp đổ, bạn cần tư duy của một Kỹ sư Phần mềm thực thụ."*

Chào mừng bạn đến với **Level 5** trong lộ trình [Roadmap IT](../README.md). Đây là cấp độ tạo ra khoảng cách lớn nhất giữa một lập trình viên bình thường và một kỹ sư cấp cao (Senior Engineer): chuyển đổi từ việc viết code tùy tiện sang **nghệ thuật thiết kế phần mềm sạch (Clean Code), mẫu thiết kế chuẩn mực (Design Patterns), kiến trúc bền vững (Clean Architecture) và kiểm thử tự động (Automated Testing & TDD)**.

---

## 🧭 Bức Tranh Tiến Hóa: Từ Coder Đến Kỹ Sư Phần Mềm

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. TƯ DUY MÃ SẠCH & NGUYÊN LÝ NỀN TẢNG (CLEAN CODE & SOLID)                 │
│ Naming, Small Functions ──► DRY, KISS, YAGNI ──► SOLID Principles           │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. CÁC MẪU THIẾT KẾ THỰC CHIẾN (GOF DESIGN PATTERNS)                        │
│ Creational: Factory, Builder ──► Structural: Adapter, Decorator              │
│ Behavioral: Strategy, Observer ──► Architectural: Repository & DI           │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 3. KIẾN TRÚC PHẦN MỀM BỀN VỮNG (SOFTWARE ARCHITECTURE)                      │
│ Modular Monolith ──► Clean Architecture (Dependency Rule) ──► Hexagonal     │
│ Core Domain Entities ◄── Use Cases ◄── Controllers ◄── Frameworks/DB        │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 4. CHIẾN LƯỢC KIỂM THỬ TOÀN DIỆN & TDD (TESTING & QUALITY)                  │
│ Testing Pyramid: 70% Unit ──► 20% Integration ──► 10% E2E                  │
│ Test Doubles (Mocks, Stubs) ──► TDD: Red ──► Green ──► Refactor             │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 5. THỰC HÀNH TÁI CẤU TRÚC (REFACTORING LABS) & PHỎNG VẤN SENIOR             │
│ Spaghetti Code Refactoring ──► Automated Mock Testing ──► 15 Core Q&A       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là 5 chuyên đề chuyên sâu đã được biên soạn hoàn chỉnh trong thư mục này:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Mã Sạch & 5 Nguyên Lý SOLID** | Nghệ thuật Clean Code (Quy tắc đặt tên, thiết kế hàm ngắn dưới 20 dòng, DRY, KISS, YAGNI, Boy Scout Rule), Phân tích chuyên sâu 5 nguyên lý SOLID (**SRP, OCP, LSP, ISP, DIP**) kèm mã nguồn so sánh trực quan: ❌ Vi phạm vs ✅ Chuẩn hóa. | [Xem bài viết](./01-clean-code-and-solid-principles.md) |
| **02** | **Các Mẫu Thiết Kế Thực Chiến** | Bản chất 3 nhóm Design Patterns (GoF), Đi sâu vào 8 mẫu thiết kế thực tế: **Factory Method**, **Builder** (Method Chaining), **Singleton** (kèm cảnh báo Anti-pattern), **Adapter** (Cầu nối), **Decorator** (Mở rộng tính năng động), **Strategy** (Xóa sổ `if/else`), **Observer** (Pub/Sub), **Repository & DI**. | [Xem bài viết](./02-design-patterns-in-depth.md) |
| **03** | **Các Mẫu Kiến Trúc Phần Mềm** | Đập tan định kiến kiến trúc: So sánh **Monolith vs Modular Monolith vs Microservices**, Kiến trúc phân tầng (Layered 3-Tier), Làm chủ **Clean Architecture (Uncle Bob)** và **Kiến trúc Lục giác (Hexagonal / Ports & Adapters)**, Quy tắc phụ thuộc hướng vào trong. | [Xem bài viết](./03-software-architecture-patterns.md) |
| **04** | **Chiến Lược Kiểm Thử & TDD** | Tại sao phải viết Automated Tests, Mô hình **Kim tự tháp kiểm thử (Testing Pyramid: 70% Unit, 20% Integration, 10% E2E)**, Kỹ thuật làm giả đối tượng (**Test Doubles: Dummy, Stub, Spy, Mock, Fake**), Chu trình **TDD: Red ──► Green ──► Refactor** với Vitest/Jest, Bản chất Code Coverage. | [Xem bài viết](./04-testing-strategies-and-tdd.md) |
| **05** | **Thực Hành Tái Cấu Trúc & Phỏng Vấn** | **2 Bài Lab Thực Chiến:** Lab tái cấu trúc Spaghetti Code thành Clean Code áp dụng Strategy/Repository, Lab viết Unit Test & Mocking tự động 100%. **Bộ 15 câu hỏi phỏng vấn kỹ thuật Software Engineering cốt lõi** có lời giải chi tiết và Checklist tự đánh giá năng lực. | [Xem bài viết](./05-practice-refactoring-and-interview.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 5

1. **Xóa bỏ thói quen viết mã cẩu thả:** Code của bạn trở nên trong sáng, tự giải thích nghĩa, đồng đội đọc vào cảm thấy dễ chịu và tôn trọng.
2. **Thiết kế phần mềm chuẩn SOLID:** Nắm vững cách mở rộng tính năng mới mà không phải sửa code cũ, không làm hỏng tính đúng đắn của hệ thống.
3. **Làm chủ Clean Architecture:** Biết cách bảo vệ lõi nghiệp vụ của doanh nghiệp độc lập hoàn toàn khỏi sự phụ thuộc vào Database hay Web Framework.
4. **Tự tin tái cấu trúc mã (Refactoring) nhờ TDD:** Không còn nỗi sợ "sửa một dòng làm sập cả hệ thống", luôn có hệ thống kiểm thử tự động bảo vệ 24/7.

---

## ⏭️ Bước Tiếp Theo
Sau khi đã rèn luyện tư duy kỹ sư phần mềm chuẩn mực, bạn đã sẵn sàng tiến lên cấp độ tự động hóa và điện toán đám mây:
👉 **[Level 6: DevOps + Cloud](../06-devops-cloud/README.md)**
