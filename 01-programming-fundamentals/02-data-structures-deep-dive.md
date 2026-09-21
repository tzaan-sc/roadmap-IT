# 02 - Cấu Trúc Dữ Liệu Chuyên Sâu (Data Structures Deep Dive)

> *"Lập trình viên kém quan tâm đến code. Lập trình viên giỏi quan tâm đến cấu trúc dữ liệu và mối quan hệ giữa chúng."* — **Linus Torvalds** (Người sáng lập Linux & Git)

Cấu trúc dữ liệu (Data Structure) là cách thức tổ chức, quản lý và lưu trữ dữ liệu trong bộ nhớ máy tính sao cho có thể truy cập và sửa đổi một cách hiệu quả nhất.

---

## 1. Mảng (Array) & Mảng Động (Dynamic Array)

### 1.1. Mảng tĩnh (Static Array)
Mảng là cấu trúc dữ liệu tuyến tính lưu trữ các phần tử cùng kiểu trong **một khối bộ nhớ RAM vật lý liên tục (Contiguous Memory)**.

```text
Mảng 4 phần tử int (mỗi số 4 bytes), bắt đầu từ địa chỉ 0x1000:
Địa chỉ RAM:  0x1000     0x1004     0x1008     0x100C
             ┌──────────┬──────────┬──────────┬──────────┐
Giá trị:     │    42    │    15    │    88    │    07    │
             └──────────┴──────────┴──────────┴──────────┘
Chỉ số (Index):    [0]        [1]        [2]        [3]
```

- **Công thức tính địa chỉ thần thánh:**
  $$\text{Address}(A[i]) = \text{Base Address} + i \times \text{Size of Element}$$
- Nhờ công thức này, CPU có thể nhảy ngay lập tức tới phần tử thứ $i$ bất kỳ trong thời gian **$O(1)$** mà không cần duyệt qua các phần tử trước đó.

### 1.2. Mảng động (Dynamic Array - Python `list`, JS `Array`, Java `ArrayList`, C++ `std::vector`)
- Trong thực tế, kích thước dữ liệu luôn biến động. Mảng động giải quyết vấn đề này bằng cơ chế **tự động mở rộng kích thước**:
  1. Ban đầu khởi tạo mảng tĩnh có dung lượng (Capacity) là 4.
  2. Khi bạn thêm (append) phần tử thứ 5 (đầy): Mảng động sẽ cấp phát một vùng nhớ mới có dung lượng **gấp đôi (Capacity = 8)** ở chỗ khác trong RAM.
  3. Sao chép toàn bộ 4 phần tử cũ sang vùng nhớ mới, giải phóng vùng nhớ cũ.
  4. Thêm phần tử thứ 5 vào.
- **Độ phức tạp khấu hao (Amortized Time Complexity):** Dù việc nhân đôi tốn $O(N)$, nhưng thao tác này rất hiếm khi xảy ra. Tính trung bình, thêm phần tử vào cuối mảng vẫn đạt **$O(1)$**.

| Thao tác | Đầu mảng | Cuối mảng | Vị trí bất kỳ theo Index |
| :--- | :--- | :--- | :--- |
| **Truy cập (Access)** | $O(1)$ | $O(1)$ | $O(1)$ (Cực nhanh) |
| **Chèn (Insert)** | $O(N)$ (Phải dịch chuyển toàn bộ mảng sang phải) | $O(1)$ (Khấu hao) | $O(N)$ |
| **Xóa (Delete)** | $O(N)$ (Phải dồn mảng về bên trái) | $O(1)$ | $O(N)$ |

---

## 2. Danh Sách Liên Kết (Linked List)

Khác với Mảng, các phần tử trong Linked List **không cần nằm cạnh nhau trong RAM**. Mỗi phần tử là một **Node** nằm rải rác trên Heap, liên kết với nhau qua các con trỏ (Pointers).

```text
DANH SÁCH LIÊN KẾT ĐƠN (Singly Linked List)
HEAD
 │
 ▼
┌──────┬──────┐      ┌──────┬──────┐      ┌──────┬──────┐
│ Data │ Next ├───►  │ Data │ Next ├───►  │ Data │ NULL │
│  10  │ 0x2A │      │  20  │ 0x5F │      │  30  │      │
└──────┴──────┘      └──────┴──────┘      └──────┴──────┘
```

### 2.1. Phân loại Linked List
- **Singly Linked List:** Mỗi node chỉ có con trỏ `next` trỏ về phía trước.
- **Doubly Linked List:** Mỗi node có cả con trỏ `prev` và `next` (dễ dàng đi lùi và tiến).
- **Circular Linked List:** Node cuối cùng trỏ vòng ngược lại Node đầu tiên (HEAD).

### 2.2. So sánh Mảng (Array) vs Danh Sách Liên Kết (Linked List)

| Tiêu chí | Mảng (Array) | Danh Sách Liên Kết (Linked List) |
| :--- | :--- | :--- |
| **Phân bổ RAM** | Khối liên tục | Rải rác theo từng Node |
| **Truy cập theo Index** | Siêu nhanh $O(1)$ | Phải duyệt tuần tự $O(N)$ |
| **Chèn/Xóa ở đầu** | Chậm $O(N)$ vì phải dịch chuyển cả mảng | **Cực nhanh $O(1)$** (chỉ cần bẻ lại con trỏ HEAD) |
| **Bộ nhớ phụ trội** | Không có (chỉ lưu dữ liệu thuần) | Tốn thêm RAM để lưu địa chỉ con trỏ (`next`, `prev`) |
| **CPU Cache Locality** | Rất tốt (dữ liệu liên tục -> nạp đệm một lần) | Kém (dữ liệu rải rác gây Cache Miss liên tục) |

---

## 3. Ngăn Xếp (Stack) - Nguyên Lý LIFO

**Stack** hoạt động theo nguyên tắc **LIFO (Last-In, First-Out)**: Phần tử nào đưa vào sau cùng sẽ được lấy ra đầu tiên (giống như chồng đĩa ăn tiệc).

```text
               STACK OPERATIONS:
        ┌─────────────┐
PUSH ──►│  Phần tử C  │ (Đỉnh Stack - TOP) ──► POP (Lấy C ra đầu tiên)
        ├─────────────┤
        │  Phần tử B  │
        ├─────────────┤
        │  Phần tử A  │ (Đáy Stack)
        └─────────────┘
```

- **Các thao tác cốt lõi:**
  - `push(item)`: Đẩy một phần tử lên đỉnh Stack — **$O(1)$**.
  - `pop()`: Lấy và xóa phần tử ở đỉnh Stack — **$O(1)$**.
  - `peek()` / `top()`: Xem giá trị ở đỉnh Stack mà không xóa — **$O(1)$**.
- **Ứng dụng kinh điển:**
  - Quản lý lời gọi hàm của CPU (**Call Stack**).
  - Tính năng **Undo / Redo** (`Ctrl + Z`) trong Word, VS Code.
  - Lịch sử duyệt trang web (Nút "Back" của trình duyệt).
  - Kiểm tra tính hợp lệ của dấu ngoặc đóng mở trong trình biên dịch `({[]})`.

---

## 4. Hàng Đợi (Queue) - Nguyên Lý FIFO

**Queue** hoạt động theo nguyên tắc **FIFO (First-In, First-Out)**: Phần tử nào vào trước sẽ được phục vụ trước (giống như xếp hàng mua vé xem phim).

```text
      ENQUEUE (Vào hàng ở ĐUÔI)                   DEQUEUE (Ra khỏi hàng ở ĐẦU)
                         ──► ┌───┬───┬───┬───┐ ──►
     [Mới vào]               │ D │ C │ B │ A │              [Được phục vụ trước]
     (REAR / TAIL)           └───┴───┴───┴───┘             (FRONT / HEAD)
```

- **Các thao tác cốt lõi:**
  - `enqueue(item)`: Thêm một phần tử vào cuối hàng đợi — **$O(1)$**.
  - `dequeue()`: Lấy phần tử ở đầu hàng đợi ra xử lý — **$O(1)$**.
  - `front()`: Xem phần tử ở đầu hàng đợi — **$O(1)$**.
- **Biến thể quan trọng:**
  - **Deque (Double-ended queue):** Hàng đợi hai đầu, có thể thêm và rút ở cả đầu lẫn đuôi trong $O(1)$.
  - **Priority Queue:** Mỗi phần tử có mức độ ưu tiên, phần tử ưu tiên cao nhất sẽ được xử lý trước bất kể thứ tự vào hàng (thường cài bằng Binary Heap).
- **Ứng dụng:**
  - Hệ thống hàng đợi Message Queue (RabbitMQ, Apache Kafka) phân phối tác vụ cho worker.
  - Quản lý lệnh in ấn của máy in (Print Spooler).
  - Thuật toán tìm kiếm theo chiều rộng (BFS).
  - Cơ chế Task Queue trong JavaScript Event Loop.

---

## 5. Bảng Băm (Hash Table / Hash Map / Dictionary)

Đây là **cấu trúc dữ liệu quyền lực và được sử dụng nhiều nhất** trong ngành phần mềm thực tế.

```text
  KEY (Khóa)           HÀM BĂM (Hash Function)       BUCKETS (Mảng RAM)
"username"      ───►   hash("username") % 10  ───►   Index 2: "john_doe"
"email"         ───►   hash("email") % 10     ───►   Index 7: "john@work.com"
"age"           ───►   hash("age") % 10       ───►   Index 4: 28
```

### 5.1. Cơ chế hoạt động
1. Đầu vào là một `Key` dạng chuỗi hoặc số tùy ý.
2. Cho qua **Hàm băm (Hash Function)** để biến thành một con số nguyên lớn.
3. Lấy số nguyên đó chia lấy dư (`%`) cho kích thước mảng để tìm vị trí Index.
4. Ghi hoặc đọc giá trị trực tiếp tại ô Index đó trong thời gian trung bình **$O(1)$**!

### 5.2. Hiện tượng Xung đột Băm (Hash Collision)
Điều gì xảy ra khi hai Key hoàn toàn khác nhau (ví dụ `"cat"` và `"dog"`) sau khi băm lại cho ra cùng một chỉ số Index?
- **Chaining (Nối chuỗi - phổ biến nhất):** Tại mỗi ô Index của mảng, thay vì lưu 1 giá trị, ta đặt một Danh sách liên kết (Linked List). Khi bị trùng Index, ta gắn node mới vào đuôi danh sách.
- **Open Addressing (Thăm dò tuyến tính - Linear Probing):** Nếu ô đó đã có người chiếm giữ, thuật toán sẽ dò sang ô kế tiếp (Index + 1, Index + 2...) cho đến khi thấy ô trống.

| Thao tác | Trung bình (Average) | Xấu nhất (Worst-case khi collision toàn bộ) |
| :--- | :--- | :--- |
| **Tìm kiếm (Get)** | **$O(1)$** | $O(N)$ |
| **Thêm mới (Put/Set)**| **$O(1)$** | $O(N)$ |
| **Xóa (Delete)** | **$O(1)$** | $O(N)$ |

---

## 6. Tập Hợp (Set)

**Set** là cấu trúc dữ liệu lưu trữ một tập các phần tử **không có thứ tự và không cho phép trùng lặp (Unique elements)**.

```text
Mảng ban đầu: [ 1, 2, 2, 3, 4, 4, 4, 5 ]
Sau khi đưa vào Set: { 1, 2, 3, 4, 5 }  (Tự động lọc trùng hoàn hảo)
```

- Bản chất bên dưới: Set thường được xây dựng dựa trên chính Hash Table (chỉ lưu các Key mà không có Value tương ứng).
- Độ phức tạp tìm kiếm xem một phần tử có tồn tại trong Set (`has` / `in`) là **$O(1)$**, trong khi tìm trong Mảng mất $O(N)$.
- Các phép toán đại số tập hợp:
  - **Union (Hợp):** $A \cup B$ (Lấy tất cả phần tử ở cả 2 tập).
  - **Intersection (Giao):** $A \cap B$ (Chỉ lấy phần tử xuất hiện ở cả hai).
  - **Difference (Hiệu):** $A \setminus B$ (Có trong A nhưng không có trong B).

---

## 7. Cây (Trees) & Cây Tìm Kiếm Nhị Phân (Binary Search Tree - BST)

Cây là cấu trúc dữ liệu phi tuyến tính phân cấp (hierarchical).

```text
                  [ ROOT: 50 ]              ◄── Tầng 0 (Gốc)
                 /            \
        [ Node: 30 ]        [ Node: 70 ]    ◄── Tầng 1
        /          \        /          \
     [ 20 ]      [ 40 ]  [ 60 ]      [ 80 ] ◄── Tầng 2 (Leaf - Lá)
```

### 7.1. Cây tìm kiếm nhị phân (BST - Binary Search Tree)
Một cây nhị phân được gọi là BST khi với mọi node:
- Mọi node con nằm bên cây **con bên trái** đều có giá trị **nhỏ hơn** node cha.
- Mọi node con nằm bên cây **con bên phải** đều có giá trị **lớn hơn** node cha.

Nhờ tính chất này, việc tìm kiếm một phần tử trong BST tương đương với tìm kiếm nhị phân: Tại mỗi bước, ta so sánh và loại bỏ được một nửa số node!
- Tìm kiếm, chèn, xóa trên BST cân bằng: **$O(\log N)$**.

### 7.2. Các cách duyệt cây nhị phân (Tree Traversal)
- **In-order (Trái → Gốc → Phải):** Với cây BST, duyệt In-order sẽ cho ra dãy số **được sắp xếp tăng dần** (`20, 30, 40, 50, 60, 70, 80`).
- **Pre-order (Gốc → Trái → Phải):** Thường dùng để sao chép hoặc tuần tự hóa (serialize) cấu trúc cây.
- **Post-order (Trái → Phải → Gốc):** Thường dùng khi cần xóa cây hoặc giải phóng bộ nhớ (xóa con trước khi xóa cha).
- **Level-order (Theo từng tầng - BFS):** Duyệt ngang từ trái sang phải qua từng hàng độ sâu.

---

## 8. Đồ Thị (Graphs)

Đồ thị là mô hình toán học tổng quát nhất biểu diễn các mối quan hệ mạng lưới. Gồm tập hợp các **Đỉnh (Vertices / Nodes)** và các **Cạnh kết nối (Edges)**.

```text
ĐỒ THỊ VÔ HƯỚNG                           ĐỒ THỊ CÓ HƯỚNG
 (Mạng kết bạn Facebook)                   (Theo dõi Twitter / Instagram)
    (A) ─────── (B)                           (A) ──────► (B)
     │           │                             ▲           │
     │           │                             │           ▼
    (C) ─────── (D)                           (C) ◄────── (D)
```

### 8.1. Cách biểu diễn đồ thị trong mã nguồn

1. **Ma trận kề (Adjacency Matrix):** Dùng mảng 2 chiều kích thước $V \times V$.
   - Ô `matrix[i][j] = 1` nếu có cạnh nối giữa đỉnh $i$ và đỉnh $j$.
   - Ưu điểm: Kiểm tra xem 2 đỉnh có nối nhau không mất $O(1)$.
   - Nhược điểm: Tốn bộ nhớ $O(V^2)$, lãng phí khi đồ thị thưa (ít cạnh).
2. **Danh sách kề (Adjacency List - Phổ biến nhất):** Dùng một Hash Map hoặc mảng, mỗi đỉnh lưu danh sách các đỉnh kề của nó:
   ```javascript
   const graph = {
     "A": ["B", "C"],
     "B": ["A", "D"],
     "C": ["A", "D"],
     "D": ["B", "C"]
   };
   ```

---

## 9. Bảng Tổng Kết Độ Phức Tạp Của Các Cấu Trúc Dữ Liệu

| Cấu trúc dữ liệu | Truy cập ngẫu nhiên | Tìm kiếm | Chèn vào đầu | Chèn vào cuối | Xóa | Trường hợp ứng dụng |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Array** | $O(1)$ | $O(N)$ | $O(N)$ | $O(1)$ | $O(N)$ | Cần truy cập nhanh theo chỉ số, duyệt tuần tự |
| **Linked List** | $O(N)$ | $O(N)$ | $O(1)$ | $O(1)$ (nếu có tail) | $O(1)$ (khi đã ở node) | Chèn/xóa liên tục ở đầu/cuối, không cần index |
| **Stack** | $O(N)$ | $O(N)$ | $O(1)$ (push) | - | $O(1)$ (pop) | Xử lý LIFO, Undo/Redo, Call Stack |
| **Queue** | $O(N)$ | $O(N)$ | - | $O(1)$ (enqueue) | $O(1)$ (dequeue) | Xử lý FIFO, Task Queue, Hàng đợi server |
| **Hash Map / Set** | N/A | **$O(1)$** | **$O(1)$** | **$O(1)$** | **$O(1)$** | Tra cứu cực nhanh theo từ khóa, lọc trùng |
| **Binary Search Tree**| N/A | **$O(\log N)$** | - | **$O(\log N)$** | **$O(\log N)$** | Dữ liệu vừa cần tìm nhanh, vừa duy trì thứ tự |
