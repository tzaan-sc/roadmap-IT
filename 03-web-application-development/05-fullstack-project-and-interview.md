# 05 - Dự Án Thực Chiến Fullstack & Bộ Câu Hỏi Phỏng Vấn (Fullstack Project & Interview)

> **Mục tiêu bài học:** Kết nối tất cả các mảnh ghép rời rạc thành một sản phẩm hoàn chỉnh thông qua dự án Fullstack Mini Task Manager (Backend Express REST API + Frontend React TypeScript), kèm theo bộ 15 câu hỏi phỏng vấn kỹ thuật Web Fullstack có đáp án chi tiết và Checklist đánh giá năng lực.

---

## PHẦN 1: DỰ ÁN THỰC CHIẾN — MINI TASK MANAGER FULLSTACK

Dự án này mô phỏng chính xác cấu trúc thực tế của một ứng dụng web thương mại: Frontend React giao tiếp với Backend Node/Express API qua giao thức HTTP RESTful, xử lý dữ liệu JSON và cập nhật giao diện mượt mà không cần reload trang.

```text
┌───────────────────────────┐                     ┌───────────────────────────┐
│     REACT FRONTEND        │                     │     NODE.JS BACKEND       │
│  (Port 5173 - TypeScript) │                     │     (Port 3000 - Express) │
│                           │                     │                           │
│  - Danh sách công việc    │ ── GET /api/tasks ─►│ - Đọc danh sách Task      │
│  - Form thêm việc mới     │ ◄── [ { ... } ] ────│                           │
│  - Nút check hoàn thành   │                     │                           │
│  - Nút xóa việc           │ ── POST /api/tasks ─► - Thêm Task vào bộ nhớ    │
└───────────────────────────┘                     └───────────────────────────┘
```

---

### 1.1. Mã Nguồn Backend (Node.js + Express + TypeScript)

File `backend/server.ts`:
```typescript
import express, { Request, Response } from 'express';
import cors from 'cors';

const app = express();
const PORT = 3000;

// Cấu hình Middleware
app.use(cors({ origin: 'http://localhost:5173' })); // Cho phép React gọi API
app.use(express.json());

export interface Task {
  id: number;
  title: string;
  completed: boolean;
}

// Lưu trữ dữ liệu trong RAM (Mô phỏng Database)
let tasks: Task[] = [
  { id: 1, title: 'Học cú pháp HTML Semantic', completed: true },
  { id: 2, title: 'Làm chủ CSS Flexbox và Grid', completed: true },
  { id: 3, title: 'Xây dựng ứng dụng Fullstack đầu tiên', completed: false }
];

// 1. GET /api/tasks - Lấy danh sách toàn bộ tasks
app.get('/api/tasks', (req: Request, res: Response) => {
  res.status(200).json({ success: true, data: tasks });
});

// 2. POST /api/tasks - Tạo mới một task
app.post('/api/tasks', (req: Request, res: Response) => {
  const { title } = req.body;
  if (!title || title.trim() === '') {
    return res.status(400).json({ success: false, error: 'Tiêu đề task không được để trống!' });
  }

  const newTask: Task = {
    id: Date.now(), // Tạo ID độc nhất theo timestamp
    title: title.trim(),
    completed: false
  };

  tasks.push(newTask);
  res.status(201).json({ success: true, data: newTask });
});

// 3. PATCH /api/tasks/:id/toggle - Đổi trạng thái hoàn thành
app.patch('/api/tasks/:id/toggle', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const task = tasks.find(t => t.id === id);

  if (!task) {
    return res.status(404).json({ success: false, error: 'Không tìm thấy task!' });
  }

  task.completed = !task.completed;
  res.status(200).json({ success: true, data: task });
});

// 4. DELETE /api/tasks/:id - Xóa task
app.delete('/api/tasks/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  tasks = tasks.filter(t => t.id !== id);
  res.status(204).send(); // 204 No Content
});

app.listen(PORT, () => {
  console.log(`🚀 Backend Server đang chạy tại: http://localhost:${PORT}`);
});
```

---

### 1.2. Mã Nguồn Frontend (React + TypeScript)

File `frontend/src/App.tsx`:
```tsx
import React, { useState, useEffect } from 'react';

interface Task {
  id: number;
  title: string;
  completed: boolean;
}

const API_BASE_URL = 'http://localhost:3000/api/tasks';

export function App() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [newTitle, setNewTitle] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(true);
  const [errorMessage, setErrorMessage] = useState<string | null>(null);

  // 1. Tải danh sách tasks khi vừa mở trang (Mount)
  useEffect(() => {
    fetchTasks();
  }, []);

  async function fetchTasks() {
    try {
      setLoading(true);
      const res = await fetch(API_BASE_URL);
      if (!res.ok) throw new Error('Không thể kết nối máy chủ API');
      const result = await res.json();
      setTasks(result.data);
    } catch (err: any) {
      setErrorMessage(err.message);
    } finally {
      setLoading(false);
    }
  }

  // 2. Thêm task mới
  async function handleAddTask(e: React.FormEvent) {
    e.preventDefault();
    if (!newTitle.trim()) return;

    try {
      const res = await fetch(API_BASE_URL, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title: newTitle })
      });
      const result = await res.json();
      if (res.ok) {
        setTasks(prev => [...prev, result.data]);
        setNewTitle('');
      }
    } catch (err: any) {
      alert('Lỗi thêm task: ' + err.message);
    }
  }

  // 3. Đổi trạng thái hoàn thành (Toggle)
  async function handleToggleTask(id: number) {
    try {
      const res = await fetch(`${API_BASE_URL}/${id}/toggle`, { method: 'PATCH' });
      if (res.ok) {
        setTasks(prev =>
          prev.map(t => (t.id === id ? { ...t, completed: !t.completed } : t))
        );
      }
    } catch (err: any) {
      alert('Lỗi cập nhật: ' + err.message);
    }
  }

  // 4. Xóa task
  async function handleDeleteTask(id: number) {
    try {
      const res = await fetch(`${API_BASE_URL}/${id}`, { method: 'DELETE' });
      if (res.ok) {
        setTasks(prev => prev.filter(t => t.id !== id));
      }
    } catch (err: any) {
      alert('Lỗi xóa task: ' + err.message);
    }
  }

  return (
    <main style={{ maxWidth: '600px', margin: '40px auto', fontFamily: 'sans-serif', padding: '20px' }}>
      <h1>Quản Lý Công Việc (Fullstack App)</h1>

      {/* Form thêm task mới */}
      <form onSubmit={handleAddTask} style={{ display: 'flex', gap: '10px', marginBottom: '20px' }}>
        <input
          type="text"
          value={newTitle}
          onChange={e => setNewTitle(e.target.value)}
          placeholder="Nhập tên việc cần làm..."
          style={{ flex: 1, padding: '10px', fontSize: '16px' }}
        />
        <button type="submit" style={{ padding: '10px 20px', cursor: 'pointer' }}>Thêm</button>
      </form>

      {/* Thông báo trạng thái */}
      {loading && <p>Đang tải dữ liệu...</p>}
      {errorMessage && <p style={{ color: 'red' }}>{errorMessage}</p>}

      {/* Danh sách công việc */}
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {tasks.map(task => (
          <li
            key={task.id}
            style={{
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'space-between',
              padding: '12px',
              borderBottom: '1px solid #ddd',
              textDecoration: task.completed ? 'line-through' : 'none',
              color: task.completed ? '#888' : '#000'
            }}
          >
            <span onClick={() => handleToggleTask(task.id)} style={{ cursor: 'pointer' }}>
              {task.completed ? '✅' : '⏳'} {task.title}
            </span>
            <button onClick={() => handleDeleteTask(task.id)} style={{ color: 'red', cursor: 'pointer' }}>
              Xóa
            </button>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

---

## PHẦN 2: BỘ 15 CÂU HỎI PHỎNG VẤN KỸ THUẬT WEB FULLSTACK (CÓ LỜI GIẢI)

### Câu 1: Virtual DOM trong React là gì? Thuật toán Reconciliation hoạt động như thế nào?
- **Trả lời:**
  - Virtual DOM là một bản sao bằng JavaScript Object siêu nhẹ của Real DOM được lưu trên RAM.
  - Mỗi khi State hoặc Props thay đổi, React tạo ra một cây Virtual DOM mới. Sau đó, thuật toán **Reconciliation (Diffing Algorithm)** sẽ so sánh cây Virtual DOM mới với cây cũ để tìm ra chính xác những node có sự thay đổi. Cuối cùng, React thực hiện một lượt cập nhật tập trung (batch update) lên Real DOM thật, giúp tránh việc tính toán lại bố cục (Reflow) và vẽ lại (Repaint) toàn bộ trang.

### Câu 2: Phân biệt sự khác nhau giữa Props và State trong React?
- **Trả lời:**
  - **Props (Properties):** Là dữ liệu được truyền từ Component Cha xuống Component Con theo luồng một chiều. Props là bất biến (**Read-only**), Component Con không được phép sửa đổi giá trị của props nhận vào.
  - **State:** Là dữ liệu được quản lý nội bộ bên trong chính bản thân Component đó. State có tính khả biến (**Mutable**) thông qua hàm setter. Khi State thay đổi, Component sẽ tự động được re-render lại.

### Câu 3: Quy tắc sử dụng React Hooks (Rules of Hooks) là gì?
- **Trả lời:**
  1. **Chỉ gọi Hooks ở tầng trên cùng (Top level):** Không bao giờ gọi Hooks bên trong vòng lặp (`for`), câu lệnh điều kiện (`if`), hoặc các hàm lồng nhau. Điều này đảm bảo React luôn gọi các Hooks theo đúng thứ tự cố định sau mỗi lần re-render.
  2. **Chỉ gọi Hooks từ React Functional Components** hoặc từ các **Custom Hooks**, không gọi Hooks trong các hàm JavaScript thông thường.

### Câu 4: Giải thích Dependency Array trong `useEffect`? Điều gì xảy ra nếu bỏ trống hoặc truyền mảng rỗng `[]`?
- **Trả lời:**
  - Dependency Array quyết định khi nào hàm effect được chạy lại:
    - **Không truyền mảng:** Chạy sau **mọi lần component re-render**.
    - **Truyền mảng rỗng `[]`:** Chỉ chạy **đúng 1 lần duy nhất khi component vừa mount** vào giao diện (tương đương `componentDidMount`).
    - **Truyền `[a, b]`:** Chạy lại mỗi khi giá trị của biến `a` hoặc biến `b` thay đổi.

### Câu 5: Nêu các nguyên tắc cốt lõi khi thiết kế một RESTful API?
- **Trả lời:**
  1. **Tài nguyên được định danh bằng Danh từ số nhiều:** Dùng `/api/v1/users`, `/api/v1/orders` thay vì dùng động từ (`/getUsers`, `/createOrder`).
  2. **Dùng phương thức HTTP để biểu diễn hành vi:** `GET` (đọc), `POST` (tạo mới), `PUT`/`PATCH` (sửa), `DELETE` (xóa).
  3. **Trả về đúng mã trạng thái HTTP:** `200 OK`, `201 Created`, `400 Bad Request`, `401 Unauthorized`, `404 Not Found`.
  4. **Phi trạng thái (Stateless):** Mỗi request gửi lên server phải chứa đầy đủ mọi thông tin cần thiết để xử lý (không phụ thuộc vào trạng thái lưu trước đó của server).

### Câu 6: Phân biệt phương thức `PUT` và `PATCH` trong HTTP?
- **Trả lời:**
  - **`PUT` (Thay thế toàn bộ):** Dùng để cập nhật toàn bộ đối tượng. Bạn phải gửi lên tất cả các trường dữ liệu; các trường không gửi lên có thể bị xóa hoặc đặt về giá trị mặc định.
  - **`PATCH` (Cập nhật một phần - Partial update):** Chỉ gửi lên những trường thuộc tính cụ thể cần chỉnh sửa (ví dụ: chỉ gửi `{ "price": 100 }` để đổi giá mà không cần gửi lại tên hay mô tả sản phẩm).

### Câu 7: Lỗi CORS là gì? Tại sao lỗi CORS lại xuất hiện và cách sửa ở Backend?
- **Trả lời:**
  - **CORS (Cross-Origin Resource Sharing)** là cơ chế bảo mật do trình duyệt thực thi (Same-Origin Policy), chặn các đoạn mã JavaScript gửi request sang một domain, port hoặc giao thức khác với website hiện tại.
  - **Cách sửa ở Backend:** Backend phải cấu hình trả về các HTTP Response Headers cho phép, bao gồm `Access-Control-Allow-Origin: http://localhost:5173` và `Access-Control-Allow-Methods: GET, POST, PUT, DELETE`.

### Câu 8: Phân biệt cơ chế xác thực Session-Cookie và JSON Web Token (JWT)?
- **Trả lời:**
  - **Session-Cookie (Stateful):** Dữ liệu phiên đăng nhập được lưu trên RAM/Redis của server. Trình duyệt chỉ lưu một mã Cookie Session ID. Ưu điểm: Thu hồi quyền (hủy session) tức thì. Nhược điểm: Tốn RAM server và khó scale trên kiến trúc cụm server phân tán.
  - **JWT (Stateless):** Dữ liệu user được mã hóa và ký bằng chữ ký mật mã rồi gửi cho Client tự giữ. Server không cần lưu gì trong bộ nhớ. Ưu điểm: Siêu nhẹ, scale vô hạn trên Microservices. Nhược điểm: Khó thu hồi token trước hạn hết hạn (Expiration time).

### Câu 9: Tại sao không nên lưu JWT Token trong `localStorage`?
- **Trả lời:**
  - Dữ liệu trong `localStorage` có thể bị truy cập bởi bất kỳ đoạn mã JavaScript nào đang chạy trên trang. Nếu website bị dính lỗ hổng **XSS (Cross-Site Scripting)**, hacker có thể tiêm mã độc để đọc trộm token và đánh cắp tài khoản.
  - **Giải pháp an toàn hơn:** Lưu Token trong **`HttpOnly Cookie`**. Trình duyệt sẽ tự động gửi cookie này khi gọi API nhưng nghiêm cấm mã JavaScript đọc nó, miễn nhiễm hoàn toàn với các cuộc tấn công XSS.

### Câu 10: SQL Injection là gì và làm thế nào để phòng chống?
- **Trả lời:**
  - SQL Injection là kỹ thuật tấn công chèn các đoạn mã SQL độc hại vào ô input của người dùng nhằm đánh lừa CSDL thực thi câu lệnh ngoài ý muốn (ví dụ nhập `' OR '1'='1`).
  - **Cách phòng chống:** Luôn sử dụng **Parameterized Queries (Truy vấn có tham số)** hoặc dùng các thư viện **ORM (Prisma, TypeORM)** để dữ liệu đầu vào luôn được xử lý thuần túy dưới dạng chuỗi văn bản, không bao giờ bị dịch thành mã lệnh.

### Câu 11: CSS Box Model gồm những thành phần nào? Tại sao nên đặt `box-sizing: border-box`?
- **Trả lời:**
  - Gồm 4 lớp từ trong ra ngoài: **Content** (nội dung), **Padding** (đệm trong), **Border** (viền), **Margin** (khoảng cách ngoài).
  - Đặt `box-sizing: border-box` giúp kích thước `width` và `height` của phần tử bao trọn cả phần `padding` và `border`, tránh tình trạng phần tử bị phình to ra ngoài ý muốn làm vỡ bố cục giao diện.

### Câu 12: Khi nào nên dùng Flexbox và khi nào nên dùng CSS Grid?
- **Trả lời:**
  - **Flexbox:** Thiết kế cho bố cục **1 chiều** (theo một hàng hoặc một cột). Rất thích hợp cho: Thanh điều hướng (Navbar), căn giữa phần tử, các nhóm nút bấm.
  - **CSS Grid:** Thiết kế cho bố cục **2 chiều** (cả hàng lẫn cột đồng thời). Thích hợp cho: Lưới danh sách sản phẩm, bố cục tổng thể trang web (Header, Sidebar, Content, Footer).

### Câu 13: Event Bubbling và Event Capturing trong DOM là gì?
- **Trả lời:**
  - Khi một sự kiện click xảy ra trên một phần tử con nằm sâu bên trong:
    - **Event Capturing:** Sự kiện đi từ gốc `window` xuống dần qua các thẻ cha cho tới phần tử mục tiêu.
    - **Event Bubbling (Mặc định):** Sau khi kích hoạt ở phần tử mục tiêu, sự kiện sẽ "sủi bọt" lan ngược từ dưới lên trên các thẻ cha bao quanh nó. Dùng `event.stopPropagation()` để ngăn hiện tượng sủi bọt này.

### Câu 14: Sự khác nhau giữa `Promise` và `async/await`?
- **Trả lời:**
  - `Promise` xử lý bất đồng bộ qua chuỗi hàm callback `.then().catch()`, nếu có nhiều tác vụ lồng nhau vẫn có thể gây rối code.
  - `async/await` thực chất là lớp vỏ cú pháp (Syntactic Sugar) được xây dựng bên trên Promise. Nó cho phép bạn viết mã bất đồng bộ trông giống hệt như mã đồng bộ tuần tự, dễ đọc, dễ bảo trì và dễ bắt lỗi bằng khối `try-catch`.

### Câu 15: Bài toán N+1 Query trong ORM là gì và cách khắc phục?
- **Trả lời:**
  - Xảy ra khi bạn truy vấn 1 danh sách gồm $N$ phần tử (1 query), sau đó trong vòng lặp bạn lại gọi tiếp 1 query cho mỗi phần tử để lấy dữ liệu liên quan ($N$ queries phụ). Tổng cộng tốn $1 + N$ queries tới CSDL làm sập hiệu năng.
  - **Cách khắc phục:** Sử dụng kỹ thuật **Eager Loading** (dùng `include` trong Prisma hoặc `relations` trong TypeORM) để ORM gộp lại thành câu lệnh `JOIN` hoặc dùng mệnh đề `WHERE id IN (...)` chỉ với 1 hoặc 2 query duy nhất.

---

## PHẦN 3: CHECKLIST TỰ ĐÁNH GIÁ KỸ NĂNG (LEVEL 3 COMPETENCY)

Hãy tự kiểm tra xem bạn đã thành thạo các kỹ năng của Level 3 chưa:

- [ ] Tôi biết cách dùng các thẻ HTML5 Semantic chuẩn SEO và Accessibility.
- [ ] Tôi tự tin căn chỉnh giao diện bằng CSS Flexbox và chia lưới bằng CSS Grid.
- [ ] Tôi hiểu tư duy Mobile-First và viết được Media Queries responsive.
- [ ] Tôi thành thạo JavaScript bất đồng bộ với `async/await` và Fetch API.
- [ ] Tôi giải thích được Virtual DOM và thuật toán Reconciliation của React.
- [ ] Tôi phân biệt được Props vs State và làm chủ các hooks: `useState`, `useEffect`, `useRef`.
- [ ] Tôi hiểu chu trình Request/Response của HTTP và các mã trạng thái (200, 201, 400, 401, 403, 404, 500).
- [ ] Tôi thiết kế được một hệ thống RESTful API chuẩn mực bằng danh từ số nhiều và HTTP Verbs.
- [ ] Tôi hiểu cơ chế Connection Pooling và biết cách phòng chống lỗi SQL Injection.
- [ ] Tôi phân biệt được Session-Cookie và JWT, hiểu cấu trúc 3 phần của token JWT.
- [ ] Tôi biết cách cấu hình CORS ở Backend và hiểu rủi ro bảo mật của XSS/CSRF.
- [ ] Tôi đã tự tay xây dựng thành công 1 ứng dụng Fullstack kết nối React với Backend API.
