# 01 - Kiến Trúc Phần Cứng Máy Tính (Hardware Architecture)

> **Mục tiêu bài học:** Nắm vững cấu tạo phần cứng cốt lõi của máy tính, cách các thành phần giao tiếp với nhau và hiểu sâu về hệ thống phân cấp bộ nhớ (Memory Hierarchy) – nền tảng quyết định hiệu năng của mọi dòng code bạn viết.

---

## 1. Mô hình Kiến trúc Von Neumann

Hầu hết mọi máy tính cá nhân, server và điện thoại ngày nay đều hoạt động dựa trên mô hình **Von Neumann** đề xuất từ năm 1945:

```text
┌─────────────────────────────────────────────────────────┐
│                     MÁY TÍNH (COMPUTER)                │
│                                                         │
│  ┌─────────────────────────┐   Bus Dữ liệu / Địa chỉ    │
│  │   CPU                   │◄────────────────────────►┌─┴────────────┐
│  │ ┌─────────────────────┐ │                          │ BỘ NHỚ CHÍNH │
│  │ │ Khối điều khiển     │ │                          │ (RAM)        │
│  │ │ (Control Unit - CU) │ │                          │              │
│  │ ├─────────────────────┤ │                          │ Chứa Lệnh    │
│  │ │ Khối số học & logic │ │                          │ & Dữ liệu    │
│  │ │ (ALU)               │ │                          └─┬────────────┘
│  │ ├─────────────────────┤ │                            ▲
│  │ │ Thanh ghi           │ │                            │
│  │ │ (Registers)         │ │                            │
│  └─┴─────────────────────┘ │                            │
└──────────────▲─────────────┘                            │
               │                                          │
    ┌──────────┴──────────┐                    ┌──────────┴──────────┐
    │  THIẾT BỊ ĐẦU VÀO   │                    │  THIẾT BỊ ĐẦU RA    │
    │  (Bàn phím, Chuột)  │                    │  (Màn hình, Loa)    │
    └─────────────────────┘                    └─────────────────────┘
```

### Nguyên lý cốt lõi:
- **Stored-program concept:** Cả **chương trình (các dòng mã lệnh máy)** và **dữ liệu mà chương trình xử lý** đều được nạp chung vào một bộ nhớ duy nhất (RAM).
- CPU liên tục lặp lại chu trình: **Fetch** (Lấy lệnh từ RAM) → **Decode** (Giải mã lệnh) → **Execute** (Thực thi lệnh).

---

## 2. CPU (Central Processing Unit) - Bộ Não Của Hệ Thống

CPU chịu trách nhiệm thực thi các chỉ thị (instructions) từ phần mềm. Một CPU hiện đại bao gồm các thành phần trọng yếu:

### 2.1. Cấu tạo bên trong CPU

1. **ALU (Arithmetic Logic Unit - Khối tính toán số học & logic):**
   - Thực hiện các phép toán cộng, trừ, nhân, chia.
   - Thực hiện các phép toán logic Boolean: `AND`, `OR`, `NOT`, `XOR`, phép dịch bit (shift/rotate).

2. **CU (Control Unit - Khối điều khiển):**
   - Đọc chỉ thị từ bộ nhớ, giải mã chúng và phát tín hiệu điều khiển tới các thành phần khác (ALU, Registers, RAM) để thi hành chỉ thị.

3. **Thanh ghi (Registers):**
   - Là những ô nhớ **nằm ngay bên trong chip CPU**.
   - Tốc độ nhanh nhất trong toàn bộ hệ thống (dưới 1 nanosecond - chỉ mất 1 chu kỳ xung nhịp CPU).
   - Dung lượng rất nhỏ (vài chục đến vài trăm byte).
   - Ví dụ các thanh ghi quan trọng trong kiến trúc x86-64:
     - `RAX`, `RBX`, `RCX`, `RDX`: Thanh ghi đa năng để tính toán.
     - `RIP` (Instruction Pointer / Program Counter): Lưu **địa chỉ bộ nhớ** của chỉ thị máy tính tiếp theo cần thực thi.
     - `RSP` (Stack Pointer): Trỏ tới đỉnh của Call Stack hiện tại.
     - `RBP` (Base Pointer): Trỏ tới đáy của Stack Frame hiện tại.

4. **Bộ nhớ đệm CPU (CPU Cache):**
   - Do RAM cách CPU một quãng đường vật lý trên bo mạch chủ và có độ trễ lớn, CPU trang bị các tầng Cache tốc độ cực cao:
     - **L1 Cache (Level 1):** Nằm sát core nhất, tốc độ ~0.5 - 1 ns, chia thành L1i (chứa lệnh) và L1d (chứa dữ liệu), dung lượng ~32KB - 64KB mỗi core.
     - **L2 Cache (Level 2):** Dung lượng ~512KB - 1MB mỗi core, độ trễ ~3 - 5 ns.
     - **L3 Cache (Level 3):** Dùng chung cho tất cả các core, dung lượng ~16MB - 96MB+, độ trễ ~10 - 20 ns.

> [!TIP]
> **Cache Miss và Cache Hit:** Khi CPU cần dữ liệu, nó sẽ tìm trong L1 → L2 → L3 → RAM.
> - **Cache Hit:** Tìm thấy ngay trong Cache → Tốc độ xử lý đạt tối đa.
> - **Cache Miss:** Không thấy, CPU phải dừng đợi nạp dữ liệu từ RAM (mất hàng trăm chu kỳ xung nhịp). Viết code duyệt mảng liên tục theo dòng bộ nhớ (spatial locality) giúp tận dụng Cache Hit cực tốt.

### 2.2. Xung nhịp, Core và Thread

- **Xung nhịp (Clock Speed - GHz):**
  - Số chu kỳ CPU có thể thực hiện mỗi giây.
  - Ví dụ: CPU `3.5 GHz` = 3.5 tỷ chu kỳ dao động mỗi giây.
  - *Lưu ý:* Xung nhịp cao chưa chắc chạy nhanh hơn nếu chỉ số **IPC (Instructions Per Cycle - số lệnh xử lý trên mỗi chu kỳ)** thấp.
- **Physical Core (Nhân vật lý):**
  - Mỗi nhân là một đơn vị xử lý độc lập có đầy đủ ALU, CU và Cache riêng. CPU 8 nhân có thể thực hiện song song 8 luồng tính toán vật lý thực sự cùng một thời điểm.
- **Hyper-Threading / SMT (Simultaneous Multithreading):**
  - Công nghệ giả lập giúp 1 nhân vật lý xử lý đồng thời 2 luồng tính toán (Logical Cores/Threads).
  - Ví dụ: Chip 8 Cores / 16 Threads. Khi 1 thread đang chờ dữ liệu từ RAM, core sẽ tận dụng ALU đang rảnh để tính toán cho thread còn lại.

---

## 3. Bộ Nhớ Trong - RAM (Random Access Memory)

- **Tại sao gọi là "Random Access" (Truy xuất ngẫu nhiên)?**
  - Bạn có thể truy xuất đến bất kỳ ô nhớ nào theo địa chỉ với thời gian như nhau, không cần phải đọc tuần tự từ đầu đến cuối như băng từ.
- **Tính chất Volatile (Khả biến):**
  - Dữ liệu trong RAM chỉ tồn tại khi có dòng điện nuôi. Khi tắt máy hoặc mất điện đột ngột, toàn bộ dữ liệu trên RAM sẽ bị xóa sạch.
- **Băng thông và Kênh đôi (Dual Channel):**
  - Lắp 2 thanh RAM 8GB chạy Dual-Channel (128-bit bus) đem lại băng thông dữ liệu gấp đôi so với 1 thanh RAM 16GB Single-Channel (64-bit bus).

---

## 4. Bộ Nhớ Lưu Trữ - SSD vs HDD (Secondary Storage)

Là bộ nhớ **Non-volatile (Phi khả biến)**: Dữ liệu được lưu giữ vĩnh viễn ngay cả khi ngắt nguồn điện.

| Tiêu chí | HDD (Hard Disk Drive) | SSD (Solid State Drive) - SATA | SSD NVMe (PCIe) |
| :--- | :--- | :--- | :--- |
| **Công nghệ** | Đĩa từ quay vật lý, kim đọc cơ học | Chip nhớ flash NAND | Chip nhớ flash NAND giao tiếp PCIe |
| **Tốc độ đọc/ghi tuần tự** | ~100 - 200 MB/s | ~500 - 550 MB/s | ~3,500 - 7,500+ MB/s |
| **IOPS (Truy xuất ngẫu nhiên 4K)** | ~75 - 150 IOPS (cực chậm do chờ kim quay) | ~50,000 - 90,000 IOPS | ~500,000 - 1,000,000+ IOPS |
| **Độ trễ (Latency)** | ~10 - 15 ms (mili-giây) | ~50 - 100 µs (micro-giây) | ~10 - 20 µs |
| **Độ bền cơ học** | Dễ hỏng khi va đập, rơi rớt | Chống sốc tốt | Chống sốc tốt |
| **Ứng dụng thực tế** | Lưu trữ dữ liệu lạnh (Backups, Phim, Ảnh) | Nâng cấp máy tính cũ | Chạy OS, Database, Build code, Server cao cấp |

> [!IMPORTANT]
> Trong lập trình Backend và Database, chỉ số **IOPS (Input/Output Operations Per Second)** quan trọng hơn nhiều so với tốc độ đọc ghi tuần tự, vì Database liên tục đọc ghi các mẩu dữ liệu nhỏ nằm rải rác trên ổ cứng.

---

## 5. Kim Tự Tháp Phân Cấp Bộ Nhớ (Memory Hierarchy)

Trong khoa học máy tính, tốc độ luôn tỷ lệ nghịch với dung lượng và chi phí:

```text
               ▲                TỐC ĐỘ        CHI PHÍ/GB     DUNG LƯỢNG
              / \             (Nhanh nhất)    (Đắt nhất)    (Nhỏ nhất)
             /   \
            / Reg \           < 1 ns          Cực đắt        Vài Byte
           /───────\
          / L1/L2/L3\         1 - 20 ns       Rất đắt        Vài MB
         /───────────\
        /     RAM     \       50 - 100 ns     Đắt            8 - 64 GB
       /───────────────\
      /    NVMe SSD     \     10 - 25 µs      Trung bình     512GB - 2TB
     /───────────────────\
    /     HDD / SATA      \   5 - 15 ms       Rẻ             2TB - 10TB+
   /───────────────────────\
  /   Cloud / Cold Storage  \ (Chậm nhất)     (Rẻ nhất)      (Vô hạn)
```

**Quy tắc ngón tay cái giúp lập trình viên hình dung độ trễ:**
Nếu lấy 1 chu kỳ CPU (CPU Register access = 0.3 ns) tương đương **1 giây** trong thế giới con người:
- Truy xuất L1 Cache: ~ **3 giây**
- Truy xuất L2 Cache: ~ **14 giây**
- Truy xuất RAM: ~ **4 phút**
- Đọc SSD NVMe: ~ **1.5 ngày**
- Đọc từ HDD: ~ **1 đến 12 tháng**!
- Gửi 1 gói tin mạng từ VN sang Mỹ và quay về: ~ **vài năm**!

Chính vì vậy, các lập trình viên giỏi luôn tối ưu giải thuật để giảm thiểu truy cập đĩa cứng (Disk I/O) và gọi mạng (Network calls) thông qua việc sử dụng **RAM Caching (như Redis, Memcached)**.

---

## 6. GPU (Graphics Processing Unit) - Tại Sao Quan Trọng Trong AI & Đồ Họa?

### Sự khác biệt cốt lõi giữa CPU và GPU:

```text
      CPU ARCHITECTURE                         GPU ARCHITECTURE
  (Vài Core Lớn, Tối ưu độ trễ)          (Hàng Ngàn Core Nhỏ, Tối ưu thông lượng)

  ┌─────────────────────────┐          ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐
  │         Core 1          │          │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
  │    (Rất mạnh, đa năng)  │          ├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
  ├─────────────────────────┤          │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
  │         Core 2          │          ├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
  ├─────────────────────────┤          │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
  │         Core 3          │          ├─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┼─┤
  ├─────────────────────────┤          │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │ │
  │         Core 4          │          └─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
  └─────────────────────────┘          (Thousands of SIMD processing cores)
```

- **CPU (Đầu bếp trưởng 5 sao):** Có từ 4 đến 64 cores. Mỗi core có thể giải quyết các tác vụ tuần tự logic cực kỳ phức tạp (chạy hệ điều hành, rẽ nhánh if/else, quản lý tệp).
- **GPU (Đội ngũ 10,000 học viên phụ bếp):** Có từ vài nghìn đến chục nghìn cores nhỏ. Mỗi core tính toán đơn giản, nhưng tất cả cùng tính **đồng thời (Massive Parallel Processing)** theo mô hình **SIMD (Single Instruction, Multiple Data)**.

### Ứng dụng:
1. **Render đồ họa / Game:** Tính toán màu sắc và vị trí của hàng triệu pixel trên màn hình cùng một lúc.
2. **Trí tuệ nhân tạo (AI & Machine Learning):** Bản chất của việc huấn luyện và suy luận mạng nơ-ron (Deep Learning, LLM) là hàng tỷ phép nhân ma trận số thực (`Matrix Multiplication`). GPU tính toán các phép toán ma trận này nhanh hơn CPU từ 50 đến 100 lần.

---

## 7. Bo Mạch Chủ (Motherboard) và Hệ Thống Bus

- **Bo mạch chủ (Mainboard/Motherboard):** Là bảng mạch in trung tâm kết nối mọi linh kiện: CPU, RAM, ổ cứng, GPU, card mạng.
- **Hệ thống Bus:** Các đường truyền tín hiệu điện tử mang dữ liệu:
  - **Data Bus:** Chở dữ liệu giữa CPU và các linh kiện.
  - **Address Bus:** Chở địa chỉ ô nhớ cần đọc/ghi.
  - **Control Bus:** Mang tín hiệu điều khiển (đọc hay ghi, ngắt lệnh).
  - **PCIe (PCI Express):** Giao tiếp tốc độ siêu cao nối trực tiếp CPU với GPU và ổ SSD NVMe.

---

## 8. Tóm Tắt & Ghi Nhớ Nhanh

1. **CPU:** Bộ não điều khiển và tính toán; xung nhịp càng cao và nhiều core thì xử lý càng mạnh.
2. **Registers & Cache:** Ô nhớ siêu tốc ngay trên CPU; giảm thiểu tối đa tình trạng chờ đợi dữ liệu từ RAM.
3. **RAM:** Nơi nạp chương trình đang chạy; tốc độ cao, mất dữ liệu khi mất điện.
4. **SSD/HDD:** Nơi lưu trữ vĩnh viễn; SSD NVMe nhanh hơn HDD gấp hàng chục lần.
5. **GPU:** Chuyên gia tính toán song song ma trận; thống trị lĩnh vực Render và Deep Learning/AI.
6. **Nguyên tắc vàng tối ưu code:** Giữ dữ liệu càng gần CPU càng tốt (tận dụng Cache và RAM, hạn chế tối đa I/O Disk & Network).
