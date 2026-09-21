# 03 - Kiến Trúc Backend & Thiết Kế RESTful API (Backend & REST APIs)

> **Mục tiêu bài học:** Thấu hiểu vai trò của máy chủ Backend trong ứng dụng hiện đại, giải phẫu giao thức HTTP và chu trình Request/Response, nắm vững ý nghĩa các mã trạng thái (HTTP Status Codes), làm chủ quy chuẩn thiết kế RESTful API chuyên nghiệp và có cái nhìn tổng quan về các hệ sinh thái Backend phổ biến (Node.js, Python, Java).

---

## 1. Vai Trò Của Backend & Kiến Trúc Phân Tầng (Layered Architecture)

Nếu Frontend là mặt tiền của một nhà hàng (bàn ghế, menu đẹp mắt, ánh sáng), thì **Backend là khu bếp chuyên nghiệp và kho nguyên liệu**:
- **Bảo mật tuyệt đối:** Mã nguồn và logic kiểm tra quyền hạn nằm trên server; người dùng ở trình duyệt không thể xem trộm mã Backend hay can thiệp vào quy trình thanh toán.
- **Xử lý nghiệp vụ (Business Logic):** Tính toán đơn hàng, trừ kho, gửi email thông báo, liên kết cổng thanh toán ngân hàng.
- **Quản lý dữ liệu:** Giao tiếp với Cơ sở dữ liệu, quản lý Cache (Redis) và đảm bảo tính nhất quán của dữ liệu.

```text
                           KIẾN TRÚC BACKEND 3 TẦNG CHUẨN
┌────────────────────────────────────────────────────────────────────────┐
│ 1. CONTROLLER / ROUTER LAYER (Tầng tiếp nhận)                          │
│ - Nhận HTTP Request từ Client (React, Mobile App).                     │
│ - Kiểm tra tính hợp lệ của dữ liệu đầu vào (Validation).               │
│ - Trả về HTTP Response và mã trạng thái (200, 400...).                 │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 2. SERVICE LAYER (Tầng xử lý nghiệp vụ - Business Logic)               │
│ - Trái tim của hệ thống: Tính tiền, áp mã giảm giá, kiểm tra số dư.   │
│ - Hoàn toàn độc lập với giao thức HTTP (Dễ dàng viết Unit Test).      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 3. REPOSITORY / DATA ACCESS LAYER (Tầng truy cập dữ liệu)             │
│ - Giao tiếp trực tiếp với Database (SQL / ORM).                        │
│ - Chạy các câu lệnh SELECT, INSERT, UPDATE, DELETE.                   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Giao Thức HTTP: Chu Trình Request - Response

Mọi giao tiếp trên Web giữa Client và Backend đều tuân theo giao thức **HTTP (HyperText Transfer Protocol)**:

```text
[ CLIENT (React App) ]                                  [ BACKEND (API Server) ]
         │                                                         │
         │──── HTTP REQUEST: POST /api/v1/products ───────────────►│
         │     Headers: Content-Type: application/json             │
         │     Body: {"name": "Bàn phím cơ", "price": 50}          │
         │                                                         │
         │                                                         │ (Xử lý lưu DB...)
         │                                                         │
         │◄─── HTTP RESPONSE: 201 Created ─────────────────────────│
         │     Headers: Content-Type: application/json             │
         │     Body: {"id": 12, "name": "Bàn phím cơ", "price": 50}│
```

---

## 3. Bản Đồ Mã Trạng Thái HTTP (HTTP Status Codes)

Hiểu đúng mã trạng thái giúp Frontend và Backend giao tiếp ăn ý và chuyên nghiệp:

| Nhóm mã | Tên gọi | Các mã cốt lõi cần nhớ | Ý nghĩa thực tế |
| :--- | :--- | :--- | :--- |
| **`2xx`** | **Thành công (Success)** | **`200 OK`**<br>**`201 Created`**<br>**`204 No Content`** | Request thành công bình thường.<br>Tạo mới tài nguyên thành công (sau lệnh POST).<br>Thành công nhưng không có body trả về (thường dùng cho DELETE). |
| **`3xx`** | **Chuyển hướng (Redirection)** | **`301 Moved Permanently`**<br>**`304 Not Modified`** | Trang web đã chuyển vĩnh viễn sang địa chỉ mới.<br>Dữ liệu chưa thay đổi, Client hãy dùng bản Cache trong máy. |
| **`4xx`** | **Lỗi phía Client (Client Error)** | **`400 Bad Request`**<br>**`401 Unauthorized`**<br>**`403 Forbidden`**<br>**`404 Not Found`**<br>**`422 Unprocessable`** | Dữ liệu gửi lên sai định dạng cú pháp.<br>**Chưa đăng nhập** (Thiếu hoặc sai Token).<br>**Đã đăng nhập nhưng không có quyền** (Ví dụ user thường đòi vào trang Admin).<br>Không tìm thấy tài nguyên theo URL.<br>Dữ liệu đúng cú pháp JSON nhưng vi phạm logic nghiệp vụ (ví dụ tuổi âm). |
| **`5xx`** | **Lỗi phía Server (Server Error)** | **`500 Internal Error`**<br>**`502 Bad Gateway`**<br>**`503 Service Unavailable`**<br>**`504 Gateway Timeout`** | Code Backend bị crash hoặc dính ngoại lệ chưa được xử lý.<br>Server Nginx không thể kết nối tới app Node/Python ở phía sau.<br>Server bị quá tải hoặc đang bảo trì.<br>Xử lý quá lâu dẫn đến hết thời gian chờ (Timeout). |

---

## 4. Thiết Kế Chuẩn Mực RESTful API

**REST (Representational State Transfer)** là kiến trúc thiết kế API phổ biến nhất thế giới.

### 4.1. Quy tắc 1: Định danh bằng DANH TỪ SỐ NHIỀU, không dùng ĐỘNG TỪ
URL chỉ dùng để định danh **Tài nguyên (Resources)**. Hành động thực hiện trên tài nguyên đó được xác định bởi **Phương thức HTTP (HTTP Verbs)**:

```text
❌ CÁCH THIẾT KẾ SAI LẦM (Rối rắm, lạm dụng URL):
GET    /api/getAllProducts
POST   /api/createProduct
POST   /api/updateProductById?id=5
GET    /api/deleteProduct?id=5

✅ THIẾT KẾ CHUẨN RESTFUL (Thanh lịch, khoa học):
GET    /api/v1/products         ──► Lấy danh sách sản phẩm
POST   /api/v1/products         ──► Tạo mới một sản phẩm
GET    /api/v1/products/5       ──► Lấy chi tiết sản phẩm có id = 5
PUT    /api/v1/products/5       ──► Cập nhật toàn bộ thông tin sản phẩm 5
PATCH  /api/v1/products/5       ──► Cập nhật một phần (ví dụ chỉ đổi giá)
DELETE /api/v1/products/5       ──► Xóa sản phẩm 5
```

### 4.2. Quy tắc 2: Tài nguyên lồng nhau (Nested Resources)
Khi tài nguyên này là con của tài nguyên khác trong mối quan hệ 1-N:
```text
GET /api/v1/users/10/orders      ──► Lấy danh sách các đơn hàng của user số 10
POST /api/v1/posts/42/comments   ──► Thêm bình luận mới vào bài viết số 42
```

### 4.3. Quy tắc 3: Tính Lũy Thừa (Idempotency)
- Một phương thức HTTP được gọi là **Idempotent** khi bạn gọi nó 1 lần hay 100 lần liên tiếp thì **trạng thái hệ thống trên server vẫn không thay đổi**:
  - `GET`, `PUT`, `DELETE`: Có tính lũy thừa (Xóa 1 phần tử rồi thì các lần xóa sau phần tử đó vẫn đã mất).
  - `POST`: **KHÔNG có tính lũy thừa** (Bấm nút gửi POST 5 lần sẽ tạo ra 5 bản ghi đơn hàng trùng lặp!).

---

## 5. So Sánh Các Hệ Sinh Thái Backend Phổ Biến

| Hệ sinh thái | Framework nổi bật | Thế mạnh vượt trội | Thích hợp nhất cho |
| :--- | :--- | :--- | :--- |
| **Node.js (JavaScript / TypeScript)** | **Express.js**, Fastify, NestJS | Dùng chung 1 ngôn ngữ với Frontend; Kiến trúc Event Loop xử lý hàng vạn kết nối đồng thời non-blocking; Hệ sinh thái `npm` khổng lồ | Ứng dụng Realtime (Chat, Socket), API Fullstack, Startup cần phát triển siêu tốc |
| **Python** | **FastAPI**, Flask, Django | Cú pháp trong sáng, tích hợp OpenAPI/Swagger tự động; Hiệu năng FastAPI cực cao nhờ Asyncio; Dễ dàng tích hợp các mô hình AI/ML | Ứng dụng AI/Data, Microservices hiện đại, Backend tích hợp Machine Learning |
| **Java** | **Spring Boot** | Độ ổn định và bảo mật tuyệt đối; Ép tuân thủ kiến trúc chặt chẽ; Tối ưu cực tốt cho khối lượng giao dịch tài chính khủng | Hệ thống Ngân hàng, Bảo hiểm, Thương mại điện tử quy mô lớn, Doanh nghiệp Enterprise |

---

## 6. Code Mẫu Xây Dựng REST API Hoàn Chỉnh

### Code mẫu bằng Node.js (Express + TypeScript):
```typescript
import express, { Request, Response } from 'express';

const app = express();
app.use(express.json()); // Middleware phân tích body JSON

interface Item {
  id: number;
  name: string;
}

let items: Item[] = [
  { id: 1, name: 'Sách Clean Code' },
  { id: 2, name: 'Bàn phím cơ' }
];

// 1. GET /api/items - Lấy danh sách
app.get('/api/items', (req: Request, res: Response) => {
  res.status(200).json({ success: true, data: items });
});

// 2. POST /api/items - Tạo mới
app.post('/api/items', (req: Request, res: Response) => {
  const { name } = req.body;
  if (!name) {
    return res.status(400).json({ success: false, error: 'Tên không được để trống!' });
  }

  const newItem: Item = { id: items.length + 1, name };
  items.push(newItem);
  res.status(201).json({ success: true, data: newItem });
});

// 3. DELETE /api/items/:id - Xóa
app.delete('/api/items/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  items = items.filter(item => item.id !== id);
  res.status(204).send(); // 204 No Content
});

app.listen(3000, () => console.log('API Server đang chạy tại port 3000'));
```

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. Backend chịu trách nhiệm xử lý **Business Logic**, bảo mật và tương tác Cơ sở dữ liệu theo **Mô hình 3 tầng (Controller - Service - Repository)**.
2. Hiểu rõ các mã trạng thái HTTP: `200` (OK), `201` (Tạo mới), `400` (Client gửi sai), `401` (Chưa login), `403` (Không đủ quyền), `404` (Không tìm thấy), `500` (Server crash).
3. Thiết kế chuẩn **RESTful**: Dùng **danh từ số nhiều** cho URL (`/api/v1/products`), dùng **phương thức HTTP** (`GET`, `POST`, `PUT`, `DELETE`) để biểu diễn hành động.
4. Nắm vững tính lũy thừa (**Idempotency**): `GET`, `PUT`, `DELETE` có tính lũy thừa; `POST` thì không.
