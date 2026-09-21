# Level 0 — Computer Fundamentals (Kiến Thức Nền Tảng Máy Tính)

> *"Đừng chỉ học viết code bề nổi. Muốn đi xa trong ngành kỹ thuật phần mềm, bạn phải hiểu cỗ máy bên dưới đang vận hành như thế nào."*

Chào mừng bạn đến với **Level 0** trong lộ trình [Roadmap IT](../README.md). Đây là tầng móng vững chắc nhất để bạn hiểu cách máy tính lưu trữ dữ liệu, thực thi câu lệnh và quản lý tài nguyên hệ thống trước khi bắt tay vào viết bất kỳ dòng mã nào.

---

## 🗺️ Bản Đồ Kiến Thức (Knowledge Flow)

```text
┌─────────────────────────────────────────────────────────────┐
│ 1. HARDWARE ARCHITECTURE                                    │
│ CPU (ALU/CU/Cache) ──► RAM ──► SSD/NVMe ──► GPU             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. OPERATING SYSTEM & FILE SYSTEM                           │
│ Kernel (Ring 0) ◄── System Calls ──► User Mode (Ring 3)     │
│ File Systems: Root (/ or C:\), Absolute vs Relative Paths   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. THE EXECUTION LIFECYCLE                                  │
│ Code ──► Binary ──► Process ──► Thread ──► CPU & Memory     │
│ Memory Segments: [ Stack | Heap | Data | Code ]             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. DATA REPRESENTATION                                      │
│ Bits & Bytes ──► Binary / Hex ──► Integers (Two's Comp)     │
│ IEEE 754 Floating Point ──► ASCII & UTF-8                   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. COMMAND LINE MASTERY (CLI)                               │
│ Terminal & Shells (Bash/PowerShell) ──► File Ops ──► Pipes   │
│ Redirection ──► Environment Variables ($PATH)               │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. LABS & TECHNICAL INTERVIEW PREPARATION                   │
│ System Monitoring Labs ──► Code Demos ──► 10 Core Q&A       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là các chuyên đề học tập đã được biên soạn chi tiết và đầy đủ trong thư mục này:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Kiến Trúc Phần Cứng Máy Tính** | Mô hình Von Neumann, CPU (ALU, CU, Registers, Cache L1/L2/L3), RAM, SSD NVMe vs HDD, Phân cấp bộ nhớ, Khác biệt CPU vs GPU. | [Xem bài viết](./01-hardware-architecture.md) |
| **02** | **Hệ Điều Hành & File System** | Vai trò của OS, Kiến trúc Kernel, Phân tầng bảo vệ User Mode vs Kernel Mode, System Calls, Cây thư mục Windows vs Linux, Phân quyền file (rwx). | [Xem bài viết](./02-operating-system-basics.md) |
| **03** | **Vòng Đời Thực Thi Chương Trình** | Program → Process → Thread, Bố cục bộ nhớ tiến trình (Text, Data, Heap, Stack), So sánh sâu Stack vs Heap, CPU Fetch-Decode-Execute, Context Switch, Virtual Memory & Paging. | [Xem bài viết](./03-program-to-process-thread.md) |
| **04** | **Biểu Diễn Dữ Liệu Trong Máy Tính** | Đơn vị Bit/Byte (SI vs IEC), Hệ đếm Binary, Decimal, Hexadecimal, Số nguyên có dấu & Bù 2, Số thực IEEE 754 & giải thích `0.1 + 0.2 != 0.3`, Bảng mã ký tự ASCII, Unicode và chuẩn UTF-8, Endianness. | [Xem bài viết](./04-data-representation.md) |
| **05** | **Làm Chủ Dòng Lệnh (CLI)** | Phân biệt Terminal vs Shell, Bảng đối chiếu lệnh PowerShell / CMD vs Linux Bash, Thao tác tệp và thư mục, Pipeline (`\|`), Điều hướng (`>`, `>>`), Biến môi trường `$PATH`, Phím tắt năng suất. | [Xem bài viết](./05-command-line-cli.md) |
| **06** | **Thực Hành & Ôn Tập Phỏng Vấn** | Lab quan sát tài nguyên (Task Manager / htop), Thử thách tạo dự án bằng CLI trong 5 phút, Thí nghiệm code Stack Overflow & CPU Cache Locality, Bộ 10 câu hỏi phỏng vấn kỹ thuật có đáp án chi tiết. | [Xem bài viết](./06-practice-and-questions.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 0

Sau khi học xong 6 chuyên đề trên, bạn sẽ:
1. **Không còn bỡ ngỡ với khái niệm hệ thống:** Tự tin trả lời mạch lạc các câu hỏi phỏng vấn hóc búa về Process, Thread, Stack, Heap, Virtual Memory.
2. **Hiểu bản chất của mã nguồn:** Biết được mỗi biến bạn khai báo sẽ nằm ở đâu trong RAM (Stack hay Heap), dung lượng bao nhiêu byte và được CPU nạp như thế nào.
3. **Thành thạo công cụ làm việc hàng ngày:** Không còn ngại ngùng khi mở Terminal, tự tin thao tác và làm việc trên các môi trường Linux/Cloud sau này.
4. **Viết code tối ưu hơn:** Hiểu được tại sao phải tránh rò rỉ bộ nhớ (Memory Leak), tại sao không nên đệ quy quá sâu (Stack Overflow), và tại sao duyệt mảng theo hàng lại nhanh hơn theo cột.

---

## ⏭️ Bước Tiếp Theo
Sau khi hoàn thành toàn bộ bài tập và checklist trong [06-practice-and-questions.md](./06-practice-and-questions.md), bạn đã sẵn sàng tiến lên:
👉 **[Level 1: Programming Fundamentals](../01-programming-fundamentals/README.md)**
