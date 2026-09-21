# 02 - Hệ Điều Hành & Hệ Thống Tệp Tin (Operating System & File System)

> **Mục tiêu bài học:** Hiểu được vai trò của Hệ điều hành (OS), cơ chế phân quyền phần cứng (User Mode vs Kernel Mode), các hàm gọi hệ thống (System Calls), so sánh các họ OS (Windows, Linux, macOS) và làm chủ cấu trúc tệp tin (File System).

---

## 1. Hệ Điều Hành (Operating System - OS) Là Gì?

Hệ điều hành là phần mềm hệ thống quan trọng nhất trên máy tính. Nó đóng vai trò là **người quản gia trung gian** đứng giữa phần cứng vật lý và các ứng dụng người dùng.

```text
┌─────────────────────────────────────────────────────────────┐
│                 ỨNG DỤNG NGƯỜI DÙNG (APPS)                  │
│       (Web Browser, VS Code, Game, Python script...)        │
└──────────────────────────────┬──────────────────────────────┘
                               │ Gọi hàm (API / System Calls)
┌──────────────────────────────▼──────────────────────────────┐
│                    HỆ ĐIỀU HÀNH (OS)                        │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │                         KERNEL                          │ │
│ │  ┌───────────────┐ ┌────────────────┐ ┌───────────────┐ │ │
│ │  │ CPU Scheduler │ │ Memory Manager │ │ File System   │ │ │
│ │  ├───────────────┤ ├────────────────┤ ├───────────────┤ │ │
│ │  │ Device Driver │ │ Network Stack  │ │ Security/Perm │ │ │
│ │  └───────────────┘ └────────────────┘ └───────────────┘ │ │
│ └─────────────────────────────────────────────────────────┘ │
└──────────────────────────────┬──────────────────────────────┘
                               │ Tương tác trực tiếp
┌──────────────────────────────▼──────────────────────────────┐
│                    PHẦN CỨNG VẬT LÝ                         │
│             (CPU, RAM, Ổ cứng, Card mạng, GPU)              │
└─────────────────────────────────────────────────────────────┘
```

### 3 Nhiệm vụ cốt lõi của OS:
1. **Quản lý tài nguyên (Resource Management):** Chia sẻ thời gian CPU (CPU Scheduling), phân bổ RAM, kiểm soát băng thông mạng và đĩa cứng cho hàng trăm tiến trình cùng lúc một cách công bằng.
2. **Trừu tượng hóa phần cứng (Hardware Abstraction):** Lập trình viên không cần biết ổ cứng của hãng nào, giao tiếp qua chân cắm nào. Bạn chỉ cần gọi hàm `open("file.txt")`, OS sẽ tự xử lý việc đọc từng byte từ đĩa.
3. **Bảo vệ và Cách ly (Protection & Isolation):** Ngăn không cho một ứng dụng bị lỗi (như trình duyệt bị sập) phá hỏng toàn bộ hệ thống hoặc truy cập lén lút vào dữ liệu của ứng dụng khác.

---

## 2. Kernel, Chế Độ Bảo Vệ (Ring Levels) & System Calls

### 2.1. Kernel là gì?
**Kernel (Nhân hệ điều hành)** là phần code cốt lõi chạy thường trực trong RAM từ khi bật máy đến khi tắt. Kernel có toàn quyền kiểm soát tuyệt đối với phần cứng.

- **Monolithic Kernel (Nhân nguyên khối - Linux, Windows NT):** Tất cả dịch vụ (Scheduler, Memory, File system, Drivers) đều chạy chung trong cùng một không gian bộ nhớ của Kernel. Tốc độ cực nhanh.
- **Microkernel (Nhân vi mô - ví dụ QNX, Minix):** Chỉ giữ các chức năng tối thiểu trong Kernel; các dịch vụ khác (Drivers, File system) chạy như ứng dụng thông thường. Độ an toàn cao hơn nhưng đánh đổi hiệu năng.

### 2.2. Phân tách User Mode vs Kernel Mode

Để ngăn chặn lỗi phần mềm phá hoại phần cứng, CPU hỗ trợ các vòng bảo vệ (**Protection Rings**):

```text
       [ Ring 3: User Mode ]
     (Ứng dụng người dùng, Game, Browser)
                   │
                   ▼ (System Call)
       [ Ring 0: Kernel Mode ]
     (Toàn quyền truy cập phần cứng & RAM)
```

- **User Mode (Chế độ người dùng - Ring 3):**
  - Mọi ứng dụng thông thường của bạn chạy ở chế độ này.
  - Ứng dụng **không được phép** tự ý giao tiếp trực tiếp với card mạng, không được ghi đè trực tiếp lên RAM của tiến trình khác, không được truy cập trực tiếp vào đĩa cứng.
- **Kernel Mode (Chế độ nhân / Đặc quyền - Ring 0):**
  - Chỉ có mã của Kernel và Device Drivers mới được chạy ở đây.
  - Được phép thực thi mọi tập lệnh CPU và truy cập vào bất kỳ địa chỉ bộ nhớ vật lý nào.

### 2.3. System Calls (Syscalls) - Cổng Giao Tiếp An Toàn

Khi một ứng dụng chạy ở User Mode muốn làm việc với phần cứng (ví dụ: ghi file ra đĩa, gửi dữ liệu qua mạng), nó **bắt buộc phải thực hiện System Call**:

```text
[Ứng dụng: C code] printf("Hello!")
       │
       ▼
[C Runtime Lib - glibc] hàm write()
       │
       ▼ (CPU Software Interrupt: chuyển sang Ring 0)
[Kernel] System Call: sys_write()
       │
       ▼ (Gửi tín hiệu tới driver màn hình)
[Màn hình] Hiển thị chữ "Hello!" lên Terminal
```

Một số System Call phổ biến trên Linux:
- `fork()`: Tạo một tiến trình con mới.
- `exec()`: Nạp và chạy một chương trình khác.
- `open()`, `read()`, `write()`, `close()`: Thao tác tệp tin.
- `socket()`, `connect()`, `bind()`: Thao tác kết nối mạng.

---

## 3. So Sánh Các Hệ Điều Hành Phổ Biến

| Tiêu chí | Linux | Windows | macOS |
| :--- | :--- | :--- | :--- |
| **Nhân (Kernel)** | Linux Kernel (Mã nguồn mở) | Windows NT (Đóng mã nguồn) | XNU / Darwin (Dựa trên BSD/Unix) |
| **Triết lý thiết kế** | Mọi thứ đều là file ("Everything is a file"), tối ưu tự động hóa qua CLI | Trải nghiệm đồ họa (GUI) thân thiện, tương thích ngược cực cao | Trực quan, tối ưu tuyệt đối cho phần cứng Apple, chuẩn UNIX |
| **Thị phần Server/Cloud** | **> 90%** các máy chủ Internet, Docker, Kubernetes chạy Linux | Phổ biến trong doanh nghiệp dùng hệ sinh thái Active Directory, .NET | Rất hiếm khi dùng làm máy chủ sản xuất (production server) |
| **Thị phần máy tính cá nhân** | ~3% (Phù hợp dân kỹ thuật/Dev) | ~70% (Phổ thông nhất, Game, Văn phòng) | ~15-20% (Rất được giới Developer & Designer ưa chuộng) |
| **Quy ước đường dẫn** | Dùng gạch chéo `/` (case-sensitive) | Dùng gạch chéo ngược `\` (case-insensitive) | Dùng gạch chéo `/` (case-preserving) |

> [!IMPORTANT]
> **Tại sao Lập trình viên bắt buộc phải học Linux?**
> Mặc dù bạn có thể dùng máy cá nhân Windows hay Mac để viết code, nhưng khi ứng dụng được đưa lên môi trường thật (Production, Cloud AWS/GCP, Docker), 95% chúng sẽ chạy trên nền tảng **Linux**.

---

## 4. Hệ Thống Tệp Tin (File System)

File System là phương thức và cấu trúc dữ liệu mà Hệ điều hành dùng để tổ chức, lưu trữ, đặt tên và truy xuất các tệp tin trên ổ cứng.

### 4.1. Cấu trúc cây thư mục: Windows vs Linux

```text
WINDOWS (Theo ổ đĩa)                     LINUX / macOS (Một cây duy nhất từ gốc)
C:\ (Ổ đĩa cài OS)                       / (Root directory)
├── Program Files                        ├── bin    (Các lệnh thực thi cơ bản)
├── Users                                ├── etc    (File cấu hình hệ thống)
│   └── Lenovo                           ├── home   (Thư mục người dùng)
│       └── Documents                    │   └── ubuntu
└── Windows                              ├── var    (Logs, dữ liệu thay đổi liên tục)
D:\ (Ổ đĩa dữ liệu)                      ├── dev    (Các thiết bị phần cứng dạng file)
└── Học code                             └── mnt    (Nơi gắn ổ đĩa ngoài / phân vùng)
    └── roadmap-IT
```

### 4.2. Đường dẫn Tuyệt đối vs Đường dẫn Tương đối

- **Đường dẫn tuyệt đối (Absolute Path):**
  - Đi từ thư mục gốc của hệ thống (`/` trên Linux hoặc `C:\` trên Windows).
  - Không phụ thuộc vào vị trí bạn đang đứng.
  - Ví dụ:
    - Windows: `D:\Học code\roadmap-IT\README.md`
    - Linux: `/home/user/projects/roadmap-IT/README.md`
- **Đường dẫn tương đối (Relative Path):**
  - Đi từ thư mục làm việc hiện tại (**Current Working Directory - CWD**).
  - Ký hiệu đặc biệt:
    - `.` : Đại diện cho thư mục hiện tại.
    - `..`: Đại diện cho thư mục cha (thư mục cấp trên).
  - Ví dụ: Nếu đang đứng ở `00-computer-fundamentals`:
    - `./README.md` trỏ tới file README trong thư mục này.
    - `../01-programming-fundamentals` trỏ sang thư mục ngang hàng cấp trên.

### 4.3. Phân quyền tệp tin trên Linux (File Permissions)

Trên Linux, mỗi file và thư mục đều gắn liền với quyền hạn cho 3 đối tượng:
1. **User (u):** Chủ sở hữu file.
2. **Group (g):** Nhóm người dùng có quyền.
3. **Others (o):** Tất cả những người dùng khác trong hệ thống.

Mỗi đối tượng có 3 quyền cơ bản:
- `r` (Read - Đọc): Giá trị nhị phân = 4.
- `w` (Write - Ghi / Sửa / Xóa): Giá trị nhị phân = 2.
- `x` (Execute - Thực thi như 1 chương trình): Giá trị nhị phân = 1.

```text
Ví dụ lệnh: ls -l myfile.sh
Kết quả:
- r w x r - x r - -
┬ └───┘ └───┘ └───┘
│   │     │     │
│   │     │     └── Others: chỉ đọc (r--) = 4
│   │     └──────── Group: đọc & chạy (r-x) = 4 + 1 = 5
│   └────────────── User: đọc, ghi & chạy (rwx) = 4 + 2 + 1 = 7
└────────────────── Loại: '-' là file thông thường ('d' là directory)
```
=> Quyền của file trên là: **`754`**.

### 4.4. Tệp văn bản (Text File) vs Tệp nhị phân (Binary File)

- **Text File (`.txt`, `.py`, `.json`, `.html`):**
  - Chứa các chuỗi ký tự có thể đọc được bằng mắt người (mỗi ký tự được ánh xạ bằng mã ASCII/UTF-8).
  - Có các ký tự ngắt dòng: `\n` (LF - Linux/Mac) hoặc `\r\n` (CRLF - Windows).
- **Binary File (`.exe`, `.jpg`, `.mp4`, `.zip`, `.dll`):**
  - Chứa các chuỗi byte tùy ý được cấu trúc theo format riêng mà chỉ phần mềm tương ứng mới giải mã được.
  - Nếu mở binary file bằng Notepad, bạn sẽ chỉ thấy các ký tự rác vô nghĩa.

---

## 5. Tóm Tắt & Ghi Nhớ Nhanh

1. **Hệ điều hành** quản lý tài nguyên, bảo vệ hệ thống và cung cấp giao diện trừu tượng để lập trình viên tương tác với phần cứng.
2. **Kernel** chạy ở **Ring 0 (Kernel Mode)** với quyền hạn cao nhất; ứng dụng chạy ở **Ring 3 (User Mode)**.
3. Ứng dụng muốn đọc file, mở mạng hay tạo tiến trình đều phải dùng **System Calls**.
4. **Linux** là nền tảng tối quan trọng mà mọi kỹ sư IT / Backend / DevOps đều cần thành thạo.
5. Cần phân biệt rõ đường dẫn tương đối (`.`, `..`) và tuyệt đối để tránh lỗi kinh điển khi cấu hình server hoặc deploy dự án.
