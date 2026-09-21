# 02 - Frontend Hiện Đại: TypeScript & React (Modern Frontend Mastery)

> **Mục tiêu bài học:** Hiểu lý do tại sao TypeScript trở thành tiêu chuẩn bắt buộc trong các dự án công nghệ lớn, nắm vững tư duy thiết kế Component trong React, cơ chế hoạt động của Virtual DOM, làm chủ luồng dữ liệu (Props/State) và các React Hooks cốt lõi (`useState`, `useEffect`, `useRef`).

---

## 1. Tại Sao TypeScript Thống Trị Ngành Frontend?

JavaScript thuần là ngôn ngữ định kiểu động (**Dynamically Typed**). Điều này dẫn tới lỗi kinh điển mà mọi lập trình viên JS đều từng "khóc thét" trên môi trường sản xuất:
```text
Uncaught TypeError: Cannot read properties of undefined (reading 'title')
```

**TypeScript (TS)** là một siêu tập (Superset) của JavaScript do Microsoft phát triển. TypeScript bổ sung **Hệ thống kiểu tĩnh (Static Type System)** giúp phát hiện lỗi ngay trong lúc bạn đang gõ phím trên VS Code, trước khi code kịp chạy!

```text
       JAVASCRIPT                              TYPESCRIPT
  function getLength(str) {              function getLength(str: string): number {
    return str.length;                     return str.length;
  }                                      }
  getLength(12345); // Không báo lỗi!   getLength(12345); // BÁO ĐỎ GẠCH CHÂN NGAY!
  // Chạy thực tế -> Crash undefined!   // Argument of type 'number' is not assignable to 'string'
```

### 1.1. Kiểu dữ liệu cơ bản & Interface trong TypeScript
```typescript
// Định nghĩa kiểu dữ liệu cho một sản phẩm (Product)
interface Product {
  id: number;
  name: string;
  price: number;
  isAvailable: boolean;
  tags?: string[]; // Dấu '?' nghĩa là thuộc tính này không bắt buộc (Optional)
}

// Sử dụng Interface: IDE sẽ tự động gợi ý từng trường thuộc tính!
const laptop: Product = {
  id: 101,
  name: "MacBook Pro M3",
  price: 2000,
  isAvailable: true,
  tags: ["apple", "laptop", "tech"]
};
```

---

## 2. Tư Duy React & Bản Chất Của Virtual DOM

**React** là thư viện JavaScript phổ biến số 1 thế giới để xây dựng giao diện người dùng theo kiến trúc **SPA (Single Page Application)**.

### 2.1. Cây DOM Thật vs Virtual DOM (DOM Ảo)
- Thao tác trực tiếp trên DOM thật của trình duyệt (Real DOM) là một việc **cực kỳ tốn kém tài nguyên**, vì mỗi lần thay đổi 1 thẻ HTML, trình duyệt phải tính toán lại toàn bộ vị trí (Layout/Reflow) và vẽ lại các pixel (Repaint).
- **Virtual DOM** là một bản sao bằng JavaScript Object siêu nhẹ của cây Real DOM nằm trên bộ nhớ RAM:

```text
               KHI DỮ LIỆU THAY ĐỔI (STATE CHANGE):
┌─────────────────────────┐         ┌─────────────────────────┐
│  CÂY VIRTUAL DOM CŨ     │         │  CÂY VIRTUAL DOM MỚI    │
│  (Nằm trên RAM)         │         │  (Nằm trên RAM)         │
└────────────┬────────────┘         └────────────┬────────────┘
             │                                   │
             └─────────────────┬─────────────────┘
                               ▼
               THUẬT TOÁN ĐỐI SOÁT (RECONCILIATION / DIFFING)
             So sánh tìm xem CHÍNH XÁC node nào bị thay đổi!
                               │
                               ▼ (Chỉ cập nhật đúng 1 phần tử này!)
               CẬP NHẬT TẬP TRUNG VÀO REAL DOM THẬT
```

Nhờ cơ chế này, giao diện web React phản hồi tức thì với tốc độ cực cao kể cả khi trang web có hàng nghìn phần tử phức tạp.

---

## 3. Kiến Trúc Hướng Thành Phần (Component-Based Architecture)

Trong React, bạn không viết một file HTML dài 2,000 dòng. Bạn chia nhỏ giao diện thành các **Components (Thành phần)** độc lập, có thể tái sử dụng và ghép chúng lại với nhau như những miếng ghép Lego:

```text
                        [ ỨNG DỤNG (App) ]
                                │
         ┌──────────────────────┼──────────────────────┐
         ▼                      ▼                      ▼
    [ Navbar ]            [ ProductList ]          [ Footer ]
                                │
                  ┌─────────────┴─────────────┐
                  ▼                           ▼
           [ ProductCard ]             [ ProductCard ]
```

### 3.1. Cú pháp JSX (JavaScript XML)
JSX cho phép bạn viết cú pháp giống HTML ngay bên trong code JavaScript/TypeScript:
```tsx
// Một Component đơn giản bằng React + TypeScript
interface GreetingProps {
  userName: string;
}

export function Greeting({ userName }: GreetingProps) {
  return (
    <div className="greeting-card">
      <h2>Xin chào, {userName}!</h2>
      <p>Chào mừng bạn quay trở lại hệ thống.</p>
    </div>
  );
}
```

---

## 4. Props vs State & Luồng Dữ Liệu Một Chiều (One-Way Data Flow)

| Tiêu chí | Props (Properties) | State (Trạng thái) |
| :--- | :--- | :--- |
| **Bản chất** | Dữ liệu truyền từ **Component Cha xuống Component Con** | Dữ liệu sống **nội bộ bên trong chính Component đó** |
| **Tính khả biến** | **Bất biến (Read-only)**; con không được sửa props | **Có thể thay đổi** thông qua hàm cập nhật state |
| **Ví dụ** | Nút bấm nhận prop `label="Xóa"`, `color="red"` | Số lượt đếm trong giỏ hàng, chữ đang gõ trong ô input |

### Luồng dữ liệu một chiều (One-Way Data Flow):
Dữ liệu chỉ chảy theo một hướng duy nhất: từ trên xuống dưới (Cha ──► Con).
- Nếu Component Con muốn thay đổi dữ liệu của Cha, Cha phải truyền một **hàm callback** xuống cho Con thông qua Props. Khi Con xảy ra sự kiện, nó gọi hàm callback đó để báo cho Cha cập nhật.

---

## 5. Làm Chủ Các React Hooks Cốt Lõi

React Hooks là các hàm đặc biệt cho phép bạn sử dụng State và các tính năng của React trong Functional Components.

### 5.1. `useState` - Quản lý trạng thái thay đổi
Mỗi khi giá trị của State thay đổi thông qua hàm setter, **React sẽ tự động gọi lại hàm Component để vẽ lại giao diện mới (Re-render)**:

```tsx
import { useState } from 'react';

export function Counter() {
  // count: giá trị hiện tại; setCount: hàm cập nhật
  const [count, setCount] = useState<number>(0);

  return (
    <div>
      <p>Số lần bạn đã bấm: {count}</p>
      <button onClick={() => setCount(count + 1)}>Tăng (+1)</button>
      <button onClick={() => setCount(0)}>Đặt lại (Reset)</button>
    </div>
  );
}
```

---

### 5.2. `useEffect` - Xử lý Side Effects (Gọi API, Đăng ký sự kiện)
Side Effects là các tác vụ tương tác với thế giới bên ngoài component: gửi request mạng HTTP, chỉnh sửa title trang web, thiết lập bộ đếm thời gian (timer).

```tsx
import { useState, useEffect } from 'react';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

export function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  // useEffect nhận 2 tham số:
  // 1. Hàm effect xử lý logic
  // 2. Dependency Array []: Xác định khi nào effect được chạy lại
  useEffect(() => {
    async function fetchTodos() {
      try {
        const res = await fetch('https://jsonplaceholder.typicode.com/todos?_limit=5');
        const data: Todo[] = await res.json();
        setTodos(data);
      } catch (err) {
        console.error('Lỗi khi tải todos:', err);
      } finally {
        setLoading(false);
      }
    }

    fetchTodos();
  }, []); // [] rỗng nghĩa là CHỈ CHẠY ĐÚNG 1 LẦN DUY NHẤT khi component vừa xuất hiện (Mount)!

  if (loading) return <p>Đang tải dữ liệu...</p>;

  return (
    <ul>
      {todos.map((todo) => (
        // Bắt buộc phải có prop 'key' duy nhất cho mỗi item trong danh sách!
        <li key={todo.id}>
          {todo.completed ? '✅' : '⏳'} {todo.title}
        </li>
      ))}
    </ul>
  );
}
```

> [!IMPORTANT]
> **Quy tắc của Dependency Array trong `useEffect`:**
> - `useEffect(fn)` (Không có mảng): Chạy **sau mỗi lần component re-render** (rất dễ gây vòng lặp vô hạn!).
> - `useEffect(fn, [])` (Mảng rỗng): Chỉ chạy **1 lần duy nhất** lúc component mount.
> - `useEffect(fn, [userId])`: Chạy lại mỗi khi giá trị biến `userId` thay đổi.

---

### 5.3. `useRef` - Lưu trữ giá trị không gây Re-render
Khác với `useState`, khi bạn thay đổi giá trị của `ref.current`, **React KHÔNG re-render lại component**. Ngoài ra, `useRef` thường dùng để truy cập trực tiếp vào phần tử DOM (như tự động focus con trỏ vào ô input):

```tsx
import { useRef } from 'react';

export function AutoFocusInput() {
  // Gắn tham chiếu vào phần tử HTML input
  const inputRef = useRef<HTMLInputElement>(null);

  const handleClick = () => {
    // Focus con trỏ chuột vào ô input mà không cần document.getElementById
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Nhập tên của bạn..." />
      <button onClick={handleClick}>Kích hoạt con trỏ chuột</button>
    </div>
  );
}
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **TypeScript** bổ sung hệ thống kiểu tĩnh cho JavaScript, loại bỏ các lỗi truy cập `undefined` lúc runtime.
2. React sử dụng **Virtual DOM** và thuật toán Reconciliation để chỉ cập nhật những phần tử thực sự thay đổi trên Real DOM.
3. Chia giao diện thành các **Components** tái sử dụng với cú pháp **JSX**.
4. **Props** truyền dữ liệu từ cha xuống con (bất biến); **State** lưu trữ dữ liệu nội bộ thay đổi theo thời gian.
5. Luôn nhớ thuộc tính **`key`** duy nhất khi render danh sách bằng `.map()`.
6. Làm chủ 3 hooks cốt lõi: **`useState`** (dữ liệu động), **`useEffect`** (gọi API / side effects), và **`useRef`** (truy cập DOM / giữ giá trị không re-render).
