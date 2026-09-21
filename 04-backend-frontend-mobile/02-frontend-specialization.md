# 02 - Chuyên Sâu Kỹ Sư Frontend (Frontend Engineering Specialization)

> **Mục tiêu bài học:** Nâng tầm kỹ năng Frontend từ các ứng dụng React cơ bản lên đẳng cấp sản xuất chuyên nghiệp: làm chủ Next.js và các cơ chế Rendering (SSR, SSG, ISR, React Server Components), phân tách triệt để Client State (Zustand) và Server State (TanStack Query), cùng với nghệ thuật tối ưu hóa hiệu năng theo chuẩn Google Core Web Vitals.

---

## 1. Cuộc Cách Mạng Frontend: Từ React SPA Sang Next.js

Trong nhiều năm, các ứng dụng React thuần túy hoạt động theo mô hình **CSR (Client-Side Rendering)**:
- Trình duyệt tải về một file `index.html` gần như rỗng tuếch chỉ có `<div id="root"></div>`, cùng một file JavaScript khổng lồ (vài Megabytes).
- Thiết bị của người dùng phải mất từ 3 đến 5 giây để tải, giải mã và tự dựng giao diện.
- **Hậu quả:** Bọ tìm kiếm của Google (SEO) không đọc được nội dung thực tế, và người dùng ở vùng mạng 3G/4G yếu sẽ bỏ đi vì màn hình trắng quá lâu.

**Next.js** (do Vercel phát triển) giải quyết triệt để bài toán này bằng cách đưa quá trình render một phần hoặc toàn bộ về phía **Máy chủ (Server)**.

---

## 2. Bốn Cơ Chế Rendering Cốt Lõi Trong Next.js

Hiểu rõ 4 cơ chế này giúp bạn tối ưu hóa tốc độ và chi phí vận hành website:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        4 CƠ CHẾ RENDERING TRÊN WEB                     │
└───────┬──────────────────┬───────────────────┬──────────────────┬──────┘
        │                  │                   │                  │
        ▼                  ▼                   ▼                  ▼
      1. CSR             2. SSR              3. SSG             4. ISR
  (Client-Side)      (Server-Side)       (Static Site)     (Incremental)
  Trình duyệt tự     Mỗi request tạo     Build sẵn file    Trang tĩnh nhưng
  chạy JS để vẽ      HTML trên server    HTML tĩnh một     tự động cập nhật
  giao diện.         rồi mới gửi về.     lần duy nhất.     ngầm theo chu kỳ!
```

### Bảng so sánh chi tiết:

| Cơ chế | Thời điểm tạo HTML | Tốc độ tải trang | Khả năng SEO | Khi nào nên sử dụng? |
| :--- | :--- | :--- | :--- | :--- |
| **CSR** *(Client-Side)* | Lúc chạy trên trình duyệt | Ban đầu chậm, sau đó mượt | Kém | Trang Dashboard quản trị nội bộ, cài đặt cá nhân, trang yêu cầu đăng nhập |
| **SSR** *(Server-Side)* | Mỗi khi có request từ Client | Nhanh, server tốn CPU | Tuyệt vời | Trang kết quả tìm kiếm thời gian thực, bảng tin thay đổi liên tục theo từng giây |
| **SSG** *(Static)* | Tại thời điểm Build (Deploy) | **Siêu tốc (Lưu trên CDN)** | Hoàn hảo | Trang Blog, Giới thiệu công ty, Tài liệu Documentation, Landing Page |
| **ISR** *(Incremental)* | Build tĩnh + Tạo lại ngầm định kỳ | **Siêu tốc trên CDN** | Hoàn hảo | Trang chi tiết sản phẩm Thương mại điện tử (Cần cập nhật giá/tồn kho mỗi 60s) |

---

## 3. Kiến Trúc App Router & React Server Components (RSC)

Từ Next.js 13/14 trở đi, mô hình **React Server Components (RSC)** đã thay đổi hoàn toàn cách chúng ta viết code React:

```text
                           CÂY COMPONENT TRONG NEXT.JS
┌────────────────────────────────────────────────────────────────────────┐
│ SERVER COMPONENT (Mặc định)                                            │
│ - Chạy 100% trên Server Node.js.                                       │
│ - Có thể gọi trực tiếp Database: `const users = await db.users.find()` │
│ - KHÔNG gửi mã JavaScript của thư viện xuống trình duyệt (Giảm Bundle) │
│ - Mã nguồn và API Keys an toàn tuyệt đối!                              │
│                                                                        │
│   ┌──────────────────────────────────────────────────────────────────┐ │
│   │ CLIENT COMPONENT (Đánh dấu bằng 'use client')                    │ │
│   │ - Tải mã JS xuống trình duyệt để người dùng tương tác.           │ │
│   │ - Dùng khi cần: `onClick`, `onChange`, `useState`, `useEffect`.  │ │
│   └──────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

> [!TIP]
> **Nguyên tắc vàng:** Mặc định hãy giữ tất cả các Component là **Server Component**. Chỉ chuyển sang **Client Component (`'use client'`)** khi thành phần đó thực sự cần bắt sự kiện người dùng (nút bấm, form nhập liệu) hoặc dùng React Hooks!

---

## 4. Quản Lý Trạng Thái Hiện Đại: Phân Tách Client vs Server State

Nhiều lập trình viên trước đây có thói quen "nhét tất cả mọi thứ vào Redux". Tư duy hiện đại đã phân tách rõ rệt thành 2 bài toán hoàn toàn khác nhau:

```text
QUẢN LÝ TRẠNG THÁI HIỆN ĐẠI (MODERN STATE MANAGEMENT)
 ├── 1. CLIENT STATE (Zustand / Redux Toolkit)
 │      Chỉ lưu trạng thái UI thuần túy: Mở/Đóng Sidebar, Bật Dark/Light Mode, Giỏ hàng tạm.
 │
 └── 2. SERVER STATE (TanStack Query / React Query)
        Quản lý dữ liệu từ API: Danh sách sản phẩm, Trang cá nhân, Caching, Tự fetch lại.
```

### 4.1. TanStack Query (React Query) - Đỉnh cao xử lý Server State
Thay vì viết hàng chục dòng `useState`, `useEffect`, `try-catch`, và biến cờ `loading` rối rắm, React Query giải quyết mọi việc chỉ trong 3 dòng:

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchProductsList() {
  const res = await fetch('/api/v1/products');
  return res.json();
}

export function ProductsView() {
  // Tự động: Caching dữ liệu trong RAM, tự gọi lại khi chuyển tab, tự quản lý loading & error!
  const { data, isLoading, isError } = useQuery({
    queryKey: ['products'],
    queryFn: fetchProductsList,
    staleTime: 1000 * 60 * 5, // Dữ liệu được coi là mới trong 5 phút, không cần fetch lại!
  });

  if (isLoading) return <p>Đang tải sản phẩm...</p>;
  if (isError) return <p>Có lỗi xảy ra khi tải dữ liệu!</p>;

  return (
    <div>
      {data.map((p: any) => (
        <div key={p.id}>{p.name} - {p.price}$</div>
      ))}
    </div>
  );
}
```

---

## 5. UI/UX Engineering & Thước Đo Google Core Web Vitals

Một trang web đẹp chưa chắc đã là một trang web thành công. Google đánh giá thứ hạng website dựa trên trải nghiệm thực tế của người dùng thông qua **Core Web Vitals**:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                      3 CHỈ SỐ GOOGLE CORE WEB VITALS                    │
└───────┬──────────────────────────┬───────────────────────────┬─────────┘
        │                          │                           │
        ▼                          ▼                           ▼
  1. LCP (< 2.5s)            2. INP (< 200ms)            3. CLS (< 0.1)
  (Largest Contentful Paint) (Interaction to Next Paint) (Cumulative Layout Shift)
  Thời gian vẽ khối nội dung  Độ trễ phản hồi khi người  Độ giật/nhảy bố cục bất
  lớn nhất màn hình.          dùng click chuột/gõ phím.  ngờ khi ảnh vừa tải xong.
```

### Các kỹ thuật tối ưu hóa hiệu năng bắt buộc phải làm:
1. **Tối ưu hình ảnh với `next/image`:** Tự động chuyển đổi ảnh sang định dạng WebP/AVIF siêu nhẹ, nén ảnh theo kích thước màn hình và tự động Lazy Load (chỉ tải ảnh khi người dùng cuộn tới).
2. **Khắc phục CLS (Layout Shift):** Luôn đặt thuộc tính `width` và `height` rõ ràng cho các thẻ ảnh và khung video để trình duyệt giữ chỗ trước, không làm nhảy dòng chữ bên dưới khi ảnh xuất hiện.
3. **Code Splitting & Dynamic Imports:** Không tải toàn bộ code một lúc. Tách các component nặng (như biểu đồ Chart hoặc bộ soạn thảo Markdown) ra và chỉ tải khi người dùng cần dùng:
   ```tsx
   import dynamic from 'next/dynamic';
   const HeavyChart = dynamic(() => import('@/components/HeavyChart'), { ssr: false });
   ```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **Next.js** khắc phục nhược điểm của React SPA truyền thống bằng cách hỗ trợ linh hoạt **SSR, SSG, và ISR**.
2. Tận dụng sức mạnh của **React Server Components (RSC)** để gọi dữ liệu trực tiếp trên Server và giảm tối đa dung lượng bundle JS gửi xuống Client.
3. Tách biệt hoàn toàn **Client State (dùng Zustand)** và **Server State (dùng TanStack Query)**.
4. Xây dựng giao diện nhanh chóng, thẩm mỹ cao với **TailwindCSS** và **shadcn/ui**.
5. Luôn theo dõi và tối ưu 3 chỉ số **Core Web Vitals (LCP, INP, CLS)** để mang lại trải nghiệm mượt mà nhất cho người dùng và đạt thứ hạng SEO cao trên Google.
