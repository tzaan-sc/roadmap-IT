# 05 - Làm Chủ Dòng Lệnh (Command Line Interface - CLI)

> **Mục tiêu bài học:** Chuyển dịch tư duy từ người dùng giao diện đồ họa (GUI) thông thường sang một lập trình viên chuyên nghiệp: hiểu rõ bản chất của Terminal & Shell, làm chủ các lệnh thao tác tệp, quản lý tiến trình, đường ống dẫn dữ liệu (Pipes & Redirection) và hiểu thấu đáo biến môi trường `PATH`.

---

## 1. Tại Sao Lập Trình Viên Bắt Buộc Phải Thành Thạo CLI?

Nhiều người mới học thường tự hỏi: *"Tại sao phải gõ lệnh đen trắng phức tạp trong khi dùng chuột click rất trực quan?"*

1. **Quản trị máy chủ từ xa (Remote Server):** Các server đám mây (AWS, GCP, DigitalOcean) hầu như **không cài giao diện đồ họa** để tiết kiệm tài nguyên và bảo mật. Bạn chỉ có thể kết nối qua SSH và điều khiển bằng dòng lệnh.
2. **Tốc độ và Năng suất:** Tạo 100 thư mục, đổi tên 1,000 file hay lọc 5 triệu dòng log: dùng chuột click có thể mất cả ngày, nhưng dùng một dòng lệnh CLI chỉ mất đúng 2 giây.
3. **Tự động hóa (Automation & CI/CD):** Dòng lệnh có thể viết thành các script (`.sh`, `.bat`, `.ps1`) để tự động hóa quá trình test, build và deploy phần mềm liên tục mà không cần con người can thiệp.
4. **Hệ sinh thái công cụ phát triển:** Hầu hết các công cụ lập trình hiện đại như Git, Docker, Kubernetes, npm, pip, cargo, go... đều được thiết kế ưu tiên chạy trên CLI trước khi có các tiện ích GUI.

---

## 2. Phân Biệt: Terminal vs Shell vs Console

Người mới bắt đầu thường hay nhầm lẫn 3 khái niệm này:

```text
┌────────────────────────────────────────────────────────┐
│ TERMINAL (Cửa sổ ứng dụng hiển thị đồ họa)            │
│ Ví dụ: Windows Terminal, iTerm2, Alacritty, GNOME Term │
│                                                        │
│   ┌──────────────────────────────────────────────────┐ │
│   │ SHELL (Bộ thông dịch dòng lệnh - Command Runner) │ │
│   │ Ví dụ: Bash, Zsh, PowerShell, Command Prompt     │ │
│   │                                                  │ │
│   │   $ curl -I https://google.com                   │ │
│   │   (Nhận lệnh -> Tìm file binary -> Chạy -> Trả về) │
│   └──────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────┘
```

- **Terminal:** Là cửa sổ đồ họa chịu trách nhiệm nhận phím gõ từ bàn phím và vẽ các ký tự màu sắc lên màn hình.
- **Shell:** Là chương trình nằm bên trong, chịu trách nhiệm nhận văn bản từ Terminal, phân tích cú pháp (parse), gọi Kernel chạy chương trình tương ứng và trả kết quả về cho Terminal.
  - Các Shell phổ biến trên Linux/macOS: **Bash** (tiêu chuẩn), **Zsh** (hiện đại, thẩm mỹ cao), **Fish**.
  - Các Shell trên Windows: **PowerShell** (rất mạnh, hướng đối tượng), **CMD** (Command Prompt - cũ).

---

## 3. Bảng Đối Chiếu Lệnh: Windows vs Linux/macOS

Dưới đây là bảng tổng hợp các lệnh cốt lõi mà lập trình viên sử dụng hàng ngày:

### 3.1. Điều hướng thư mục & Xem vị trí

| Thao tác | Linux / macOS (Bash) | Windows (PowerShell) | Windows (CMD) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| Xem thư mục hiện tại | `pwd` | `pwd` hoặc `Get-Location` | `cd` | In ra đường dẫn đầy đủ nơi đang đứng |
| Liệt kê file và folder | `ls` hoặc `ls -la` | `ls` hoặc `dir` | `dir` | `-la` để hiện cả file ẩn và phân quyền |
| Chuyển thư mục | `cd <path>` | `cd <path>` | `cd <path>` | Chuyển tới thư mục chỉ định |
| Lên thư mục cha | `cd ..` | `cd ..` | `cd ..` | Lùi lại 1 cấp thư mục |
| Về thư mục Home | `cd ~` | `cd ~` | `cd %USERPROFILE%` | Về thư mục cá nhân của user |
| Vẽ cây thư mục | `tree` | `tree` | `tree` | Hiển thị cấu trúc cây trực quan |

### 3.2. Thao tác tệp tin và thư mục

| Thao tác | Linux / macOS (Bash) | Windows (PowerShell) | Windows (CMD) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| Tạo thư mục | `mkdir <name>` | `mkdir <name>` | `mkdir <name>` | Tạo thư mục mới |
| Tạo thư mục lồng nhau | `mkdir -p a/b/c` | `mkdir -p a/b/c` | `mkdir a\b\c` | Tạo thư mục cha nếu chưa có |
| Tạo file rỗng | `touch app.js` | `New-Item app.js` | `type nul > app.js` | Tạo file mới nhanh chóng |
| Sao chép file | `cp file.txt copy.txt` | `cp file.txt copy.txt` | `copy file.txt copy.txt` | Sao chép một file |
| Sao chép thư mục | `cp -r dir1 dir2` | `cp -r dir1 dir2` | `xcopy /e dir1 dir2` | `-r` là recursive (đệ quy cả bên trong) |
| Đổi tên / Di chuyển | `mv old.txt new.txt` | `mv old.txt new.txt` | `move old.txt new.txt` | Di chuyển hoặc đổi tên |
| Xóa file | `rm file.txt` | `rm file.txt` | `del file.txt` | Xóa file vĩnh viễn (không vào Thùng rác!) |
| Xóa thư mục có file | `rm -rf <dir>` | `rm -r -Force <dir>` | `rmdir /s /q <dir>` | **Cẩn trọng tuyệt đối với lệnh này** |

> [!CAUTION]
> Lệnh xóa bằng dòng lệnh (`rm` hoặc `del`) sẽ **xóa vĩnh viễn dữ liệu** mà không đưa vào Thùng rác (Recycle Bin / Trash). Đừng bao giờ chạy lệnh `rm -rf /` trên Linux vì nó sẽ xóa sạch toàn bộ hệ điều hành!

### 3.3. Xem và tìm kiếm nội dung file

| Thao tác | Linux / macOS (Bash) | Windows (PowerShell) | Windows (CMD) | Giải thích |
| :--- | :--- | :--- | :--- | :--- |
| Đọc toàn bộ file | `cat file.txt` | `cat file.txt` / `Get-Content` | `type file.txt` | In toàn bộ nội dung file ra màn hình |
| Xem n dòng đầu file | `head -n 10 file.txt` | `Get-Content file.txt -Head 10` | *(không có sẵn)* | Xem phần mở đầu |
| Xem n dòng cuối file | `tail -n 10 file.txt` | `Get-Content file.txt -Tail 10` | *(không có sẵn)* | Thường dùng xem log ứng dụng |
| Xem log realtime | `tail -f app.log` | `Get-Content app.log -Wait` | *(không có sẵn)* | Tự cuộn khi có dòng log mới ghi vào |
| Tìm kiếm chuỗi | `grep "ERROR" app.log` | `Select-String "ERROR" app.log` | `findstr "ERROR" app.log`| Lọc các dòng chứa từ khóa |

### 3.4. Quản lý tiến trình & Mạng

| Thao tác | Linux / macOS (Bash) | Windows (PowerShell) | Windows (CMD) |
| :--- | :--- | :--- | :--- |
| Xem tiến trình đang chạy | `ps aux` hoặc `top` / `htop` | `Get-Process` | `tasklist` |
| Tắt tiến trình theo PID | `kill -9 <PID>` | `Stop-Process -Id <PID>` | `taskkill /F /PID <PID>` |
| Tắt tiến trình theo tên | `killall node` | `Stop-Process -Name node` | `taskkill /F /IM node.exe` |
| Kiểm tra kết nối mạng | `ping google.com` | `ping google.com` | `ping google.com` |
| Xem địa chỉ IP máy | `ip a` hoặc `ifconfig` | `ipconfig` | `ipconfig` |
| Gửi request HTTP | `curl -I https://api.com` | `curl -I https://api.com` | `curl -I https://api.com` |
| Xem các cổng đang mở | `netstat -tulnp` | `Get-NetTCPConnection` | `netstat -ano` |

---

## 4. Dòng Chảy Dữ Liệu: Standard I/O, Redirection & Pipes

Mọi chương trình chạy trong Shell đều mặc định mở 3 luồng giao tiếp tiêu chuẩn:
- **`stdin` (Standard Input - Kênh 0):** Dữ liệu đầu vào (mặc định từ bàn phím).
- **`stdout` (Standard Output - Kênh 1):** Kết quả in ra bình thường (mặc định ra màn hình).
- **`stderr` (Standard Error - Kênh 2):** Thông báo lỗi (mặc định ra màn hình).

```text
               ┌───────────────────────┐
  stdin (0) ──►│                       ├──► stdout (1)
(Bàn phím)     │  CHƯƠNG TRÌNH / LỆNH  │    (Màn hình terminal)
               │                       ├──► stderr (2)
               └───────────────────────┘    (Màn hình hiển thị lỗi)
```

### 4.1. Điều hướng xuất nhập (Redirection: `>`, `>>`, `<`)

- **`>` (Ghi đè file):** Chuyển kết quả từ màn hình vào một file. Nếu file đã có dữ liệu, nó sẽ bị xóa và ghi lại từ đầu:
  ```bash
  echo "Hello World" > output.txt
  ```
- **`>>` (Ghi nối tiếp vào đuôi file):** Thêm dữ liệu mới vào cuối file mà không làm mất nội dung cũ:
  ```bash
  echo "Dòng tiếp theo" >> output.txt
  ```
- **`<` (Đọc dữ liệu từ file vào lệnh):**
  ```bash
  sort < names.txt
  ```

### 4.2. Đường ống (Pipe: `|`) - Sức mạnh vô tận của CLI

Dấu Pipe `|` cho phép lấy **`stdout` của lệnh phía trước làm `stdin` cho lệnh phía sau**. Bạn có thể ghép nhiều lệnh đơn giản lại với nhau để giải quyết bài toán phức tạp mà không cần viết script code dài dòng:

```bash
# Đọc file log -> Lọc các dòng có chữ ERROR -> Đếm xem có bao nhiêu lỗi:
cat server.log | grep "ERROR" | wc -l

# Xem danh sách tiến trình -> Lọc các tiến trình nodejs:
ps aux | grep node
```

---

## 5. Biến Môi Trường (Environment Variables) & Bí Mật Của Biến `PATH`

### 5.1. Biến môi trường là gì?
Là các cặp `KEY=VALUE` được lưu trong bộ nhớ của hệ điều hành, giúp các ứng dụng và script đọc được cấu hình hệ thống mà không cần fix cứng vào code.
- Xem biến môi trường:
  - Linux/Mac: `printenv` hoặc `echo $USER`
  - Windows PowerShell: `Get-ChildItem Env:` hoặc `echo $env:USERNAME`

### 5.2. Biến `PATH` hoạt động như thế nào?
Khi bạn gõ lệnh `python` hay `git` trong Terminal, làm sao hệ điều hành biết file thực thi `python.exe` hay `git.exe` nằm ở đâu để chạy?

1. Hệ điều hành đọc danh sách các đường dẫn thư mục được phân cách bởi dấu chấm phẩy `;` (Windows) hoặc dấu hai chấm `:` (Linux) nằm trong biến môi trường tên là **`PATH`**.
2. Nó kiểm tra tuần tự từng thư mục trong danh sách từ trái sang phải:
   - Thư mục 1: `C:\Windows\system32` (không có file `python.exe`)
   - Thư mục 2: `C:\Program Files\Git\bin` (không có file `python.exe`)
   - Thư mục 3: `C:\Python312` (tìm thấy `python.exe`!) → Khởi chạy ngay lập tức.
3. Nếu quét hết các thư mục trong `PATH` mà không thấy, hệ điều hành sẽ báo lỗi kinh điển:
   `'python' is not recognized as an internal or external command` (trên Windows) hoặc `command not found` (trên Linux).

> [!TIP]
> Mỗi khi bạn cài một ngôn ngữ mới (Node.js, Go, Python, Rust) mà Terminal không nhận lệnh, **99% nguyên nhân là đường dẫn tới thư mục `bin` của phần mềm đó chưa được thêm vào biến `PATH`**.

---

## 6. Các Phím Tắt Năng Suất (Cheatsheet)

| Phím tắt | Tác dụng |
| :--- | :--- |
| **`Tab`** | Tự động điền nốt tên lệnh hoặc đường dẫn file/thư mục (Auto-complete). Bấm 2 lần để xem gợi ý |
| **`Ctrl + C`** | Ngắt và hủy ngay lập tức lệnh/tiến trình đang chạy vô hạn |
| **`Ctrl + L`** | Xóa sạch màn hình Terminal (tương đương lệnh `clear` trên Linux hoặc `cls` trên Windows) |
| **`Mũi tên Lên / Xuống`** | Duyệt lại lịch sử các lệnh vừa gõ trước đó |
| **`Ctrl + R`** | Tìm kiếm thông minh ngược lại trong lịch sử lệnh (Reverse Search) |
| **`Ctrl + A` / `Ctrl + E`** | Nhảy nhanh con trỏ về đầu dòng / cuối dòng lệnh (trên Bash/Zsh) |

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **CLI** là công cụ sống còn để quản trị server, chạy Git, build dự án và tự động hóa.
2. Nắm vững nhóm lệnh cơ bản: điều hướng (`cd`, `ls/dir`), thao tác file (`mkdir`, `touch`, `cp`, `mv`, `rm`).
3. Sử dụng `>` để ghi đè, `>>` để ghi nối tiếp và `|` (Pipe) để kết nối dòng dữ liệu giữa các lệnh.
4. Hiểu bản chất biến **`PATH`**: danh sách thư mục mà OS sẽ lùng sục khi bạn gọi một lệnh từ bất kỳ đâu.
