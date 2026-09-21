# 05 - Đồ Án Portfolio Thực Chiến & Bộ Câu Hỏi Phỏng Vấn Chuyên Sâu (Projects & Interview)

> **Mục tiêu bài học:** Hiện thực hóa kiến thức chuyên môn thành các đồ án Portfolio ấn tượng giúp bạn vượt qua vòng lọc hồ sơ (CV Screening), kèm theo bộ 15 câu hỏi phỏng vấn kỹ thuật phân nhánh chuyên sâu (Backend, Frontend, Mobile) có lời giải chi tiết và Checklist tự đánh giá năng lực.

---

## PHẦN 1: BA ĐỒ ÁN PORTFOLIO "CHUẨN CHỈNH" GÂY ẤN TƯỢNG VỚI NHÀ TUYỂN DỤNG

Đừng đưa vào CV những bài tập cơ bản như TodoList hay máy tính bỏ túi. Nhà tuyển dụng cần nhìn thấy **khả năng giải quyết bài toán nghiệp vụ thực tế**:

```text
       NHÁNH BACKEND                      NHÁNH FRONTEND                     NHÁNH MOBILE
┌──────────────────────────┐       ┌──────────────────────────┐       ┌──────────────────────────┐
│ E-Commerce Ticketing API │       │ Next.js Analytics Dash   │       │ Offline Expense Tracker  │
│ - NestJS / FastAPI       │       │ - Next.js 14 App Router  │       │ - React Native / Flutter │
│ - PostgreSQL + Redis     │       │ - TailwindCSS + shadcn   │       │ - Local SQLite / Realm   │
│ - Xử lý Race Condition   │       │ - TanStack Query + Charts│       │ - Biometrics & Push Noti │
└──────────────────────────┘       └──────────────────────────┘       └──────────────────────────┘
```

---

### Đề Tài 1 (Dành Cho Backend): Hệ Thống API Đặt Vé Xem Phim & Chống Trùng Ghế
- **Công nghệ đề xuất:** Node.js (NestJS) hoặc Python (FastAPI) + PostgreSQL + Redis + BullMQ/Celery + Docker.
- **Thách thức kỹ thuật nổi bật cần giải quyết:**
  1. **Xử lý bất đồng thời (Race Condition):** Khi có 20 người cùng click đặt đúng 1 chiếc ghế VIP cuối cùng trong rạp tại cùng một mili-giây, làm sao để chỉ duy nhất 1 người mua thành công mà không bị trừ tiền oan? (Giải pháp: Dùng **Distributed Lock với Redis** hoặc `SELECT ... FOR UPDATE` trong Transaction của PostgreSQL).
  2. **Giữ chỗ tạm thời (Seat Reservation with TTL):** Khi người dùng chọn ghế, khóa giữ chỗ chiếc ghế đó trong vòng 10 phút. Nếu sau 10 phút người dùng không thanh toán, hệ thống tự động nhả ghế lại cho người khác (Dùng Redis Key Expiration).
  3. **Xử lý tác vụ nền:** Đẩy việc gửi vé điện tử (mã QR) qua email vào hàng đợi Message Queue để API phản hồi ngay lập tức.

---

### Đề Tài 2 (Dành Cho Frontend): Bảng Điều Khiển Phân Tích Dữ Liệu Doanh Nghiệp (Next.js Analytics Dashboard)
- **Công nghệ đề xuất:** Next.js 14/15 (App Router, Server Components) + TypeScript + TailwindCSS + shadcn/ui + TanStack Query + Recharts.
- **Thách thức kỹ thuật nổi bật cần giải quyết:**
  1. **Hiệu năng hiển thị dữ liệu lớn (Virtualization):** Hiển thị bảng danh sách 50,000 đơn hàng mà vẫn cuộn mượt mà 60 FPS mà không làm lag trình duyệt (dùng TanStack Virtual).
  2. **Bộ lọc đa chiều thời gian thực:** Kết hợp URL Search Parameters (`?page=1&status=paid&from=2024-01-01`) với Server-Side Rendering để khi người dùng copy link gửi cho đồng nghiệp, trang web hiển thị chính xác kết quả đã lọc.
  3. **Tối ưu Core Web Vitals:** Tải trước các khối giao diện quan trọng, áp dụng Skeleton Loading khi tải dữ liệu, đạt 95+ điểm trên Google PageSpeed Insights.

---

### Đề Tài 3 (Dành Cho Mobile): Ứng Dụng Quản Lý Tài Chính Cá Nhân Ngoại Tuyến (Offline-First Expense Tracker)
- **Công nghệ đề xuất:** React Native (Expo) hoặc Flutter + SQLite / WatermelonDB + Biometrics.
- **Thách thức kỹ thuật nổi bật cần giải quyết:**
  1. **Kiến trúc Offline-First:** Ứng dụng hoạt động 100% không cần mạng Internet. Mọi giao dịch thu chi được ghi tức thì vào CSDL cục bộ trên máy. Khi có kết nối mạng, hệ thống tự động đồng bộ ngầm với máy chủ đám mây.
  2. **Bảo mật sinh trắc học:** Đăng nhập an toàn bằng dấu vân tay hoặc Face ID qua phần cứng điện thoại.
  3. **Thông báo nhắc nhở cục bộ:** Lập lịch nhắc người dùng nhập chi tiêu vào 21:00 mỗi tối bằng Local Notifications mà không cần server tốn kém.

---

## PHẦN 2: BỘ 15 CÂU HỎI PHỎNG VẤN CHUYÊN MÔN PHÂN NHÁNH (CÓ LỜI GIẢI)

---

### CHUYÊN ĐỀ 1: CÂU HỎI PHỎNG VẤN BACKEND (5 CÂU)

#### Câu 1: Dependency Injection (DI) và Inversion of Control (IoC) là gì? Lợi ích khi áp dụng?
- **Trả lời:**
  - **IoC (Đảo ngược quyền điều khiển):** Thay vì một Class tự chủ động khởi tạo các đối tượng phụ thuộc bằng từ khóa `new Service()`, quyền kiểm soát việc khởi tạo đó được chuyển giao cho Framework (NestJS hoặc Spring Boot Container) quản lý.
  - **DI (Tiêm phụ thuộc):** Là một kỹ thuật triển khai IoC, trong đó các đối tượng phụ thuộc được truyền (tiêm) vào một Class thông qua Constructor hoặc Property.
  - **Lợi ích:** Giúp giảm sự phụ thuộc cứng nhắc (Loose Coupling) giữa các module, mã nguồn sạch sẽ và việc viết Unit Test trở nên vô cùng dễ dàng vì ta có thể dễ dàng "tiêm" một Mock Service giả lập vào để test.

#### Câu 2: Trình bày cơ chế hoạt động của Cache-Aside Pattern trong Redis? Làm thế nào để giải quyết vấn đề Cache Invalidation?
- **Trả lời:**
  - **Cơ chế Cache-Aside:** Khi Client gọi API, Backend kiểm tra trong Redis trước. Nếu có dữ liệu (**Cache Hit**), trả về ngay lập tức. Nếu chưa có (**Cache Miss**), truy vấn Database thật, lưu kết quả vào Redis kèm thời gian sống (**TTL - Time To Live**) rồi mới trả về cho Client.
  - **Cache Invalidation (Làm mới cache):** Khi có thao tác cập nhật hoặc xóa dữ liệu trong DB (`UPDATE` / `DELETE`), hệ thống phải chủ động xóa Key tương ứng trong Redis. Ngoài ra, việc thiết lập một khoảng thời gian TTL hợp lý (ví dụ 5 - 10 phút) đảm bảo kể cả khi code quên xóa cache thì dữ liệu cũng tự động được làm mới định kỳ.

#### Câu 3: Tại sao nên dùng Message Queue (RabbitMQ / BullMQ) thay vì xử lý trực tiếp trong HTTP request?
- **Trả lời:**
  - **Giảm độ trễ (Low Latency):** Các tác vụ tốn nhiều thời gian (gửi email, xuất file PDF, resize ảnh) nếu xử lý trực tiếp sẽ khiến người dùng phải đợi hàng chục giây và dễ bị lỗi HTTP Timeout. Đẩy vào Message Queue giúp trả lời phản hồi cho người dùng trong vòng 50ms.
  - **Giảm tải đột biến (Peak Load Shaving / Throttling):** Khi có hàng triệu đơn hàng tràn vào trong ngày siêu sale, Message Queue hoạt động như một hồ chứa nước, gom các tác vụ lại để các Worker Server rút ra xử lý từ từ theo năng lực mà không làm sập Database.
  - **Khả năng phục hồi (Fault Tolerance):** Nếu dịch vụ gửi email bên thứ 3 bị sập tạm thời, tin nhắn vẫn nằm an toàn trong Queue và hệ thống sẽ tự động thử lại (Retry) cho đến khi thành công mà không làm mất dữ liệu.

#### Câu 4: Phân biệt sự khác nhau giữa RESTful API và gRPC? Khi nào nên dùng gRPC?
- **Trả lời:**
  - **RESTful API:** Dùng định dạng văn bản JSON truyền qua HTTP/1.1 hoặc HTTP/2. Ưu điểm là cực kỳ phổ biến, dễ đọc hiểu và trực quan cho người phát triển.
  - **gRPC:** Giao thức do Google phát triển dựa trên **HTTP/2** và mã hóa dữ liệu dưới dạng **nhị phân siêu nén (Protocol Buffers)**. Nó hỗ trợ gọi hàm từ xa trực tiếp (RPC) và truyền dữ liệu liên tục 2 chiều (Bidirectional Streaming).
  - **Khi nào dùng gRPC:** Dùng cho việc giao tiếp nội bộ giữa các **Microservices** đòi hỏi tốc độ truyền dữ liệu tối đa và độ trễ cực thấp (nhanh hơn REST gấp 7-10 lần).

#### Câu 5: Làm thế nào để giải quyết bài toán Race Condition khi hai người dùng cùng bấm mua một món hàng chỉ còn 1 sản phẩm duy nhất trong kho?
- **Trả lời:**
  - Có 2 giải pháp chuẩn mực:
    1. **Pessimistic Locking (Khóa bi quan) trong Database:** Sử dụng câu lệnh `SELECT quantity FROM products WHERE id = 1 FOR UPDATE;` bên trong một Transaction. Database sẽ khóa dòng này lại; request của người dùng thứ hai buộc phải xếp hàng chờ cho đến khi transaction của người thứ nhất hoàn tất.
    2. **Distributed Lock với Redis (ví dụ Redlock):** Tạo một khóa tạm thời trên Redis `SET product_lock_1 my_token NX EX 5`. Ai chiếm được khóa trước thì được vào xử lý trừ kho, người thứ hai không chiếm được khóa sẽ nhận thông báo sản phẩm đang bận hoặc đã hết hàng.

---

### CHUYÊN ĐỀ 2: CÂU HỎI PHỎNG VẤN FRONTEND (5 CÂU)

#### Câu 6: Phân biệt sự khác nhau giữa Server Components và Client Components trong Next.js App Router?
- **Trả lời:**
  - **Server Components (Mặc định):** Chạy 100% trên môi trường Node.js của server. Có thể kết nối trực tiếp CSDL, đọc file hệ thống và không gửi mã JavaScript của thư viện xuống trình duyệt (giúp giảm đáng kể kích thước bundle JS). Không thể sử dụng các hook giao diện (`useState`, `useEffect`) hay sự kiện click.
  - **Client Components (Có khai báo `'use client'` ở đầu file):** Mã nguồn được gửi xuống trình duyệt để người dùng tương tác. Dùng khi cần bắt sự kiện người dùng (`onClick`, `onChange`), quản lý state cục bộ, hoặc truy cập Web APIs của trình duyệt (`localStorage`, `window`).

#### Câu 7: So sánh sự khác nhau giữa SSR (Server-Side Rendering) và SSG (Static Site Generation)? Khi nào nên dùng ISR?
- **Trả lời:**
  - **SSR:** Mỗi khi có một HTTP request từ người dùng gửi tới, server mới bắt đầu lấy dữ liệu và tạo file HTML để trả về. Dữ liệu luôn mới nhất, nhưng tốn CPU server cho mỗi lượt truy cập.
  - **SSG:** Toàn bộ các file HTML được tạo sẵn một lần duy nhất tại thời điểm Build dự án (Deploy). Tốc độ tải trang siêu tốc vì được lưu trên mạng phân phối nội dung (CDN), nhưng không phù hợp cho dữ liệu thay đổi liên tục.
  - **ISR (Incremental Static Regeneration):** Cho phép tạo trang tĩnh như SSG nhưng có thêm cơ chế tự động tạo lại trang tĩnh ngầm ở nền sau một khoảng thời gian quy định (ví dụ `revalidate: 60s`). Rất lý tưởng cho các trang chi tiết sản phẩm Thương mại điện tử có hàng triệu mặt hàng.

#### Câu 8: Tại sao nên tách biệt Client State và Server State trong ứng dụng React? Ưu điểm của TanStack Query là gì?
- **Trả lời:**
  - **Client State:** Là dữ liệu giao diện tạm thời (trạng thái mở modal, theme sáng/tối) do máy khách toàn quyền sở hữu.
  - **Server State:** Là dữ liệu thuộc quyền sở hữu của máy chủ (danh sách bài viết, thông tin đơn hàng). Dữ liệu này có thể bị thay đổi bất kỳ lúc nào bởi người dùng khác và cần cơ chế đồng bộ, caching, refetching.
  - **Ưu điểm của TanStack Query:** Tự động hóa toàn bộ việc caching dữ liệu vào RAM, tự động gọi lại API khi người dùng quay lại tab trình duyệt (Focus Refetch), loại bỏ nhu cầu viết code `useState`/`useEffect` thủ công rườm rà.

#### Câu 9: Ba chỉ số cốt lõi trong Google Core Web Vitals (LCP, INP, CLS) đo lường điều gì và cách tối ưu chúng?
- **Trả lời:**
  - **LCP (Largest Contentful Paint - Chuẩn < 2.5s):** Đo thời gian khối nội dung lớn nhất (thường là ảnh banner hoặc tiêu đề chính) xuất hiện trên màn hình. Tối ưu: Dùng thẻ `next/image` nén ảnh WebP, ưu tiên tải trước bằng thuộc tính `priority`.
  - **INP (Interaction to Next Paint - Chuẩn < 200ms):** Đo độ trễ phản hồi thị giác khi người dùng thực hiện tương tác (bấm nút, mở menu). Tối ưu: Tránh chạy các vòng lặp tính toán nặng trên Main Thread của JavaScript, chia nhỏ tác vụ.
  - **CLS (Cumulative Layout Shift - Chuẩn < 0.1):** Đo mức độ giật/nhảy vị trí của các phần tử trên trang ngoài ý muốn khi tài nguyên vừa tải xong. Tối ưu: Luôn gán sẵn thuộc tính `width` và `height` cho ảnh và khung video để trình duyệt giữ chỗ trước.

#### Câu 10: Kỹ thuật Tree Shaking và Code Splitting trong quá trình đóng gói (Bundling) Frontend là gì?
- **Trả lời:**
  - **Tree Shaking:** Là quá trình loại bỏ các đoạn mã nguồn hoặc hàm thư viện "chết" (không bao giờ được gọi tới) ra khỏi gói bundle cuối cùng nhằm giảm dung lượng tải trang (dựa trên cú pháp ES Modules `import/export`).
  - **Code Splitting:** Chia nhỏ file bundle JavaScript khổng lồ thành nhiều mảnh nhỏ độc lập (chunks). Thay vì tải toàn bộ code của cả website ngay từ trang chủ, trình duyệt chỉ tải đúng đoạn mã cần thiết cho trang hiện tại và chỉ tải tiếp các trang khác khi người dùng chuyển hướng tới.

---

### CHUYÊN ĐỀ 3: CÂU HỎI PHỎNG VẤN MOBILE (5 CÂU)

#### Câu 11: So sánh ưu và nhược điểm giữa Native App (Kotlin/Swift) và Cross-Platform (React Native/Flutter)?
- **Trả lời:**
  - **Native App:**
    - *Ưu điểm:* Hiệu năng đạt tối đa 100%, tiếp cận các API phần cứng mới nhất (AR, AI chip, Bluetooth) ngay ngày đầu hệ điều hành ra mắt.
    - *Nhược điểm:* Chi phí nhân sự gấp đôi, thời gian phát triển lâu vì phải duy trì 2 codebase riêng biệt cho iOS và Android.
  - **Cross-Platform:**
    - *Ưu điểm:* Tiết kiệm 50% chi phí và thời gian, dùng chung một codebase duy nhất cho cả 2 nền tảng, cập nhật tính năng đồng thời.
    - *Nhược điểm:* Hiệu năng kém hơn Native một chút ở các tác vụ đồ họa 3D cực nặng, và cần chờ cộng đồng cập nhật thư viện khi Apple/Google tung ra các tính năng phần cứng mới.

#### Câu 12: Kiến trúc mới (New Architecture) trong React Native giải quyết vấn đề gì của kiến trúc Bridge cũ?
- **Trả lời:**
  - **Nhược điểm của Bridge cũ:** Mọi giao tiếp giữa mã JavaScript và Native đều phải đi qua một cầu nối (Bridge) chuyển đổi dữ liệu thành chuỗi JSON bất đồng bộ, gây hiện tượng nghẽn cổ chai (Bottleneck) làm ứng dụng bị giật khung hình khi người dùng cuộn nhanh danh sách dài.
  - **Giải pháp của New Architecture:** 
    - Dùng **JSI (JavaScript Interface)** cho phép JavaScript trực tiếp giữ tham chiếu C++ và gọi trực tiếp các phương thức Native đồng bộ mà không cần chuyển đổi JSON.
    - Sử dụng **Fabric Renderer** để render giao diện đa luồng mượt mà 120Hz.
    - Sử dụng **TurboModules** giúp các module phần cứng chỉ được nạp vào RAM khi thực sự được gọi (Lazy Loading).

#### Câu 13: Trình bày kiến trúc Offline-First trong ứng dụng di động?
- **Trả lời:**
  - Kiến trúc Offline-First coi kết nối Internet là một tùy chọn thêm chứ không phải điều kiện bắt buộc để ứng dụng hoạt động.
  - Giao diện ứng dụng (UI) **luôn luôn đọc và ghi dữ liệu trực tiếp vào Cơ sở dữ liệu cục bộ trên điện thoại** (như SQLite, WatermelonDB, MMKV). Nhờ đó, người dùng có thể mở app và thao tác tức thì kể cả khi đang ở chế độ máy bay.
  - Một tiến trình chạy nền (Background Worker) sẽ theo dõi trạng thái kết nối mạng. Khi có Internet, nó sẽ tự động đồng bộ các bản ghi mới tạo từ CSDL cục bộ lên Server và tải dữ liệu mới từ Server về máy.

#### Câu 14: Vòng đời của một ứng dụng Mobile gồm những trạng thái cơ bản nào? Tại sao cần lưu trạng thái khi app rơi vào Background?
- **Trả lời:**
  - Các trạng thái cơ bản:
    1. **Active / Foreground:** Ứng dụng đang hiển thị trực tiếp trên màn hình và nhận tương tác từ người dùng.
    2. **Inactive / Paused:** Ứng dụng bị che khuất tạm thời (ví dụ có cuộc gọi điện thoại tới hoặc người dùng vuốt thanh thông báo).
    3. **Background:** Ứng dụng đã bị thu nhỏ xuống nền khi người dùng bấm nút Home hoặc chuyển sang app khác.
    4. **Suspended / Terminated:** Hệ điều hành đóng băng hoặc giải phóng hoàn toàn bộ nhớ RAM của app.
  - **Lý do cần lưu trạng thái:** Khi điện thoại bị thiếu RAM, hệ điều hành Android/iOS có thể **tiêu diệt tiến trình ứng dụng ngầm bất kỳ lúc nào mà không báo trước**. Lập trình viên phải lắng nghe sự kiện app chuyển sang Background để kịp thời lưu lại nội dung form đang gõ dở vào bộ nhớ cục bộ, tránh làm mất dữ liệu của người dùng khi họ mở lại app.

#### Câu 15: Quy trình Ký mã xác thực (Code Signing) khi xuất bản ứng dụng lên Google Play và Apple App Store gồm những gì?
- **Trả lời:**
  - **Mục đích:** Đảm bảo tính toàn vẹn của ứng dụng, chứng minh bản build do chính nhà phát triển hợp pháp tạo ra và không bị kẻ xấu can thiệp chèn mã độc trên đường truyền.
  - **Trên Android (Google Play):** Tạo một tệp khóa bảo mật **Keystore** bằng công cụ `keytool` chứa Private Key. Bản build đóng gói dạng `.aab` (Android App Bundle) sẽ được ký bằng chữ ký số này. Sau đó Google Play App Signing sẽ quản lý khóa phát hành cuối cùng.
  - **Trên iOS (Apple App Store):** Cực kỳ nghiêm ngặt, đòi hỏi:
    1. **Certificate (Chứng chỉ nhà phát triển):** Tạo từ máy Mac qua Apple Developer Portal.
    2. **App ID:** Định danh gói ứng dụng duy nhất toàn cầu.
    3. **Provisioning Profile:** File liên kết giữa Chứng chỉ lập trình viên, App ID và danh sách các quyền hạn được cấp phép (Push Notification, In-App Purchase). Bản build `.ipa` phải được ký hợp lệ qua Xcode trước khi nộp lên App Store Connect.

---

## PHẦN 3: CHECKLIST TỰ ĐÁNH GIÁ NĂNG LỰC (LEVEL 4 COMPETENCY)

Hãy tự đánh giá mức độ chuyên môn của bạn theo nhánh đã chọn:

### Dành cho Kỹ sư Backend:
- [ ] Tôi hiểu rõ nguyên lý Dependency Injection (DI) và áp dụng được trong NestJS hoặc Spring Boot.
- [ ] Tôi biết cách triển khai Cache-Aside Pattern với Redis để tối ưu hóa truy vấn CSDL.
- [ ] Tôi biết cách tách tác vụ nặng ra xử lý nền bằng Message Queue (BullMQ, RabbitMQ hoặc Celery).
- [ ] Tôi hiểu sự khác biệt giữa REST, GraphQL và gRPC.
- [ ] Tôi biết cách xử lý bài toán Race Condition khi có nhiều giao dịch đồng thời.

### Dành cho Kỹ sư Frontend:
- [ ] Tôi phân biệt được 4 cơ chế: CSR, SSR, SSG, và ISR trong Next.js.
- [ ] Tôi hiểu rõ kiến trúc React Server Components (RSC) và biết khi nào nên dùng `'use client'`.
- [ ] Tôi biết cách quản lý Server State bằng TanStack Query thay vì lạm dụng Redux.
- [ ] Tôi hiểu và biết cách tối ưu 3 chỉ số Google Core Web Vitals (LCP, INP, CLS).
- [ ] Tôi tự tin xây dựng giao diện hiện đại với TailwindCSS và Component System.

### Dành cho Kỹ sư Mobile:
- [ ] Tôi phân biệt được ưu nhược điểm giữa lập trình Native và Cross-Platform (React Native/Flutter).
- [ ] Tôi hiểu kiến trúc New Architecture của React Native (JSI, Fabric, TurboModules).
- [ ] Tôi biết cách xây dựng ứng dụng theo mô hình Offline-First với CSDL cục bộ.
- [ ] Tôi hiểu cơ chế nhận Push Notification qua cầu nối FCM và APNs.
- [ ] Tôi nắm vững quy trình Code Signing và đóng gói phát hành app lên App Store / Google Play.
