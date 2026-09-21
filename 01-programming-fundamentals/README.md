# Level 1 — Programming Fundamentals (Tư Duy & Nền Tảng Lập Trình)

> *"Mọi người trên đất nước này nên học cách lập trình máy tính, bởi vì nó dạy bạn cách suy nghĩ."* — **Steve Jobs**

Chào mừng bạn đến với **Level 1** trong lộ trình [Roadmap IT](../README.md). Đây là cấp độ quan trọng nhất đối với người mới bắt đầu. Nếu Level 0 dạy bạn hiểu "cỗ máy", thì Level 1 sẽ trang bị cho bạn ngôn ngữ và tư duy logic để điều khiển cỗ máy đó giải quyết các bài toán trong thực tế.

---

## 🗺️ Bản Đồ Kiến Thức (Knowledge Flow)

```text
┌─────────────────────────────────────────────────────────────┐
│ 1. SYNTAX & CORE CONCEPTS                                   │
│ Variables (Stack/Heap) ──► Operators ──► Control Flow       │
│ Loops ──► Functions & Closures ──► Scope                    │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. DATA STRUCTURES DEEP DIVE                                │
│ Arrays ──► Linked Lists ──► Stacks & Queues                 │
│ Hash Tables (O(1)) ──► Sets ──► Trees (BST) ──► Graphs      │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. ALGORITHMS & BIG-O COMPLEXITY                            │
│ Big-O Notation ──► Binary Search ──► Merge/Quick Sort       │
│ Recursion ──► Two Pointers ──► Sliding Window ──► DP Basics │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. OBJECT-ORIENTED PROGRAMMING (OOP)                        │
│ Classes & Objects ──► Encapsulation ──► Abstraction         │
│ Inheritance (Composition) ──► Polymorphism ──► SOLID Basics │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. ERROR HANDLING & PROFESSIONAL DEBUGGING                  │
│ Exceptions (try/catch/finally) ──► Custom Errors            │
│ Breakpoints & Call Stack ──► Industrial Logging             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. LEETCODE CHALLENGES & REAL-WORLD OOP DESIGN              │
│ Two Sum ──► Valid Parentheses ──► Reverse Linked List       │
│ Library System Design ──► Competency Checklist              │
└─────────────────────────────────────────────────────────────┘
```

---

## 💡 Lựa Chọn Ngôn Ngữ Khởi Đầu: Nên Học Gì?

Trong lập trình, **ngôn ngữ chỉ là công cụ, tư duy giải thuật và cấu trúc dữ liệu mới là vĩnh cửu**. Tuy nhiên, chọn đúng ngôn ngữ khởi đầu sẽ giúp bạn tiết kiệm hàng trăm giờ mò mẫm:

| Ngôn ngữ | Thế mạnh vượt trội | Định hướng nghề nghiệp phù hợp nhất | Độ khó nhập môn |
| :--- | :--- | :--- | :---: |
| **Python** | Cú pháp siêu trong sáng, gần với ngôn ngữ tự nhiên, hệ sinh thái thư viện khổng lồ | Data Science, Trí tuệ nhân tạo (AI / Machine Learning), Automation, Backend API | ⭐ (Rất dễ) |
| **JavaScript / TypeScript** | Ngôn ngữ duy nhất chạy trên trình duyệt web, phổ biến số 1 thế giới, chạy cả Frontend lẫn Backend (Node.js) | Lập trình Web Fullstack (React, Next.js, Node.js), Mobile App (React Native) | ⭐⭐ (Trung bình) |
| **Java / C#** | Hướng đối tượng chuẩn mực tuyệt đối, cực kỳ chặt chẽ, bảo mật cao | Doanh nghiệp lớn (Enterprise), Ngân hàng, Tài chính, Backend quy mô khủng | ⭐⭐⭐ (Khá) |
| **C / C++ / Rust** | Quản lý trực tiếp con trỏ và bộ nhớ RAM, tốc độ thực thi tối đa | Lập trình nhúng (IoT), Game Engine (Unreal), Hệ điều hành, Driver phần cứng | ⭐⭐⭐⭐⭐ (Khó) |

> [!TIP]
> **Khuyến nghị vàng cho người mới:**
> - Nếu mục tiêu là **Lập trình Web**: Chọn **JavaScript / TypeScript**.
> - Nếu mục tiêu là **AI / Data / Tự động hóa**: Chọn **Python**.
> - Nếu muốn xây dựng **nền tảng tư duy cấu trúc bộ nhớ sâu sắc nhất**: Học một chút **C/C++**, sau đó chuyển sang Python hoặc TypeScript.

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là 6 chuyên đề cốt lõi của Level 1 được biên soạn chuẩn mực:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Cú Pháp Cốt Lõi & Tư Duy Lập Trình** | Biến, Hằng số, Primitive vs Reference Type trong bộ nhớ, Toán tử & Short-circuit, Cấu trúc rẽ nhánh, Vòng lặp, Thiết kế hàm (Pure function), Scope & Closure. | [Xem bài viết](./01-syntax-and-core-concepts.md) |
| **02** | **Cấu Trúc Dữ Liệu Chuyên Sâu** | Mảng tĩnh vs Mảng động, Danh sách liên kết (Singly/Doubly), Ngăn xếp (Stack - LIFO), Hàng đợi (Queue - FIFO), Bảng băm (Hash Table - $O(1)$), Set, Cây (Binary Tree, BST), Đồ thị (Graph). | [Xem bài viết](./02-data-structures-deep-dive.md) |
| **03** | **Giải Thuật & Độ Phức Tạp Big-O** | Thang đo Big-O, Tìm kiếm nhị phân ($O(\log N)$), Sắp xếp tối ưu (Merge Sort, Quick Sort), Bản chất đệ quy & Call Stack, Mẫu tư duy Two Pointers, Sliding Window, Quy hoạch động (DP) cơ bản. | [Xem bài viết](./03-algorithms-and-big-o.md) |
| **04** | **Lập Trình Hướng Đối Tượng (OOP)** | Class vs Object, Bản chất con trỏ `this`/`self`, 4 Trụ Cột Vàng: Đóng gói (Encapsulation), Trừu tượng (Abstraction), Kế thừa (Inheritance), Đa hình (Polymorphism), Giới thiệu 5 nguyên lý SOLID. | [Xem bài viết](./04-object-oriented-programming.md) |
| **05** | **Quản Lý Lỗi & Kỹ Năng Debugging** | Phân loại lỗi (Syntax, Runtime, Logic), Cơ chế `try-catch-finally`, Custom Exceptions, Đọc hiểu Stack Trace từ dưới lên, Sử dụng Breakpoints & Debugger, Logging chuyên nghiệp. | [Xem bài viết](./05-error-handling-and-debugging.md) |
| **06** | **Thực Hành Thuật Toán & OOP Thử Thách** | 5 bài toán kinh điển phỏng vấn: Two Sum ($O(N)$), Valid Parentheses, Binary Search, Đảo ngược danh sách liên kết, Thiết kế hệ thống Thư viện OOP hoàn chỉnh kèm Checklist tự đánh giá. | [Xem bài viết](./06-practice-problems-and-challenges.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 1

1. Tự tin đọc hiểu cú pháp của bất kỳ ngôn ngữ lập trình phổ biến nào.
2. Nhìn một bài toán có thể phân tích được độ phức tạp thời gian và không gian (Big-O) để chọn đúng cấu trúc dữ liệu tối ưu nhất (Array hay Hash Map hay Tree).
3. Thiết kế mã nguồn có tổ chức, áp dụng thành thạo mô hình hướng đối tượng (OOP) sạch sẽ, bảo mật và dễ mở rộng.
4. Tự mình debug và sửa lỗi phần mềm một cách bài bản thay vì chỉ biết đoán mò.

---

## ⏭️ Bước Tiếp Theo
Sau khi vượt qua các bài tập và tích trọn bộ Checklist trong bài 06, bạn đã sẵn sàng bước sang:
👉 **[Level 2: Git + Linux + Database](../02-git-linux-database/README.md)**
