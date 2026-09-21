# 04 - Kết Nối Cơ Sở Dữ Liệu, Xác Thực & Bảo Mật Web (Database Integration & Auth)

> **Mục tiêu bài học:** Nắm vững cơ chế kết nối CSDL an toàn từ Backend thông qua Connection Pooling, phân biệt Raw SQL vs ORM, phòng chống lỗ hổng SQL Injection, làm chủ cơ chế xác thực người dùng bằng JWT vs Session, băm mật khẩu chuẩn bằng bcrypt và hiểu thấu đáo các khái niệm bảo mật web cốt lõi (CORS, XSS, CSRF).

---

## 1. Kết Nối Cơ Sở Dữ Liệu: Connection Pooling & SQL Injection

### 1.1. Tại sao cần Connection Pool (Bể kết nối)?
Mỗi lần mở một kết nối mới tới Database (PostgreSQL/MySQL), máy chủ phải thực hiện bắt tay TCP 3 bước, xác thực tài khoản và cấp phát bộ nhớ. Thao tác này mất từ **50 đến 100 mili-giây**:
- Nếu mỗi request của người dùng đều mở 1 kết nối mới rồi đóng lại: Database sẽ sập vì quá tải!
- **Connection Pool** giải quyết việc này bằng cách duy trì sẵn một nhóm (ví dụ: 10 - 20 kết nối mở sẵn thường trực). Khi có request tới, ứng dụng "mượn" một kết nối có sẵn từ pool, chạy câu lệnh SQL xong thì "trả lại" cho pool mà không đóng kết nối vật lý.

```text
┌────────────────────────────────────────────────────────┐
│                   CONNECTION POOL                      │
│   ┌──────────────┐   ┌──────────────┐   ┌────────────┐ │
│   │ Kết nối 1    │   │ Kết nối 2    │   │ Kết nối 3  │ │ (Mở sẵn tới DB)
│   └──────▲───────┘   └──────▲───────┘   └────────────┘ │
└──────────┼──────────────────┼──────────────────────────┘
           │ (Mượn)           │ (Mượn)
      Request A          Request B
```

---

### 1.2. Thảm Họa SQL Injection & Cách Phòng Chống

Lỗ hổng bảo mật nổi tiếng nhất lịch sử phát triển web xảy ra khi lập trình viên **ghép chuỗi người dùng trực tiếp vào câu lệnh SQL**:

```sql
-- ❌ NGUY HIỂM CHẾT NGƯỜI (SQL Injection):
-- Giả sử biến username lấy trực tiếp từ ô input người dùng:
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '...';
-- Vì '1'='1' luôn luôn ĐÚNG, hacker sẽ đăng nhập thành công vào tài khoản Admin
-- mà không cần biết mật khẩu!
```

#### Giải pháp chuẩn mực: Parameterized Queries (Truy vấn có tham số hóa)
Tuyệt đối không ghép chuỗi. Hãy sử dụng các dấu giữ chỗ (`$1`, `?`):
```javascript
// ✅ AN TOÀN TUYỆT ĐỐI (Dùng Parameterized Query với thư viện pg):
const text = 'SELECT * FROM users WHERE username = $1 AND password_hash = $2';
const values = [userInputUsername, hashedInputPassword];
const res = await pool.query(text, values);
```
Trình biên dịch CSDL sẽ xử lý chuỗi nhập vào thuần túy như một giá trị văn bản, không bao giờ biên dịch nó thành mã SQL thực thi!

---

## 2. So Sánh: Raw SQL vs Query Builder vs ORM

| Tiêu chí | Raw SQL (Thuần) | Query Builder (Knex.js) | ORM (Prisma, TypeORM, Hibernate) |
| :--- | :--- | :--- | :--- |
| **Bản chất** | Viết câu lệnh SQL trực tiếp dưới dạng chuỗi | Dùng hàm JS để lắp ghép câu lệnh SQL | Ánh xạ bảng thành Đối tượng Class/Model, quản lý quan hệ tự động |
| **Hiệu năng** | **Tối đa 100%**, không có lớp trung gian | Rất cao | Tốt, nhưng có thể bị chậm nếu dính lỗi **N+1 Query** |
| **Type Safety** | Không có (sai tên cột chỉ biết lúc chạy) | Tương đối | **Tuyệt đối**: IDE gợi ý chính xác từng trường dữ liệu |
| **Độ khó** | Đòi hỏi giỏi SQL | Trung bình | Dễ bắt đầu, cú pháp hướng đối tượng rất quen thuộc |

---

## 3. Xác Thực & Phân Quyền: Session-Cookie vs JWT

- **Authentication (Xác thực - AuthN):** Trả lời câu hỏi: *"Bạn là ai?"* (Đăng nhập bằng Email/Password).
- **Authorization (Phân quyền - AuthZ):** Trả lời câu hỏi: *"Bạn được phép làm gì?"* (Ví dụ: Chỉ Admin mới có quyền xóa tài khoản).

```text
CƠ CHẾ SESSION - COOKIE (Stateful)            CƠ CHẾ JSON WEB TOKEN - JWT (Stateless)
1. User đăng nhập thành công                  1. User đăng nhập thành công
2. Server tạo Session lưu vào RAM/Redis       2. Server ký một chuỗi JWT mã hóa
3. Trả về Cookie chứa sessionId               3. Trả về JWT cho Client (Client tự giữ)
4. Mỗi request sau: Client gửi kèm Cookie     4. Mỗi request sau: Gửi JWT trong Header
5. Server PHẢI tra cứu RAM để xác nhận        5. Server KHÔNG CẦN tra cứu DB, chỉ cần
   (Tốn RAM khi có hàng triệu user!)             kiểm tra chữ ký hợp lệ là xong!
```

### 3.1. Cấu trúc của JSON Web Token (JWT)
Một chuỗi JWT gồm 3 phần ngăn cách bởi dấu chấm (`.`): `header.payload.signature`

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjQyLCJyb2xlIjoiYWRtaW4ifQ.s4nF2k9...
└────────────────┬──────────────────┘ └─────────────────┬────────────────┘ └─────┬─────┘
           1. HEADER                           2. PAYLOAD                      3. SIGNATURE
     (Thuật toán mã hóa)                  (Dữ liệu: userId, role)          (Chữ ký chống sửa đổi)
```

- **Header:** Chứa loại token và thuật toán ký (ví dụ HMAC SHA256).
- **Payload:** Chứa thông tin người dùng (ví dụ: `userId: 42, role: "admin"`). **Lưu ý: Dữ liệu này chỉ được mã hóa Base64 chứ không hề bị ẩn giấu, tuyệt đối không lưu mật khẩu vào Payload!**
- **Signature (Chữ ký):** Được tạo ra bằng công thức:
  $$\text{HMACSHA256}(\text{Header} + "." + \text{Payload}, \text{SECRET\_KEY})$$
  Nếu hacker sửa đổi `role: "user"` thành `"admin"`, chữ ký sẽ bị sai ngay lập tức và server từ chối request.

---

## 4. Bảo Mật Mật Khẩu: Thuật Toán Băm Một Chiều (bcrypt)

> [!CAUTION]
> **Quy tắc đạo đức sống còn của lập trình viên:**
> **TUYỆT ĐỐI KHÔNG BAO GIỜ** lưu mật khẩu thô của người dùng vào Database. Kể cả quản trị viên hệ thống cũng không được phép biết mật khẩu thật của người dùng!

### Thuật toán Băm Có Muối (Salted Hashing với bcrypt):
- **Băm (Hash):** Là hàm toán học một chiều (One-way function). Từ mật khẩu có thể sinh ra chuỗi hash, nhưng từ chuỗi hash **không bao giờ có thể giải ngược lại** ra mật khẩu ban đầu.
- **Muối (Salt):** Một chuỗi ngẫu nhiên được tự động thêm vào trước mật khẩu để chống các cuộc tấn công tra cứu bảng tính sẵn (**Rainbow Table Attacks**).

```javascript
import bcrypt from 'bcrypt';

const passwordNguoiDung = "MatKhauSieuKho123!";

// 1. Khi ĐĂNG KÝ: Băm mật khẩu với 10 vòng muối
const saltRounds = 10;
const hashedPassword = await bcrypt.hash(passwordNguoiDung, saltRounds);
// Chuỗi lưu vào CSDL trông như thế này:
// "$2b$10$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW"

// 2. Khi ĐĂNG NHẬP: Đối chiếu mật khẩu người dùng vừa gõ với mã hash trong DB
const isMatch = await bcrypt.compare("MatKhauSieuKho123!", hashedPassword);
if (isMatch) {
  console.log("Đăng nhập thành công!");
} else {
  console.log("Sai mật khẩu!");
}
```

---

## 5. Bộ Ba Bảo Mật Web Cốt Lõi: CORS, XSS & CSRF

### 5.1. CORS (Cross-Origin Resource Sharing)
Mặc định, trình duyệt chặn các request JavaScript được gửi từ một **Origin (Nguồn gốc)** này tới một Origin khác để bảo vệ an toàn (chính sách Same-Origin Policy):
- Frontend chạy tại: `http://localhost:5173`
- Backend API chạy tại: `http://localhost:3000`
- Khác cổng port => Trình duyệt sẽ chặn và báo lỗi CORS!

**Giải pháp:** Backend phải chủ động gửi Header cho phép Frontend truy cập:
```javascript
import cors from 'cors';
app.use(cors({
  origin: 'http://localhost:5173', // Chỉ cho phép duy nhất frontend này gọi tới
  credentials: true
}));
```

---

### 5.2. XSS (Cross-Site Scripting)
- Kẻ tấn công tiêm mã JavaScript độc hại vào website (ví dụ gửi mã script trong phần bình luận bài viết). Khi người dùng khác mở bài viết, mã script đó tự chạy và gửi trộm dữ liệu LocalStorage (chứa JWT Token) về server của hacker!
- **Phòng chống:**
  1. Luôn escape và kiểm duyệt dữ liệu người dùng nhập (Sanitization).
  2. Lưu JWT Token trong **`HttpOnly Cookie`** thay vì `localStorage` (Trình duyệt chặn không cho bất kỳ mã JavaScript nào đọc được `HttpOnly Cookie`).

---

### 5.3. CSRF (Cross-Site Request Forgery)
- Kẻ tấn công lừa bạn click vào một liên kết độc hại trên website A trong khi bạn vẫn đang đăng nhập tài khoản Ngân hàng trên website B. Trình duyệt tự động đính kèm Cookie ngân hàng hợp lệ để thực hiện lệnh chuyển tiền!
- **Phòng chống:**
  1. Sử dụng thuộc tính Cookie: `SameSite=Strict` hoặc `SameSite=Lax`.
  2. Sử dụng mã **CSRF Token** xác thực cho mỗi lần submit form.

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. Luôn dùng **Connection Pool** để tái sử dụng kết nối CSDL và dùng **Parameterized Queries** để loại bỏ 100% nguy cơ SQL Injection.
2. **ORM (như Prisma)** giúp code Type-safe và phát triển nhanh, nhưng cần cẩn trọng để tránh bài toán N+1 Query.
3. Phân biệt **Authentication** (xác định danh tính) và **Authorization** (kiểm tra quyền hạn).
4. **JWT** là cơ chế xác thực phi trạng thái (Stateless), dùng chữ ký mật mã để đảm bảo dữ liệu Payload không bị làm giả.
5. Luôn băm mật khẩu bằng **bcrypt** kèm muối (Salt) trước khi lưu vào CSDL.
6. Cấu hình **CORS** chính xác ở Backend và bảo vệ Token khỏi **XSS** bằng cờ `HttpOnly Cookie`.
