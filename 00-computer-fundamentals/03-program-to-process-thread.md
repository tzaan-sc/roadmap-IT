# 03 - Vòng Đời Thực Thi: Program → Process → Thread → CPU & Memory

> **Mục tiêu bài học:** Thấu hiểu tường tận hành trình một đoạn mã (source code) biến thành chương trình (program), được nạp vào bộ nhớ (RAM) thành tiến trình (process), chia nhỏ thành các luồng (threads) và được CPU thực thi như thế nào. Nắm vững cấu trúc bộ nhớ Stack vs Heap và cơ chế bộ nhớ ảo (Virtual Memory).

---

## 1. Hành Trình Từ Mã Nguồn Tới Mã Máy

Khi bạn viết một file code (ví dụ `main.c` hay `main.py`), nó chỉ là một file văn bản thuần túy. Làm sao máy tính có thể hiểu và chạy được?

```text
  [ SOURCE CODE ] (file .c, .cpp, .rs, .go)
         │
         ▼ (Compiler: dịch cú pháp sang Assembly)
  [ ASSEMBLY CODE ] (Mã hợp ngữ tương ứng với kiến trúc CPU)
         │
         ▼ (Assembler: chuyển sang mã nhị phân thô)
  [ OBJECT FILE ] (.obj, .o)
         │
         ▼ (Linker: kết nối thư viện, gán địa chỉ)
  [ EXECUTABLE BINARY ] (.exe trên Windows, file ELF trên Linux)
```

- **Ngôn ngữ biên dịch (Compiled: C, C++, Go, Rust):** Được dịch toàn bộ trước sang mã máy trực tiếp (`010101...`). Chạy cực nhanh, độc lập phần cứng đích.
- **Ngôn ngữ thông dịch (Interpreted: Python, Ruby, JavaScript):** Có một chương trình thông dịch (Interpreter / Engine như Python Interpreter hay V8) đọc từng dòng lệnh và chuyển hóa tức thời thành hành động thực thi.
- **Ngôn ngữ lai (JIT / Bytecode: Java, C#):** Biên dịch sang mã trung gian (Bytecode như `.class` hay IL), sau đó máy ảo (JVM / CLR) sẽ thông dịch và biên dịch JIT (Just-In-Time) sang mã máy lúc chạy.

---

## 2. Phân Biệt: Program vs Process vs Thread

Đây là khái niệm kinh điển thường xuyên xuất hiện trong các buổi phỏng vấn kỹ thuật:

```text
┌─────────────────────────────────────────────────────────────┐
│ PROGRAM (Tệp thực thi nằm im trên ổ đĩa SSD/HDD)            │
│ Ví dụ: C:\Program Files\Google\Chrome\chrome.exe            │
└──────────────────────────────┬──────────────────────────────┘
                               │ Double click / Chạy lệnh
                               │ OS Loader nạp vào RAM
┌──────────────────────────────▼──────────────────────────────┐
│ PROCESS (Tiến trình đang chạy trong bộ nhớ RAM)             │
│ - Có PID (Process ID) riêng, ví dụ: PID 4520                │
│ - Có không gian bộ nhớ ảo (Virtual Memory) độc lập          │
│                                                             │
│   ┌───────────────────────────────────────────────────────┐ │
│   │                 BÊN TRONG 1 PROCESS                   │ │
│   │                                                       │ │
│   │  DÙNG CHUNG: [ Code Segment | Heap | Global Data ]   │ │
│   │                                                       │ │
│   │  ┌──────────────────────┐   ┌──────────────────────┐  │ │
│   │  │       THREAD 1       │   │       THREAD 2       │  │ │
│   │  │  - Stack riêng       │   │  - Stack riêng       │  │ │
│   │  │  - Registers riêng   │   │  - Registers riêng   │  │ │
│   │  │  - Program Counter   │   │  - Program Counter   │  │ │
│   │  └──────────────────────┘   └──────────────────────┘  │ │
│   └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Bảng so sánh Process và Thread:

| Tiêu chí | Process (Tiến trình) | Thread (Luồng thực thi) |
| :--- | :--- | :--- |
| **Bản chất** | Một thực thể chương trình đang chạy | Đơn vị thực thi nhỏ nhất bên trong một Process |
| **Không gian bộ nhớ** | Độc lập hoàn toàn. Process A **không thể** đọc/ghi vào RAM của Process B | Dùng chung không gian Heap và biến toàn cục của Process cha |
| **Chi phí khởi tạo** | Đắt đỏ (Cần cấp phát bảng trang bộ nhớ, nạp file, tài nguyên OS) | Nhẹ và nhanh hơn rất nhiều (Lightweight) |
| **Giao tiếp dữ liệu** | Phải dùng IPC (Inter-Process Communication: Pipes, Sockets, Shared Memory) | Truy cập trực tiếp qua biến chung trên Heap |
| **Rủi ro sập ứng dụng** | Nếu Process A bị crash, Process B hoàn toàn không bị ảnh hưởng | Nếu 1 Thread bị lỗi bộ nhớ (Segmentation Fault), **toàn bộ Process sập** |

---

## 3. Bố Cục Bộ Nhớ Của Một Tiến Trình (Process Memory Layout)

Khi hệ điều hành nạp một chương trình vào RAM, nó phân bổ một không gian bộ nhớ địa chỉ ảo (thường từ `0x00000000` đến `0x7FFFFFFFFFFF` trên hệ thống 64-bit) thành các phân vùng chuẩn:

```text
  Địa chỉ cao (High Address)
  ┌────────────────────────────────────────────────────────┐
  │                   HỆ ĐIỀU HÀNH KERNEL                  │
  │           (Ứng dụng người dùng không thể chạm vào)     │
  ├────────────────────────────────────────────────────────┤
  │             STACK (Ngăn xếp - Phát triển GIẢM XUỐNG)    │
  │  - Chứa biến cục bộ, tham số hàm, địa chỉ trả về      │
  │  - Tự động cấp phát và giải phóng cực nhanh            │
  │                           │                            │
  │                           ▼                            │
  │                                                        │
  │                           ▲                            │
  │                           │                            │
  │             HEAP (Đống - Phát triển TĂNG LÊN)          │
  │  - Cấp phát động theo yêu cầu (malloc, new, dynamic)  │
  │  - Phải tự giải phóng (C/C++) hoặc dùng Garbage Coll   │
  ├────────────────────────────────────────────────────────┤
  │  BSS SEGMENT (Uninitialized Data)                      │
  │  - Chứa biến toàn cục / static chưa gán giá trị (= 0)  │
  ├────────────────────────────────────────────────────────┤
  │  DATA SEGMENT (Initialized Data)                       │
  │  - Chứa biến toàn cục / static đã khởi tạo giá trị     │
  ├────────────────────────────────────────────────────────┤
  │  TEXT / CODE SEGMENT (Chỉ đọc - Read-only)             │
  │  - Chứa mã máy nhị phân thực thi của chương trình      │
  └────────────────────────────────────────────────────────┘
  Địa chỉ thấp (Low Address: 0x00000000)
```

### So Sánh Chi Tiết: STACK vs HEAP

| Tiêu chí | Stack (Ngăn xếp) | Heap (Bộ nhớ đống) |
| :--- | :--- | :--- |
| **Cơ chế hoạt động** | Cấu trúc LIFO (Last In, First Out). Khi gọi hàm, 1 Stack Frame được đẩy vào; khi hàm kết thúc, Frame bị hủy | Cấp phát tự do, linh hoạt theo kích thước tùy ý lúc runtime |
| **Tốc độ** | **Cực nhanh** (CPU chỉ cần dịch con trỏ thanh ghi `RSP`) | Chậm hơn (OS phải tìm khoảng trống bộ nhớ phù hợp) |
| **Kích thước** | Rất nhỏ (thường mặc định chỉ 1MB - 8MB) | Rất lớn (giới hạn bởi dung lượng RAM vật lý và ổ Swap) |
| **Vòng đời dữ liệu** | Tự động sinh ra và tự động hủy khi thoát khỏi phạm vi hàm (`{ ... }`) | Sống cho đến khi được giải phóng thủ công hoặc do GC (Garbage Collector) dọn |
| **Lỗi thường gặp** | **Stack Overflow**: Đệ quy vô hạn hoặc khai báo mảng cục bộ quá lớn | **Memory Leak**: Cấp phát mà quên giải phóng làm RAM đầy dần |

---

## 4. Chu Trình Thực Thi Của CPU (Fetch - Decode - Execute)

CPU không chạy toàn bộ code cùng lúc. Nó làm việc theo chu kỳ nhịp nhàng:

```text
    ┌────────────────────────────────────────┐
    │  1. FETCH (Lấy lệnh)                   │
    │  Đọc lệnh máy tại địa chỉ nằm trong    │
    │  thanh ghi con trỏ lệnh (PC / RIP)     │
    └──────────────────┬─────────────────────┘
                       │
                       ▼
    ┌────────────────────────────────────────┐
    │  2. DECODE (Giải mã lệnh)              │
    │  Khối CU giải mã xem lệnh cần làm gì   │
    │  (Ví dụ: Lệnh ADD, MOV, JUMP, PUSH)    │
    └──────────────────┬─────────────────────┘
                       │
                       ▼
    ┌────────────────────────────────────────┐
    │  3. EXECUTE (Thực thi lệnh)            │
    │  Khối ALU tính toán hoặc đọc/ghi ô nhớ.│
    │  Cập nhật con trỏ lệnh PC sang lệnh kế.│
    └──────────────────┬─────────────────────┘
                       │
                       └────────► Lặp lại hàng tỷ lần/giây!
```

---

## 5. Đa Nhiệm & Chuyển Ngữ Cảnh (Context Switching)

Tại sao một máy tính chỉ có 8 nhân CPU nhưng có thể mở cùng lúc hàng trăm ứng dụng (Chrome với 50 tabs, Spotify nghe nhạc, VS Code, Discord...)?

### 5.1. Concurrency (Đồng thời) vs Parallelism (Song song)

```text
CONCURRENCY (Xử lý đồng thời)              PARALLELISM (Xử lý song song thực sự)
- 1 Đầu bếp làm 2 món ăn xen kẽ:            - 2 Đầu bếp làm 2 món ăn cùng 1 lúc:
  Xào rau 30s -> Trở cá 30s -> Xào rau        Đầu bếp 1: Xào rau
- CPU đơn nhân chuyển đổi tác vụ cực nhanh    Đầu bếp 2: Trở cá
  tạo cảm giác chạy cùng lúc                 - Đòi hỏi CPU phải có nhiều Core vật lý
```

### 5.2. Context Switching là gì?
Khi CPU dừng chạy Process A để chuyển sang chạy Process B:
1. CPU phải **lưu toàn bộ trạng thái hiện tại** của Process A (giá trị các thanh ghi Registers, con trỏ lệnh `PC`, con trỏ ngăn xếp `SP`) vào bộ nhớ (PCB - Process Control Block).
2. Nạp trạng thái trước đó của Process B từ PCB vào các thanh ghi của CPU.
3. CPU bắt đầu thực thi tiếp lệnh của Process B.

> [!WARNING]
> **Chi phí Context Switch:** Thao tác chuyển đổi ngữ cảnh tiêu tốn từ vài micro-giây đến mili-giây và làm "nguội" CPU Cache (Cache Invalidation). Tạo quá nhiều Thread không làm app chạy nhanh hơn mà sẽ khiến CPU kiệt sức vì bận chuyển đổi ngữ cảnh thay vì làm việc thực tế!

---

## 6. Bộ Nhớ Ảo (Virtual Memory) & Phân Trang (Paging)

Trong các máy tính hiện đại, **ứng dụng KHÔNG BAO GIỜ nhìn thấy địa chỉ RAM vật lý thật**.

### 6.1. Tại sao cần Bộ Nhớ Ảo?
1. **Bảo mật tuyệt đối:** Mỗi ứng dụng sống trong một thế giới địa chỉ ảo độc lập (từ 0 đến $2^{64}-1$). Tiến trình này không thể nào đọc trộm dữ liệu của tiến trình kia.
2. **Ảo ảnh dung lượng lớn:** Cho phép chạy các phần mềm yêu cầu 16GB RAM trên một máy tính chỉ có 8GB RAM thực.

```text
[ Địa chỉ ảo của App ] (Virtual Address)
          │
          ▼
[ Phần cứng MMU (Memory Management Unit) ]
          │ (Tra cứu trong Bảng Trang - Page Table)
          ▼
┌──────────────────────────────────────────────────────────┐
│  Nếu trang nằm trong RAM  │  Nếu trang nằm trên Ổ Đĩa    │
│  => Truy xuất RAM vật lý  │  => Phát sinh PAGE FAULT     │
│     (Physical RAM)        │     OS đọc dữ liệu từ SWAP   │
│                           │     vào nạp lại RAM          │
└───────────────────────────┴──────────────────────────────┘
```

- **Paging (Phân trang):** Hệ điều hành chia bộ nhớ thành các khối nhỏ cố định gọi là **Page** (thường là `4KB`).
- **Page Fault:** Xảy ra khi một chương trình cố gắng truy cập vào một Page ảo hiện không nằm trong RAM thật mà đang bị đẩy ra ổ cứng (Swap Space / Pagefile). Hệ điều hành sẽ tạm dừng ứng dụng, nạp trang từ ổ cứng vào RAM rồi mới cho ứng dụng chạy tiếp. Nếu RAM bị thiếu trầm trọng, máy sẽ rơi vào tình trạng **Thrashing** (liên tục swap ổ cứng khiến máy bị đơ cứng).

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **Program** là file tĩnh trên đĩa cứng; khi được chạy, OS nạp vào RAM tạo thành **Process**.
2. **Process** chứa các **Thread**. Các Thread trong cùng một Process chia sẻ chung tài nguyên Heap nhưng có Stack riêng.
3. **Stack** dành cho biến cục bộ của hàm, tốc độ siêu tốc nhưng dung lượng nhỏ.
4. **Heap** dành cho dữ liệu động, dung lượng lớn nhưng cần quản lý giải phóng cẩn thận để tránh memory leak.
5. **CPU** thực hiện chu trình Fetch → Decode → Execute liên tục theo sự điều phối của thanh ghi con trỏ lệnh (`PC`).
6. **Virtual Memory** bảo vệ các ứng dụng không xâm phạm lẫn nhau và hỗ trợ cơ chế Paging/Swap khi thiếu RAM.
