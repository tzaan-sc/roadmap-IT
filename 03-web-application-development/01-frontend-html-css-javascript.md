# 01 - Nền Tảng Web: HTML5, CSS3 Hiện Đại & JavaScript (Frontend Core)

> **Mục tiêu bài học:** Nắm vững kiềng 3 chân của mọi trang web: sử dụng HTML5 Semantic để dựng khung xương chuẩn SEO và Accessibility, làm chủ CSS Flexbox/Grid và Responsive Mobile-First để tạo giao diện đẹp mắt, cùng với JavaScript hiện đại để thao tác DOM và xử lý bất đồng bộ với `async/await` & Fetch API.

---

## 1. Trình Duyệt Web Hoạt Động Như Thế Nào?

Khi bạn gõ `https://mysite.com` vào thanh địa chỉ trình duyệt, một chuỗi sự kiện diễn ra ở tầng sâu:

```text
[ TRÌNH DUYỆT (BROWSER) ]                                [ MÁY CHỦ (WEB SERVER) ]
        │                                                           │
        │─── 1. Gửi HTTP GET Request ──────────────────────────────►│
        │                                                           │
        │◄── 2. Trả về file HTML thô ───────────────────────────────│
        │                                                           │
        ├─► Phân tích HTML tạo thành cây DOM (Document Object Model)│
        │                                                           │
        │─── 3. Tải tiếp các file style.css, script.js, ảnh ────────►│
        │                                                           │
        ├─► Phân tích CSS tạo thành cây CSSOM (CSS Object Model)   │
        ├─► Ghép DOM + CSSOM = CÂY RENDER (Render Tree)             │
        ├─► Layout: Tính toán kích thước và vị trí pixel            │
        └─► Paint: Vẽ các pixel màu lên màn hình cho bạn nhìn thấy! │
```

- **HTML (HyperText Markup Language):** Bộ khung xương và cấu trúc nội dung.
- **CSS (Cascading Style Sheets):** Lớp da thịt, trang phục, màu sắc và bố cục thẩm mỹ.
- **JavaScript:** Hệ cơ và thần kinh, giúp trang web cử động, tương tác và phản hồi hành vi của người dùng.

---

## 2. HTML5 Semantic: Chuẩn SEO & Trợ Năng (Accessibility)

Nhiều người mới học thường lạm dụng thẻ `<div>` cho mọi thành phần (hiện tượng "Div Soup"). **Semantic HTML** sinh ra để gắn nhãn ngữ nghĩa rõ ràng cho từng khối nội dung:

```text
┌────────────────────────────────────────────────────────┐
│ <header> : Logo, Khẩu hiệu, Tiêu đề đầu trang          │
├────────────────────────────────────────────────────────┤
│ <nav>    : Thanh menu điều hướng (Trang chủ, Tin tức)  │
├────────────────────────────┬───────────────────────────┤
│ <main> : Nội dung chính    │ <aside> : Thanh bên cạnh  │
│ ┌────────────────────────┐ │ (Quảng cáo, Tin liên quan)│
│ │ <article> : Bài viết   │ │                           │
│ │ ┌────────────────────┐ │ │                           │
│ │ │ <section> : Mục con│ │ │                           │
│ │ └────────────────────┘ │ │                           │
│ └────────────────────────┘ │                           │
├────────────────────────────┴───────────────────────────┤
│ <footer> : Bản quyền, Thông tin liên hệ, Mạng xã hội   │
└────────────────────────────────────────────────────────┘
```

### Tại sao bắt buộc phải dùng HTML Semantic?
1. **Tối ưu hóa công cụ tìm kiếm (SEO):** Bọ tìm kiếm của Google (Googlebot) hiểu được đâu là nội dung bài viết chính (`<article>`), đâu là tiêu đề quan trọng (`<h1>` đến `<h6>`), giúp website đạt thứ hạng cao hơn.
2. **Khả năng tiếp cận (Accessibility - A11y):** Các phần mềm đọc màn hình (Screen Readers) dành cho người khiếm thị dựa vào các thẻ ngữ nghĩa này để điều hướng bằng giọng nói.
3. **Mã nguồn sạch, dễ bảo trì:** Đồng đội nhìn vào cấu trúc code là hiểu ngay vị trí của từng thành phần.

### Biểu mẫu (Forms) chuẩn mực:
```html
<form id="loginForm" method="POST" action="/api/login">
  <!-- Luôn gắn thẻ <label> với <input> qua thuộc tính 'for' và 'id' -->
  <div class="form-group">
    <label for="userEmail">Địa chỉ Email:</label>
    <input 
      type="email" 
      id="userEmail" 
      name="email" 
      placeholder="ban@example.com" 
      required 
    />
  </div>

  <div class="form-group">
    <label for="userPassword">Mật khẩu:</label>
    <input 
      type="password" 
      id="userPassword" 
      name="password" 
      minlength="8" 
      required 
    />
  </div>

  <button type="submit">Đăng Nhập</button>
</form>
```

---

## 3. CSS3 Hiện Đại: Box Model, Flexbox & CSS Grid

### 3.1. Mô Hình Hộp (CSS Box Model)
Mọi phần tử trên trang web đều là một chiếc hộp hình chữ nhật gồm 4 lớp từ trong ra ngoài:

```text
┌──────────────────────────────────────────────┐
│ MARGIN (Khoảng cách với các hộp bên ngoài)   │
│  ┌────────────────────────────────────────┐  │
│  │ BORDER (Đường viền khung của hộp)      │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │ PADDING (Khoảng đệm từ viền đến  │  │  │
│  │  │          nội dung bên trong)     │  │  │
│  │  │  ┌────────────────────────────┐  │  │  │
│  │  │  │ CONTENT (Nội dung chữ/ảnh) │  │  │  │
│  │  │  └────────────────────────────┘  │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

> [!TIP]
> **Quy tắc vàng của mọi lập trình viên CSS chuyên nghiệp:**
> Mặc định trong CSS cũ, khi bạn đặt `width: 200px` và thêm `padding: 20px`, chiều rộng thực tế của hộp sẽ bị phình to thành `240px` làm vỡ layout. Để khắc phục triệt để, luôn thêm dòng này ở đầu file CSS của bạn:
> ```css
> *, *::before, *::after {
>   box-sizing: border-box; /* Padding và Border sẽ nằm gọn bên trong width! */
>   margin: 0;
>   padding: 0;
> }
> ```

---

### 3.2. Bố Cục 1 Chiều Siêu Linh Hoạt: CSS Flexbox

Flexbox (`display: flex`) được thiết kế để căn chỉnh các phần tử theo **một chiều duy nhất** (theo hàng ngang hoặc theo cột dọc).

```text
       MAIN AXIS (Trục chính - Mặc định nằm ngang: justify-content)
 ────────────────────────────────────────────────────────────────────────►
 ┌─────────────────────────────────────────────────────────────────────┐
 │ ┌─────────────┐       ┌─────────────┐       ┌─────────────┐         │ ▲
 │ │   Item 1    │       │   Item 2    │       │   Item 3    │         │ │ CROSS AXIS
 │ └─────────────┘       └─────────────┘       └─────────────┘         │ │ (Trục phụ:
 └─────────────────────────────────────────────────────────────────────┘ ▼ align-items)
```

```css
.navbar {
  display: flex;
  flex-direction: row;          /* Xếp theo hàng ngang (hoặc 'column' nếu muốn dọc) */
  justify-content: space-between;/* Đẩy Logo sang kịch trái, Menu sang kịch phải */
  align-items: center;          /* Căn giữa các item theo chiều dọc (Cross Axis) */
  gap: 20px;                    /* Khoảng cách chuẩn giữa các item */
}
```

---

### 3.3. Bố Cục 2 Chiều Toàn Diện: CSS Grid

CSS Grid (`display: grid`) là hệ thống bố cục 2 chiều (cả hàng lẫn cột) mạnh mẽ nhất của CSS, lý tưởng cho việc tạo lưới sản phẩm, dashboard:

```css
.product-grid {
  display: grid;
  /* Phép màu tự động co giãn số cột theo kích thước màn hình: */
  /* Tự chia cột: Mỗi cột tối thiểu 280px, tối đa 1 phần đều nhau (1fr) */
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}
```

---

### 3.4. Thiết Kế Thích Ứng (Responsive Web Design) & Mobile-First

**Mobile-First (Ưu tiên thiết bị di động):** Viết CSS cho màn hình điện thoại trước (không dùng media query), sau đó dùng Media Query `@media (min-width: ...)` để bổ sung giao diện cho Tablet và Desktop khi màn hình rộng ra:

```css
/* 1. Mặc định cho Mobile (< 768px): Hiển thị 1 cột duy nhất */
.container {
  width: 100%;
  padding: 16px;
}

/* 2. Màn hình Tablet & Laptop (>= 768px) */
@media (min-width: 768px) {
  .container {
    max-width: 720px;
    margin: 0 auto;
  }
}

/* 3. Màn hình Desktop lớn (>= 1200px) */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
  }
}
```

---

## 4. JavaScript Tương Tác DOM & Xử Lý Sự Kiện

**DOM (Document Object Model)** là một cấu trúc cây các đối tượng đại diện cho file HTML, cho phép JavaScript truy cập và chỉnh sửa giao diện lúc runtime.

```javascript
// 1. Lựa chọn phần tử trên trang
const submitBtn = document.querySelector("#submitBtn");
const messageBox = document.querySelector(".message-box");

// 2. Lắng nghe sự kiện người dùng click
submitBtn.addEventListener("click", (event) => {
  event.preventDefault(); // Ngăn chặn hành vi mặc định (ví dụ reload lại trang)
  
  // 3. Thay đổi giao diện động
  messageBox.textContent = "Dữ liệu đang được gửi đi...";
  messageBox.classList.add("active"); // Thêm class CSS
});
```

> [!CAUTION]
> **Cảnh báo bảo mật XSS (Cross-Site Scripting):**
> Hạn chế dùng `element.innerHTML = userInput` vì nếu người dùng cố ý nhập `<script>mã_độc()</script>`, mã đó sẽ được trình duyệt thực thi! Hãy ưu tiên dùng **`element.textContent`** để an toàn tuyệt đối.

---

## 5. JavaScript Bất Đồng Bộ: Promises & `async/await`

Trình duyệt là môi trường đơn luồng (Single-threaded). Nếu một thao tác mạng mất 3 giây mà chạy đồng bộ (blocking), toàn bộ giao diện sẽ bị đơ cứng, người dùng không thể bấm chuột hay cuộn trang!

```text
ĐỒNG BỘ (BLOCKING) ──────► GỌI API MẠNG (Đơ toàn bộ web 3 giây!) ──► TIẾP TỤC
BẤT ĐỒNG BỘ (ASYNC) ────► ĐẨY RA NỀN XỬ LÝ ──► WEB VẪN CUỘN MƯỢT MÀ!
                                                  │
                                                  ▼ (Khi có dữ liệu thì báo về)
```

### 5.1. Cú pháp hiện đại: `async / await` & Fetch API

```javascript
// Hàm bất đồng bộ lấy danh sách người dùng từ Server API
async function loadUsersList() {
  const loadingSpinner = document.querySelector("#loading");
  
  try {
    loadingSpinner.style.display = "block";
    
    // Gửi request HTTP bằng Fetch API (Bất đồng bộ)
    const response = await fetch("https://jsonplaceholder.typicode.com/users");
    
    // Kiểm tra mã trạng thái HTTP (200 OK)
    if (!response.ok) {
      throw new Error(`Lỗi kết nối HTTP: Mã lỗi ${response.status}`);
    }
    
    // Giải mã JSON body
    const users = await response.json();
    
    // Hiển thị ra màn hình
    renderUsersToDOM(users);
    
  } catch (error) {
    console.error("Không thể tải người dùng:", error.message);
    document.querySelector("#error").textContent = "Có lỗi xảy ra khi kết nối máy chủ!";
  } finally {
    loadingSpinner.style.display = "none";
  }
}

function renderUsersToDOM(users) {
  const listContainer = document.querySelector("#usersList");
  listContainer.innerHTML = "";
  
  users.forEach(user => {
    const li = document.createElement("li");
    li.textContent = `${user.name} - ${user.email}`;
    listContainer.appendChild(li);
  });
}

// Kích hoạt hàm
loadUsersList();
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. Dùng **HTML5 Semantic** (`<header>`, `<nav>`, `<main>`, `<article>`) để trang web có cấu trúc chuẩn SEO và hỗ trợ người dùng khiếm thị.
2. Luôn đặt `box-sizing: border-box` cho mọi phần tử trong CSS.
3. Dùng **Flexbox** cho bố cục 1 chiều (navbar, căn giữa), dùng **CSS Grid** cho lưới 2 chiều (danh sách thẻ sản phẩm).
4. Áp dụng tư duy **Mobile-First**: viết CSS cho điện thoại trước, sau đó mở rộng bằng Media Query `@media (min-width: ...)`.
5. Dùng `addEventListener` để bắt sự kiện; luôn dùng **`async / await` kết hợp `try-catch`** khi gọi API mạng với `fetch()` để giao diện người dùng không bao giờ bị đơ cứng.
