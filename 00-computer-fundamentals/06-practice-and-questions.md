# 06 - Bài Tập Thực Hành & Bộ Câu Hỏi Ôn Tập Phỏng Vấn

> **Mục tiêu bài học:** Chuyển hóa toàn bộ lý thuyết về phần cứng, hệ điều hành, tiến trình và dòng lệnh thành kỹ năng thực chiến thông qua các bài lab quan sát hệ thống thực tế, thử thách gõ lệnh CLI và bộ câu hỏi phỏng vấn có lời giải chi tiết.

---

## PHẦN 1: CÁC BÀI LAB THỰC HÀNH HỆ THỐNG

### LAB 1: Khảo Sát "Nội Tạng" Máy Tính Bằng Công Cụ Hệ Thống

#### Mục tiêu:
Nhìn thấy tận mắt các khái niệm: CPU Cores/Threads, Phân cấp bộ nhớ RAM/Pagefile, Process ID, Handles và Threads đang chạy trên chính máy tính của bạn.

#### Hướng dẫn thực hiện trên Windows:
1. Bấm tổ hợp phím **`Ctrl + Shift + Esc`** để mở **Task Manager**. (Nếu đang thu gọn, bấm *More details*).
2. Chuyển sang tab **Performance (Hiệu năng)**:
   - Chọn mục **CPU**:
     - Xem số nhân vật lý (**Cores**) và số luồng logic (**Logical processors**).
     - Xem xung nhịp gốc (**Base speed**) so với xung nhịp thực tế đang chạy.
     - Xem dung lượng các tầng đệm: **L1 cache, L2 cache, L3 cache**.
   - Chọn mục **Memory (Bộ nhớ)**:
     - Xem dung lượng RAM vật lý: Đang dùng (**In use**), Còn trống (**Available**), Đang đệm (**Cached**).
     - Nhìn dòng **Committed**: Ví dụ `12.5 / 24.0 GB`. Đây chính là tổng không gian bộ nhớ ảo (RAM vật lý + Kích thước file ảo hóa `pagefile.sys` trên ổ cứng SSD).
3. Chuyển sang tab **Details**:
   - Nhấp chuột phải vào thanh tiêu đề cột → Bấm **Select columns** → Tích chọn thêm:
     - **PID** (Process ID)
     - **Threads** (Số luồng đang chạy trong tiến trình)
     - **Handles** (Số tài nguyên OS như file, socket mà tiến trình đang mở)
     - **Working set (memory)** (Dung lượng RAM vật lý thực tế tiến trình chiếm giữ)
   - Quan sát xem trình duyệt Chrome / Edge đang chạy bao nhiêu tiến trình (Processes) và mỗi tiến trình có bao nhiêu Threads!

#### Hướng dẫn thực hiện trên Linux / macOS / WSL:
Mở Terminal và gõ:
```bash
# Xem thông tin chi tiết CPU:
lscpu

# Xem dung lượng RAM và Swap còn trống (định dạng MB/GB dễ đọc):
free -h

# Mở trình giám sát trực quan tương tác:
top
# Hoặc cài công cụ đẹp mắt hơn (nếu có):
htop
```

---

### LAB 2: Thử Thách CLI 5 Phút - Xây Dựng Dự Án Từ Dòng Lệnh

#### Đề bài:
Không sử dụng chuột hay giao diện File Explorer, hãy dùng Terminal để tạo một cấu trúc dự án web gồm các thư mục và file sau:

```text
my-web-project/
├── index.html
├── assets/
│   ├── css/
│   │   └── style.css
│   └── js/
│       └── main.js
└── README.md
```

#### Lời giải chi tiết:

##### Cách làm trên Linux / macOS / Git Bash:
```bash
# 1. Tạo thư mục dự án và các thư mục con lồng nhau cùng lúc
mkdir -p my-web-project/assets/css my-web-project/assets/js

# 2. Di chuyển vào thư mục dự án
cd my-web-project

# 3. Tạo các file rỗng
touch index.html README.md assets/css/style.css assets/js/main.js

# 4. Ghi nội dung vào README.md
echo "# Dự Án Web Của Tôi" > README.md
echo "Được khởi tạo bằng CLI hoàn toàn!" >> README.md

# 5. Kiểm tra kết quả
cat README.md
tree
```

##### Cách làm trên Windows PowerShell:
```powershell
# 1. Tạo thư mục lồng nhau
mkdir -p my-web-project/assets/css, my-web-project/assets/js

# 2. Di chuyển vào thư mục dự án
cd my-web-project

# 3. Tạo các file
New-Item index.html, README.md, assets/css/style.css, assets/js/main.js

# 4. Ghi nội dung vào README.md
Set-Content README.md "# Dự Án Web Của Tôi"
Add-Content README.md "Được khởi tạo bằng CLI hoàn toàn!"

# 5. Kiểm tra kết quả
Get-Content README.md
tree /F
```

---

### LAB 3: Thí Nghiệm Code Minh Họa Bản Chất Hệ Thống

#### Thí nghiệm 1: Gây ra lỗi tràn Stack (Stack Overflow)
Chạy thử đoạn code Python dưới đây để thấy đệ quy làm cạn kiệt Stack Frame như thế nào:

```python
# stack_overflow_demo.py
count = 0

def recursive_function():
    global count
    count += 1
    # Mỗi lần hàm tự gọi lại chính nó, 1 Stack Frame mới được đẩy vào Stack
    # mà không hề được giải phóng.
    recursive_function()

try:
    recursive_function()
except RecursionError as e:
    print(f"Đã chạm ngưỡng giới hạn Stack tại tầng gọi thứ: {count}")
    print(f"Lỗi: {e}")
```

#### Thí nghiệm 2: Sức mạnh của CPU Cache Hit (Spatial Locality)
Tại sao duyệt mảng 2 chiều theo hàng (Row-major) lại nhanh hơn duyệt theo cột (Column-major) gấp nhiều lần?

```python
# cache_locality_demo.py
import time

SIZE = 5000
# Tạo ma trận 5000 x 5000 số nguyên
matrix = [[1] * SIZE for _ in range(SIZE)]

# Cách 1: Duyệt theo hàng (Dữ liệu nằm liền kề nhau trong RAM -> Cache Hit liên tục)
start = time.time()
total_row = 0
for r in range(SIZE):
    for c in range(SIZE):
        total_row += matrix[r][c]
print(f"Duyệt theo hàng (Cache-friendly): {time.time() - start:.4f} giây")

# Cách 2: Duyệt theo cột (Nhảy cóc ô nhớ -> Cache Miss liên tục, CPU phải chờ RAM)
start = time.time()
total_col = 0
for c in range(SIZE):
    for r in range(SIZE):
        total_col += matrix[r][c]
print(f"Duyệt theo cột (Cache-unfriendly): {time.time() - start:.4f} giây")
```
*Kết quả:* Cách 1 thường nhanh hơn Cách 2 từ **3 đến 5 lần**, dù số phép tính hoàn toàn giống hệt nhau! Đó chính là sức mạnh của việc thấu hiểu phần cứng và CPU Cache.

---

## PHẦN 2: BỘ CÂU HỎI ÔN TẬP & PHỎNG VẤN CỐT LÕI (CÓ LỜI GIẢI)

### Câu 1: Phân biệt sự khác nhau giữa Process và Thread?
- **Trả lời:**
  - **Process (Tiến trình)** là một chương trình đang thực thi, sở hữu không gian địa chỉ bộ nhớ (RAM) độc lập, có PID riêng, việc tạo mới hoặc chuyển đổi tốn nhiều tài nguyên. Lỗi ở một Process thường không làm sập Process khác.
  - **Thread (Luồng)** là một nhánh thực thi nằm bên trong Process. Tất cả các Thread trong cùng 1 Process dùng chung không gian bộ nhớ Heap, biến toàn cục và tài nguyên mở, nhưng mỗi Thread có Call Stack và thanh ghi riêng. Chi phí tạo Thread rẻ hơn nhiều, nhưng nếu một Thread phát sinh lỗi bộ nhớ nghiêm trọng có thể kéo sập toàn bộ Process.

### Câu 2: Bộ nhớ Stack và Heap khác nhau như thế nào? Khi nào dữ liệu nằm ở Stack, khi nào ở Heap?
- **Trả lời:**
  - **Stack:** Có cấu trúc LIFO (vào sau ra trước), được CPU quản lý trực tiếp bằng thanh ghi con trỏ ngăn xếp nên tốc độ truy xuất cực nhanh. Dùng để chứa các biến cục bộ nguyên thủy, tham số hàm và địa chỉ trả về của hàm. Kích thước Stack nhỏ cố định và tự động giải phóng khi hàm kết thúc.
  - **Heap:** Là vùng nhớ tự do dùng cho việc cấp phát bộ nhớ động (`malloc`, `new`, khởi tạo object/mảng kích thước linh hoạt lúc chạy). Tốc độ chậm hơn Stack, dung lượng lớn (phụ thuộc RAM), cần lập trình viên chủ động giải phóng hoặc dựa vào bộ dọn rác (Garbage Collector).

### Câu 3: Tại sao trong hầu hết các ngôn ngữ lập trình: `0.1 + 0.2 != 0.3`?
- **Trả lời:**
  - Máy tính sử dụng hệ nhị phân (cơ số 2) để lưu trữ theo chuẩn số thực dấu phẩy động IEEE 754.
  - Trong hệ nhị phân, các phân số như $0.1$ ($\frac{1}{10}$) và $0.2$ ($\frac{1}{5}$) là số vô hạn tuần hoàn (tương tự như số $\frac{1}{3} = 0.3333...$ trong hệ thập phân).
  - Do số bit lưu trữ là hữu hạn (32-bit hay 64-bit), máy tính buộc phải cắt bớt và làm tròn. Khi thực hiện phép cộng hai số đã bị làm tròn, kết quả sinh ra sai số cực nhỏ ở phần đuôi (`0.30000000000000004`).

### Câu 4: Phân biệt User Mode (Ring 3) và Kernel Mode (Ring 0)? System Call đóng vai trò gì?
- **Trả lời:**
  - **User Mode:** Chế độ hạn chế đặc quyền dành cho ứng dụng người dùng, không được phép can thiệp trực tiếp vào phần cứng hoặc vùng nhớ của tiến trình khác.
  - **Kernel Mode:** Chế độ đặc quyền tối cao của nhân hệ điều hành, có toàn quyền thực thi mọi tập lệnh CPU và thao tác trực tiếp với phần cứng.
  - **System Call (Syscall):** Là cầu nối an toàn duy nhất giúp ứng dụng chạy ở User Mode yêu cầu Kernel thực hiện các tác vụ đặc quyền (như đọc/ghi file trên đĩa cứng, gửi nhận dữ liệu qua card mạng hay tạo tiến trình mới).

### Câu 5: Biến môi trường `PATH` là gì? Điều gì xảy ra khi bạn gõ một lệnh trên Terminal?
- **Trả lời:**
  - Biến môi trường `PATH` chứa danh sách các đường dẫn thư mục mà hệ điều hành sẽ tìm kiếm các file thực thi (executable binary).
  - Khi gõ một lệnh (ví dụ `git status`), hệ điều hành sẽ quét lần lượt các thư mục trong biến `PATH`. Nếu tìm thấy file `git` (hoặc `git.exe`), nó sẽ nạp file đó vào RAM để chạy. Nếu duyệt hết danh sách mà không thấy, nó sẽ báo lỗi `command not found`.

### Câu 6: Bộ nhớ ảo (Virtual Memory) là gì? Cơ chế Phân trang (Paging) và Page Fault hoạt động như thế nào?
- **Trả lời:**
  - **Virtual Memory** là kỹ thuật mà OS kết hợp phần cứng MMU tạo ra một không gian địa chỉ ảo liên tục cho mỗi tiến trình, che giấu địa chỉ RAM vật lý thật nhằm bảo vệ an toàn dữ liệu và tối ưu dung lượng.
  - Bộ nhớ ảo được chia thành các khối nhỏ gọi là **Page** (thường 4KB), tương ứng với các **Frame** trong RAM vật lý.
  - Khi ứng dụng truy xuất một Page hiện không nằm trong RAM mà đang bị lưu tạm trên ổ cứng (Swap / Pagefile), phần cứng sẽ kích hoạt một ngắt gọi là **Page Fault**. Hệ điều hành sẽ nạp trang đó từ đĩa cứng vào RAM rồi mới cho chương trình tiếp tục thực thi.

### Câu 7: Phân biệt Concurrency (Xử lý đồng thời) và Parallelism (Xử lý song song)?
- **Trả lời:**
  - **Concurrency** liên quan đến *cấu trúc chương trình*: Quản lý nhiều công việc cùng lúc bằng cách chuyển đổi qua lại cực nhanh giữa chúng (Time-slicing). Có thể xảy ra ngay trên CPU chỉ có 1 nhân đơn.
  - **Parallelism** liên quan đến *thực thi vật lý*: Thực sự chạy hai hay nhiều tác vụ tại cùng một mili-giây chính xác trên các nhân CPU (Cores) hoặc GPU vật lý khác nhau.

### Câu 8: Tại sao SSD NVMe lại nhanh hơn HDD hàng trăm lần? Chỉ số nào quan trọng nhất cho Database?
- **Trả lời:**
  - **HDD** dùng đĩa từ quay cơ học và kim đọc vật lý, thời gian tìm kiếm track/sector (seek time) mất từ 5 - 15ms.
  - **SSD NVMe** dùng chip nhớ flash điện tử và cắm trực tiếp vào bus PCIe của CPU, không có bộ phận chuyển động cơ học, độ trễ chỉ tính bằng micro-giây.
  - Với Database, chỉ số **Random Read/Write IOPS** (số lượt truy xuất ngẫu nhiên trên giây) là quan trọng nhất, vì dữ liệu bản ghi và index nằm rải rác. SSD NVMe đạt từ 500,000 đến 1,000,000 IOPS so với chỉ 75 - 150 IOPS của HDD.

### Câu 9: Bảng mã UTF-8 có gì ưu việt hơn ASCII và UTF-16?
- **Trả lời:**
  - ASCII chỉ hỗ trợ 128 ký tự tiếng Anh bằng 7 bits.
  - UTF-8 biểu diễn được toàn bộ các ngôn ngữ trên thế giới (Unicode) bằng cơ chế **độ dài biến thiên linh hoạt từ 1 đến 4 bytes**:
    - Ký tự tiếng Anh/ASCII chuẩn dùng đúng 1 byte (tương thích 100% ngược với hệ thống cũ).
    - Tiếng Việt / ký tự Latin dùng 2 - 3 bytes.
    - Tiếng Trung/Nhật và Emoji dùng 3 - 4 bytes.
  - Điều này giúp tiết kiệm băng thông và dung lượng lưu trữ hơn rất nhiều so với UTF-16 (luôn bắt buộc tối thiểu 2 bytes cho cả ký tự tiếng Anh).

### Câu 10: Dấu Pipe (`|`) trong Terminal có ý nghĩa gì? Cho ví dụ thực tế.
- **Trả lời:**
  - Dấu Pipe (`|`) chuyển luồng đầu ra tiêu chuẩn (`stdout`) của lệnh phía trước thành luồng đầu vào tiêu chuẩn (`stdin`) của lệnh phía sau.
  - Ví dụ thực tế: `cat access.log | grep "404" | wc -l` (Lấy nội dung file log -> Lọc các dòng có mã lỗi 404 -> Đếm tổng số dòng để biết có bao nhiêu request bị lỗi 404).

---

## 3. Checklist Tự Đánh Giá Kiến Thức (Self-Assessment)

Hãy tích chọn những mục bạn đã tự tin giải thích được mà không cần nhìn lại tài liệu:

- [ ] Tôi có thể giải thích được kiến trúc Von Neumann và chu trình Fetch-Decode-Execute của CPU.
- [ ] Tôi hiểu rõ sự khác biệt giữa L1/L2/L3 Cache, RAM và SSD/HDD.
- [ ] Tôi biết cách phân biệt CPU và GPU và lý do GPU được dùng cho AI/Deep Learning.
- [ ] Tôi hiểu rõ sự khác nhau giữa Process và Thread, Stack và Heap.
- [ ] Tôi hiểu tại sao xảy ra lỗi Stack Overflow và Memory Leak.
- [ ] Tôi có thể chuyển đổi giữa hệ nhị phân (Binary), thập phân (Decimal) và Hexadecimal.
- [ ] Tôi hiểu cơ chế biểu diễn số âm bằng Bù 2 và hiện tượng tràn số nguyên (Integer Overflow).
- [ ] Tôi giải thích được nguyên nhân sai số dấu phẩy động `0.1 + 0.2 != 0.3`.
- [ ] Tôi hiểu sự khác biệt giữa User Mode và Kernel Mode, vai trò của System Calls.
- [ ] Tôi thành thạo ít nhất 15 lệnh CLI thông dụng trên Windows PowerShell hoặc Linux Bash.
- [ ] Tôi hiểu cách thức hoạt động của biến môi trường `PATH`.
