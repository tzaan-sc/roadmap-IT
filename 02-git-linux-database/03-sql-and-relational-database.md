# 03 - Cơ Sở Dữ Liệu Quan Hệ & Ngôn Ngữ SQL (SQL & Relational Database)

> **Mục tiêu bài học:** Nắm vững bản chất của Hệ quản trị cơ sở dữ liệu quan hệ (RDBMS), viết thành thạo các câu lệnh SQL từ CRUD cơ bản tới các phép nối bảng phức tạp (JOINs, GROUP BY, HAVING), thiết kế lược đồ quan hệ chuẩn mực (Khóa chính, Khóa ngoại) và thành thạo kỹ thuật Chuẩn hóa dữ liệu (1NF, 2NF, 3NF).

---

## 1. Cơ Sở Dữ Liệu Là Gì? RDBMS vs NoSQL

**Cơ sở dữ liệu (Database)** là một hệ thống được tổ chức khoa học để lưu trữ, quản lý và truy xuất dữ liệu một cách an toàn và bền vững trên ổ đĩa cứng.

```text
CƠ SỞ DỮ LIỆU QUAN HỆ (RDBMS - SQL)          CƠ SỞ DỮ LIỆU PHI QUAN HỆ (NoSQL)
Ví dụ: PostgreSQL, MySQL, SQLite, Oracle     Ví dụ: MongoDB, Redis, Cassandra

┌────────────────────────────────────────┐   ┌────────────────────────────────────────┐
│ BẢNG (TABLE: Users) - Schema cố định  │   │ BỘ SƯU TẬP (COLLECTION: Users) - JSON  │
│ ID  │ Tên         │ Email       │ Tuổi │   │ [                                      │
├─────┼─────────────┼─────────────┼──────┤   │   { "_id": 1, "name": "Nam" },         │
│ 1   │ Nguyen Nam  │ nam@work.com│ 25   │   │   { "_id": 2, "name": "Lan",           │
│ 2   │ Tran Lan    │ lan@work.com│ 22   │   │     "skills": ["React", "Go"] }        │
└─────┴─────────────┴─────────────┴──────┘   │ ]  (Mỗi document có thể khác trường!) │
```

| Tiêu chí | CSDL Quan hệ (RDBMS / SQL) | CSDL Phi quan hệ (NoSQL) |
| :--- | :--- | :--- |
| **Cấu trúc (Schema)** | Cố định, nghiêm ngặt (Strict Schema) | Linh hoạt, không cố định (Dynamic Schema) |
| **Dữ liệu phức tạp** | Chia thành nhiều bảng và liên kết bằng Khóa ngoại (JOIN) | Nhúng trực tiếp JSON lồng nhau (Embedded documents) |
| **Tính toàn vẹn** | Cực kỳ cao, đảm bảo giao dịch **ACID** | Chấp nhận tính nhất quán sau cùng (Eventual Consistency) |
| **Khả năng mở rộng** | Tối ưu mở rộng theo chiều dọc (Vertical Scaling - nâng cấp CPU/RAM) | Tối ưu mở rộng theo chiều ngang (Horizontal Scaling - cụm server) |
| **Ứng dụng lý tưởng** | Ngân hàng, Thương mại điện tử, Quản lý kho, ERP | Mạng xã hội, Dữ liệu log khổng lồ, Cache (Redis), Big Data |

---

## 2. Ngôn Ngữ SQL: DDL vs DML (CRUD Thần Thánh)

SQL (Structured Query Language) là ngôn ngữ tiêu chuẩn để tương tác với mọi RDBMS.

### 2.1. DDL (Data Definition Language) - Định nghĩa cấu trúc bảng
```sql
-- Tạo bảng người dùng (users)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,              -- Khóa chính, tự động tăng
    username VARCHAR(50) NOT NULL UNIQUE,-- Bắt buộc có và không được trùng
    email VARCHAR(100) NOT NULL UNIQUE,
    age INT CHECK (age >= 18),          -- Ràng buộc tuổi phải từ 18 trở lên
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- Tự động gán thời gian tạo
);

-- Sửa bảng: Thêm cột số điện thoại
ALTER TABLE users ADD COLUMN phone VARCHAR(15);

-- Xóa toàn bộ bảng khỏi database (Cẩn trọng!)
-- DROP TABLE users;
```

### 2.2. DML (Data Manipulation Language) - Thao tác dữ liệu (CRUD)

```sql
-- 1. CREATE (Tạo mới dữ liệu)
INSERT INTO users (username, email, age) 
VALUES 
    ('nguyennam', 'nam@gmail.com', 25),
    ('tranlan', 'lan@gmail.com', 22);

-- 2. READ (Đọc dữ liệu)
SELECT id, username, email FROM users WHERE age > 20;

-- 3. UPDATE (Cập nhật dữ liệu)
UPDATE users 
SET email = 'nam_new@gmail.com', age = 26 
WHERE id = 1; -- CẢNH BÁO: Quên mệnh đề WHERE sẽ cập nhật TẤT CẢ mọi hàng!

-- 4. DELETE (Xóa dữ liệu)
DELETE FROM users 
WHERE id = 2; -- CẢNH BÁO: Quên mệnh đề WHERE sẽ XÓA SẠCH toàn bộ bảng!
```

---

## 3. Truy Vấn Nâng Cao: Gom Nhóm & Lọc Dữ Liệu

### 3.1. Các hàm tổng hợp (Aggregate Functions)
- `COUNT(*)`: Đếm tổng số dòng.
- `SUM(column)`: Tính tổng giá trị.
- `AVG(column)`: Tính trung bình cộng.
- `MIN(column)` / `MAX(column)`: Tìm giá trị nhỏ nhất / lớn nhất.

### 3.2. Mệnh đề `GROUP BY` và sự khác biệt giữa `WHERE` vs `HAVING`
Giả sử bạn có bảng `orders` (đơn hàng). Hãy tính tổng tiền của từng khách hàng, và chỉ giữ lại những khách hàng có tổng chi tiêu trên 1000$:

```sql
SELECT 
    user_id, 
    COUNT(id) AS total_orders, 
    SUM(total_amount) AS total_spent
FROM orders
WHERE status = 'COMPLETED'          -- 1. WHERE: Lọc các dòng đơn lẻ TRƯỚC KHI gom nhóm
GROUP BY user_id                     -- 2. GROUP BY: Gom các đơn của cùng 1 user lại
HAVING SUM(total_amount) >= 1000     -- 3. HAVING: Lọc các nhóm SAU KHI đã tính tổng
ORDER BY total_spent DESC            -- 4. Sắp xếp người tiêu nhiều nhất lên đầu
LIMIT 5;                             -- 5. Chỉ lấy top 5 khách hàng VIP
```

> [!IMPORTANT]
> **Quy tắc phân biệt:**
> - **`WHERE`**: Lọc từng hàng dữ liệu thô **trước khi** hàm tính toán (`SUM`, `AVG`, `COUNT`) chạy. Bạn không thể dùng `WHERE SUM(total) > 1000`.
> - **`HAVING`**: Lọc kết quả của các nhóm **sau khi** hàm tính toán đã gom nhóm xong.

---

## 4. Làm Chủ Các Phép Nối Bảng (JOINs)

Vì dữ liệu trong RDBMS được tách thành nhiều bảng để tránh trùng lặp, ta dùng mệnh đề **JOIN** để kết nối các bảng lại với nhau dựa trên mối quan hệ giữa các cột (thường là Khóa ngoại trỏ vào Khóa chính).

```text
       BẢNG USERS                              BẢNG ORDERS
  ┌────┬─────────────┐                   ┌────┬─────────┬────────┐
  │ id │ name        │                   │ id │ user_id │ total  │
  ├────┼─────────────┤                   ├────┼─────────┼────────┤
  │ 1  │ Alice       │                   │ 10 │ 1       │ 150$   │
  │ 2  │ Bob         │                   │ 11 │ 1       │ 200$   │
  │ 3  │ Charlie     │                   │ 12 │ 2       │ 80$    │
  └────┴─────────────┘                   └────┴─────────┴────────┘
(Charlie chưa mua đơn nào!)            (Không có đơn nào của id=3)
```

```text
                          CÁC LOẠI PHÉP NỐI (JOINS)
     INNER JOIN                  LEFT JOIN                  FULL OUTER JOIN
┌─────────┬─────────┐      ┌─────────┬─────────┐      ┌─────────┬─────────┐
│  Users  │ Orders  │      │  Users  │ Orders  │      │  Users  │ Orders  │
│    (Chung)        │      │ (Tất cả)│ (Nếu có)│      │ (Tất cả)│ (Tất cả)│
│     █████         │      │ ████████████      │      │ █████████████████ │
└─────────┴─────────┘      └─────────┴─────────┘      └─────────┴─────────┘
```

### 4.1. `INNER JOIN` (Giao điểm chung)
Chỉ trả về các hàng mà giá trị khóa khớp nhau ở **cả hai bảng**:
```sql
SELECT users.name, orders.id AS order_id, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;
-- Kết quả: Chỉ hiện Alice (2 đơn) và Bob (1 đơn). Charlie bị loại trừ vì chưa mua hàng!
```

### 4.2. `LEFT JOIN` (Toàn bộ bên trái)
Lấy **tất cả các dòng của bảng bên trái** (`users`), nếu bảng bên phải (`orders`) không có dữ liệu khớp thì điền `NULL`:
```sql
SELECT users.name, orders.id AS order_id, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- Kết quả: Hiện cả Alice, Bob VÀ Charlie (với order_id = NULL, total = NULL).
```
> [!TIP]
> **Ứng dụng kinh điển của LEFT JOIN:** Tìm xem những ai chưa từng phát sinh đơn hàng:
> `WHERE orders.id IS NULL`.

---

## 5. Thiết Kế Cơ Sở Dữ Liệu Quan Hệ (Schema Design)

### 5.1. Khóa chính (Primary Key) & Khóa ngoại (Foreign Key)
- **Primary Key (PK):** Cột định danh duy nhất cho mỗi hàng trong bảng, không được phép `NULL` (ví dụ `user_id`, số CCCD).
- **Foreign Key (FK):** Cột trong bảng này trỏ vào Khóa chính của bảng khác để tạo sợi dây ràng buộc toàn vẹn (**Referential Integrity**).

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    
    -- Thiết lập Khóa ngoại
    CONSTRAINT fk_orders_users
        FOREIGN KEY (user_id) 
        REFERENCES users(id)
        ON DELETE CASCADE -- Nếu xóa user thì tự động xóa sạch các order của user đó!
);
```

### 5.2. Ba mối quan hệ cơ bản trong thế giới thực

1. **Một - Một (1 - 1):**
   - Một người dùng có đúng 1 hồ sơ cá nhân (`users` ── `user_profiles`).
2. **Một - Nhiều (1 - N):**
   - Một tác giả viết nhiều cuốn sách, nhưng mỗi cuốn sách chỉ thuộc về một tác giả chính (`authors` ── `books`). Khóa ngoại `author_id` đặt ở bảng `books`.
3. **Nhiều - Nhiều (N - N):**
   - Một sinh viên học nhiều môn, một môn học có nhiều sinh viên đăng ký (`students` ── `courses`).
   - **Bắt buộc phải tạo một Bảng trung gian (Junction Table):**

```text
  [ students ]                    [ enrollments ]                    [ courses ]
  - id (PK)   ◄───────────────┬── - student_id (FK)              ┌── - id (PK)
  - name                      └── - course_id (FK)  ─────────────┴── - title
                                  - enrolled_date
```

---

## 6. Chuẩn Hóa Cơ Sở Dữ Liệu (Database Normalization)

Chuẩn hóa là kỹ thuật thiết kế bảng nhằm **loại bỏ sự dư thừa dữ liệu (Data Redundancy)** và ngăn ngừa các lỗi dị thường (Anomalies) khi Thêm, Sửa hoặc Xóa dữ liệu.

### 6.1. Dạng chuẩn 1 (1NF - First Normal Form): Tính nguyên tử
- Mỗi ô dữ liệu chỉ được chứa **một giá trị đơn lẻ (Atomic Value)**, không được chứa danh sách nhiều giá trị ngăn cách bởi dấu phẩy.
- *Vi phạm 1NF:* Cột `phones` lưu `"090123456, 098765432"`.
- *Sửa đạt 1NF:* Tách thành các dòng riêng hoặc bảng số điện thoại riêng.

### 6.2. Dạng chuẩn 2 (2NF - Second Normal Form): Phụ thuộc hàm đầy đủ
- Đã đạt 1NF.
- Mọi cột không khóa phải phụ thuộc vào **toàn bộ khóa chính** (chỉ áp dụng khi khóa chính là khóa tổ hợp nhiều cột).

### 6.3. Dạng chuẩn 3 (3NF - Third Normal Form): Không phụ thuộc bắc cầu
- Đã đạt 2NF.
- Không một cột không khóa nào được phụ thuộc vào một cột không khóa khác ($A \rightarrow B \rightarrow C$).
- *Vi phạm 3NF:* Trong bảng `orders` lưu: `order_id (PK)`, `customer_id`, `customer_name`, `customer_city`. Nếu khách hàng đổi tên, bạn phải cập nhật ở 1,000 dòng đơn hàng!
- *Sửa đạt 3NF:* Tách `customer_name`, `customer_city` sang bảng `customers` riêng biệt. Bảng `orders` chỉ giữ lại `customer_id`.

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **RDBMS (SQL)** lưu trữ dữ liệu dạng bảng với lược đồ cố định, đảm bảo tính toàn vẹn cao.
2. Cú pháp cơ bản: `SELECT`, `INSERT`, `UPDATE`, `DELETE` (luôn cẩn trọng với `WHERE`).
3. Dùng `GROUP BY` kết hợp với các hàm tổng hợp (`COUNT`, `SUM`) và lọc nhóm bằng `HAVING`.
4. Nắm chắc **`INNER JOIN`** (giao nhau) và **`LEFT JOIN`** (lấy trọn vẹn bảng bên trái).
5. Mối quan hệ **Nhiều - Nhiều** luôn cần một bảng trung gian để liên kết hai khóa ngoại.
6. Tuân thủ **Chuẩn hóa đến 3NF** để loại bỏ dữ liệu dư thừa và tránh lỗi cập nhật dây chuyền.
