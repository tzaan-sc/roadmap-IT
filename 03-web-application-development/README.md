# Level 3 — Web / Application Development (Phát Triển Ứng Dụng Web Fullstack)

> *"Giai đoạn biến những dòng code trừu tượng thành sản phẩm thực tế mà hàng triệu người dùng trên khắp thế giới có thể click chuột, trải nghiệm và sử dụng."*

Chào mừng bạn đến với **Level 3** trong lộ trình [Roadmap IT](../README.md). Đây là cấp độ bước ngoặt, nơi bạn kết hợp giao diện người dùng sống động (**Frontend**) với bộ não xử lý logic và cơ sở dữ liệu (**Backend & Database**) để tạo ra các ứng dụng Web Fullstack hoàn chỉnh.

---

## 🗺️ Bản Đồ Kiến Trúc Fullstack Client - Server

```text
┌─────────────────────────────────────────────────────────────┐
│ 1. FRONTEND: CLIENT BROWSER                                 │
│ HTML5 Semantic ──► CSS3 (Flexbox & Grid, Mobile-First)      │
│ Modern React + TypeScript ──► Virtual DOM & Hooks           │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               │ HTTP / HTTPS (RESTful API)
                               │ JSON Request / Response
                               │ Headers, Auth Token (JWT)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. BACKEND: API APPLICATION SERVER                          │
│ Layered Architecture: Controllers ──► Services              │
│ Node.js (Express) / Python (FastAPI) / Java (Spring Boot)   │
│ Security: CORS, bcrypt Hash, JWT Validation                 │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               │ Connection Pool
                               │ Parameterized SQL / ORM
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. DATABASE LAYER                                           │
│ PostgreSQL / MySQL / SQLite                                 │
│ Relational Schema ──► ACID Transactions ──► B-Tree Index    │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là 5 chuyên đề chuyên sâu đã được biên soạn hoàn chỉnh trong thư mục này:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Nền Tảng Web: HTML5, CSS3 & JavaScript** | HTML5 Semantic chuẩn SEO & A11y, CSS Box Model (`box-sizing: border-box`), Bố cục linh hoạt với Flexbox & CSS Grid, Tư duy Mobile-First Responsive, Thao tác DOM, JavaScript Bất đồng bộ với `async/await` và Fetch API. | [Xem bài viết](./01-frontend-html-css-javascript.md) |
| **02** | **Frontend Hiện Đại: TypeScript & React** | Bản chất Type Safety của TypeScript, Cấu trúc SPA và bí mật tốc độ của Virtual DOM (Reconciliation), Component-Based Architecture, Props vs State, Luồng dữ liệu một chiều, Làm chủ React Hooks cốt lõi (`useState`, `useEffect`, `useRef`). | [Xem bài viết](./02-typescript-and-modern-react.md) |
| **03** | **Kiến Trúc Backend & RESTful APIs** | Kiến trúc phân 3 tầng (Controller - Service - Repository), Vòng đời bản tin HTTP Request/Response, Bản đồ ý nghĩa các mã trạng thái (2xx, 3xx, 4xx, 5xx), Quy chuẩn vàng thiết kế RESTful API bằng danh từ số nhiều, Tính lũy thừa (Idempotency), So sánh Node.js vs Python vs Java. | [Xem bài viết](./03-backend-architecture-and-rest-apis.md) |
| **04** | **Kết Nối CSDL, Xác Thực & Bảo Mật Web** | Tối ưu kết nối với Connection Pooling, Phòng chống thảm họa SQL Injection bằng Parameterized Queries, So sánh Raw SQL vs ORM, Phân biệt cơ chế xác thực Session-Cookie vs JWT (JSON Web Token), Băm mật khẩu bằng bcrypt, Bộ ba bảo mật Web: CORS, XSS, CSRF. | [Xem bài viết](./04-database-integration-and-auth.md) |
| **05** | **Dự Án Thực Chiến & Ôn Tập Phỏng Vấn** | **Dự án thực tế:** Xây dựng hoàn chỉnh ứng dụng Fullstack Mini Task Manager (Backend Express API + Frontend React TypeScript kết nối CRUD không reload trang). **Bộ 15 câu hỏi phỏng vấn kỹ thuật Web Fullstack** chuẩn mực có lời giải chi tiết và Checklist tự đánh giá. | [Xem bài viết](./05-fullstack-project-and-interview.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 3

1. **Tự tay dựng được giao diện web chuẩn chỉnh:** Không còn vỡ layout khi co kéo màn hình, thành thạo Flexbox, CSS Grid và responsive.
2. **Làm chủ thư viện React và TypeScript:** Hiểu được luồng render, không bị dính lỗi re-render vô hạn, tự tin viết các component sạch sẽ và có định kiểu an toàn.
3. **Thiết kế Backend & API chuẩn mực:** Biết cách phân chia các tầng trong backend, thiết kế các endpoint RESTful thanh lịch và trả về đúng mã trạng thái HTTP.
4. **Nắm vững bảo mật web thực tế:** Biết cách băm mật khẩu, phân quyền bằng JWT Token, cấu hình CORS và ngăn chặn triệt để các lỗ hổng SQL Injection và XSS.

---

## ⏭️ Bước Tiếp Theo
Sau khi hoàn thành dự án mẫu và tích trọn bộ Checklist trong bài 05, bạn đã đủ tự tin để chọn ngã rẽ chuyên sâu:
👉 **[Level 4: Backend / Frontend / Mobile](../04-backend-frontend-mobile/README.md)**
