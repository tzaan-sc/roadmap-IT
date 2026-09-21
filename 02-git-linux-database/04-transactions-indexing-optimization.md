# 04 - Giao Dịch, Chỉ Mục & Tối Ưu Hóa Truy Vấn (Transactions, Indexing & Optimization)

> **Mục tiêu bài học:** Nắm vững bản chất sống còn của Giao dịch cơ sở dữ liệu (Transactions) và 4 thuộc tính vàng ACID, giải phẫu cấu trúc cây B-Tree của Index, hiểu rõ sự đánh đổi giữa tốc độ Đọc và Ghi, và sử dụng công cụ `EXPLAIN ANALYZE` để chẩn đoán, biến một câu truy vấn chạy mất 10 giây trở về dưới 5 mili-giây.

---

## 1. Giao Dịch Cơ Sở Dữ Liệu (Transactions) Là Gì?

Hãy tưởng tượng kịch bản chuyển khoản ngân hàng:
> Bạn chuyển **500,000 VNĐ** từ tài khoản của mình (A) sang tài khoản của bạn bè (B).
> Hệ thống phải thực hiện 2 thao tác SQL:
> 1. Trừ 500,000 VNĐ từ số dư tài khoản A: `UPDATE accounts SET balance = balance - 500000 WHERE id = 'A'`
> 2. Cộng 500,000 VNĐ vào số dư tài khoản B: `UPDATE accounts SET balance = balance + 500000 WHERE id = 'B'`

**Thảm họa xảy ra khi:** Thao tác 1 vừa chạy xong thì máy chủ bị mất điện đột ngột hoặc đứt mạng, thao tác 2 không bao giờ được thực thi!
Kết quả: Tiền của bạn bị mất, nhưng bạn bè không nhận được. Tiền đã biến mất vào hư không!

### Giải pháp: Transaction
Một **Transaction** là một chuỗi nhiều thao tác đọc/ghi dữ liệu được gom lại thành **một đơn vị công việc duy nhất không thể tách rời**:

```sql
BEGIN TRANSACTION; -- Bắt đầu giao dịch

-- 1. Trừ tiền A
UPDATE accounts SET balance = balance - 500000 WHERE id = 'A';

-- 2. Cộng tiền B
UPDATE accounts SET balance = balance + 500000 WHERE id = 'B';

-- Nếu cả hai đều thành công không có lỗi:
COMMIT; -- Lưu vĩnh viễn thay đổi vào CSDL!

-- Nếu có bất kỳ lỗi nào xảy ra ở giữa chừng:
-- ROLLBACK; -- Khôi phục lại toàn bộ dữ liệu như chưa hề có chuyện gì xảy ra!
```

---

## 2. Bốn Thuộc Tính Vàng ACID Của Hệ Thống CSDL

Mọi hệ thống RDBMS chuẩn mực (PostgreSQL, MySQL InnoDB, Oracle, SQL Server) đều được thiết kế để đảm bảo chuẩn **ACID**:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        4 THUỘC TÍNH ACID CỦA CSDL                      │
└───────┬──────────────────┬───────────────────┬──────────────────┬──────┘
        │                  │                   │                  │
        ▼                  ▼                   ▼                  ▼
   A - ATOMICITY      C - CONSISTENCY     I - ISOLATION      D - DURABILITY
  (Tính nguyên tử)   (Tính nhất quán)    (Tính cô lập)      (Tính bền vững)
  Tất cả hoặc không  Dữ liệu luôn tuân   Các giao dịch đồng Dữ liệu đã commit
  gì cả. Lỗi 1 bước  thủ mọi ràng buộc   thời không can     thì không bao giờ
  => Rollback hết!   toàn vẹn hệ thống.  thiệp vào nhau.    bị mất (kể cả sập nguồn).
```

### 2.1. Atomicity (Tính nguyên tử)
- Đơn vị công việc không thể chia nhỏ: hoặc là **tất cả các câu lệnh trong transaction cùng thành công**, hoặc là **không có câu lệnh nào được lưu lại**.

### 2.2. Consistency (Tính nhất quán)
- Dữ liệu trước và sau transaction phải luôn thỏa mãn mọi quy tắc nghiệp vụ và ràng buộc (Constraints, Foreign Keys, Checks).
- Ví dụ: Tổng số tiền của A và B trước khi chuyển và sau khi chuyển luôn phải bằng nhau; số dư tài khoản không bao giờ được âm nếu có ràng buộc `CHECK (balance >= 0)`.

### 2.3. Isolation (Tính cô lập)
- Khi có hàng nghìn giao dịch diễn ra cùng một giây, hệ điều hành CSDL đảm bảo giao dịch này không nhìn thấy dữ liệu dở dang (chưa commit) của giao dịch kia.
- **4 Cấp độ cô lập (Isolation Levels) từ lỏng lẻo đến nghiêm ngặt:**
  1. *Read Uncommitted* (Cho phép đọc dữ liệu dở dang -> Nguy cơ dính Dirty Read).
  2. *Read Committed* (Chỉ đọc dữ liệu đã commit - Mức mặc định của PostgreSQL).
  3. *Repeatable Read* (Đảm bảo đọc lại cùng 1 hàng trong transaction thì giá trị luôn giống nhau).
  4. *Serializable* (Nghiêm ngặt nhất - Các giao dịch chạy như thể tuần tự từng cái một, an toàn tuyệt đối nhưng làm chậm hiệu năng).

### 2.4. Durability (Tính bền vững)
- Một khi bạn nhận được thông báo `COMMIT` thành công, dữ liệu được cam kết sẽ **tồn tại vĩnh viễn** kể cả khi trung tâm dữ liệu bị sét đánh hoặc mất nguồn điện ngay mili-giây sau đó.
- CSDL làm được điều này nhờ cơ chế **WAL (Write-Ahead Logging)**: Mọi thay đổi đều được ghi tuần tự vào đĩa cứng trong file log nhật ký trước khi thực sự ghi vào các file dữ liệu bảng.

---

## 3. Chỉ Mục (Database Indexing) - B-Tree & Cơ Chế Hoạt Động

### 3.1. Phép ẩn dụ về Mục lục cuốn sách
Giả sử bạn có bảng `users` chứa **10 triệu dòng**. Bạn chạy lệnh:
```sql
SELECT * FROM users WHERE email = 'alex@gmail.com';
```
- **Nếu KHÔNG có Index (Full Table Scan / Seq Scan):** Database buộc phải mở từng trang trong số 10 triệu bản ghi từ đầu đến cuối ổ cứng để so sánh chuỗi. Thời gian thực thi: **mất từ 3 đến 8 giây**, ổ cứng kêu ầm ĩ!
- **Nếu CÓ Index trên cột `email` (Index Scan):** Database tra cứu vào cấu trúc **Cây B-Tree** được sắp xếp sẵn, nhảy trực tiếp tới đúng khối dữ liệu chứa `alex@gmail.com`. Thời gian thực thi: **chỉ mất 1 mili-giây** ($O(\log N)$)!

```text
                  CẤU TRÚC CHỈ MỤC CÂY B-TREE (B-Tree Index)
                               [ Gốc: M ]
                              /          \
                     [ A - L ]            [ N - Z ]
                    /         \          /         \
                 [A - D]   [E - L]    [N - S]   [T - Z]
                   │          │          │         │
(Các nút lá trỏ trực tiếp tới địa chỉ vật lý của dòng dữ liệu trên ổ cứng!)
```

### 3.2. Cú pháp tạo Index
```sql
-- Tạo Index đơn cho một cột hay tìm kiếm:
CREATE INDEX idx_users_email ON users(email);

-- Tạo Composite Index (Chỉ mục kết hợp nhiều cột):
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

### 3.3. Cái Giá Của Việc Tạo Index (The Cost of Indexing)
Nhiều lập trình viên mới thường nghĩ: *"Cứ đánh index cho tất cả các cột để câu lệnh nào cũng chạy nhanh!"* — **Đây là sai lầm chết người!**

| Thao tác | Khi KHÔNG có Index | Khi CÓ Index |
| :--- | :--- | :--- |
| **SELECT (Đọc dữ liệu)** | Chậm $O(N)$ (Duyệt toàn bộ bảng) | **Siêu tốc $O(\log N)$** |
| **INSERT (Thêm mới)** | Rất nhanh (Chỉ cần ghi nối đuôi vào ổ đĩa) | **Chậm hơn nhiều** (Phải tính toán và chèn thêm node mới vào B-Tree) |
| **UPDATE / DELETE** | Nhanh | **Chậm hơn** (Phải cập nhật lại vị trí các node trên cây) |
| **Dung lượng lưu trữ** | Chỉ tốn RAM/Ổ cứng cho dữ liệu thật | **Tốn thêm rất nhiều RAM và dung lượng đĩa** để lưu cây Index |

> [!TIP]
> **Quy tắc vàng khi tạo Index:**
> 1. Chỉ đánh Index cho các cột thường xuyên xuất hiện trong mệnh đề **`WHERE`**, **`JOIN ON`**, hoặc **`ORDER BY`**.
> 2. Đánh Index cho các cột có độ chọn lọc cao (**High Cardinality** - dữ liệu ít bị trùng lặp như `email`, `user_id`, `uuid`). Tránh đánh index cho các cột chỉ có 2-3 giá trị lặp đi lặp lại như `gender` (Nam/Nữ) hay `status` (Active/Inactive).

---

## 4. Chẩn Đoán & Tối Ưu Truy Vấn Bằng `EXPLAIN ANALYZE`

Làm sao để biết một câu lệnh SQL đang chạy nhanh hay chậm, và CSDL có thực sự đang dùng Index mà bạn tạo hay không? Hãy dùng lệnh **`EXPLAIN ANALYZE`** (trong PostgreSQL):

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'alex@gmail.com';
```

### 4.1. Trường hợp 1: Chưa có Index (Seq Scan - Quét tuần tự)
```text
Seq Scan on users  (cost=0.00..185420.00 rows=1 width=128) (actual time=2450.120..4210.450 rows=1 loops=1)
  Filter: (email = 'alex@gmail.com'::text)
  Rows Removed by Filter: 9999999
Planning Time: 0.150 ms
Execution Time: 4215.300 ms  <--- Mất hơn 4.2 giây!
```
- `Seq Scan`: Database phải duyệt qua từng dòng một.
- `Rows Removed by Filter: 9999999`: Phải đọc và loại bỏ gần 10 triệu dòng để tìm 1 dòng!

### 4.2. Trường hợp 2: Sau khi đã tạo Index (Index Scan)
```text
Index Scan using idx_users_email on users  (cost=0.43..8.45 rows=1 width=128) (actual time=0.045..0.048 rows=1 loops=1)
  Index Cond: (email = 'alex@gmail.com'::text)
Planning Time: 0.210 ms
Execution Time: 0.075 ms  <--- Chưa đến 0.1 mili-giây! Nhanh gấp 50,000 lần!
```

---

## 5. Những Lỗi Kinh Điển Làm Vô Hiệu Hóa Index

Ngay cả khi bạn đã tạo Index, Database vẫn có thể **bỏ qua Index và quay lại quét toàn bộ bảng (Seq Scan)** nếu bạn viết SQL phạm phải các lỗi sau:

### Lỗi 1: Dùng hàm trên cột đã đánh Index
```sql
-- ❌ SAI: Bọc hàm UPPER() làm hỏng Index B-Tree
SELECT * FROM users WHERE UPPER(email) = 'ALEX@GMAIL.COM';

-- ✅ ĐÚNG: Giữ nguyên cột sạch hoặc tạo Expression Index
SELECT * FROM users WHERE email = 'alex@gmail.com';
```

### Lỗi 2: Tìm kiếm chuỗi với ký tự đại diện `%` ở đầu
```sql
-- ❌ SAI: Ký tự % nằm ở đầu khiến cây B-Tree không biết bắt đầu từ đâu!
SELECT * FROM users WHERE email LIKE '%gmail.com';

-- ✅ ĐÚNG: Ký tự % nằm ở đuôi vẫn tận dụng được Index
SELECT * FROM users WHERE email LIKE 'alex%';
```

### Lỗi 3: Vi phạm quy tắc tiền tố bên trái (Leftmost Prefix) của Composite Index
Nếu bạn tạo Composite Index trên 2 cột `(user_id, status)`:
- `WHERE user_id = 5 AND status = 'PAID'` ──► **Tận dụng tối đa Index**.
- `WHERE user_id = 5` ──► **Vẫn tận dụng được Index** (nhờ cột bên trái).
- `WHERE status = 'PAID'` ──► **KHÔNG DÙNG ĐƯỢC INDEX!** (Bỏ qua cột tiền tố đầu tiên).

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **Transaction** đảm bảo toàn bộ nhóm lệnh thành công trọn vẹn (`COMMIT`) hoặc hủy bỏ hoàn toàn (`ROLLBACK`).
2. Nhớ 4 trụ cột **ACID**: Atomicity (Nguyên tử), Consistency (Nhất quán), Isolation (Cô lập), Durability (Bền vững nhờ WAL log).
3. **B-Tree Index** tăng tốc độ đọc từ $O(N)$ về $O(\log N)$, nhưng làm chậm thao tác `INSERT`/`UPDATE` và tốn thêm RAM.
4. Luôn dùng **`EXPLAIN ANALYZE`** để kiểm tra câu truy vấn xem có đang dính `Seq Scan` nguy hiểm hay không.
5. Tránh bọc hàm hoặc dùng `LIKE '%...'` trên các cột Index để không làm vô hiệu hóa công sức tối ưu.
