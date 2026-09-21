# 03 - Chuyên Sâu Kỹ Sư Mobile (Mobile Engineering Specialization)

> **Mục tiêu bài học:** Nắm vững toàn cảnh bức tranh phát triển ứng dụng di động, phân biệt sâu sắc giữa lập trình Native (Kotlin / Swift) và Đa nền tảng (React Native / Flutter), làm chủ kiến trúc Offline-First, cơ chế Push Notifications và quy trình ký mã xuất bản ứng dụng lên Apple App Store & Google Play.

---

## 1. Sự Khác Biệt Sống Còn Giữa Lập Trình Web Và Mobile

Nhiều lập trình viên Web khi chuyển sang Mobile thường bị "ngợp" vì môi trường trên điện thoại thông minh hoàn toàn khác biệt:

```text
       MÔI TRƯỜNG WEB                                MÔI TRƯỜNG MOBILE
┌─────────────────────────────┐               ┌──────────────────────────────┐
│ - Nguồn điện cắm liên tục.  │               │ - Pin hữu hạn (Tối ưu pin!). │
│ - RAM máy tính dồi dào.     │               │ - RAM eo hẹp (OS tự giết app)│
│ - Mạng dây/Wifi ổn định.    │               │ - Mạng 4G/5G chập chờn liên  │
│ - Deploy code là người dùng │               │   tục khi di chuyển xe/thang │
│   thấy ngay lập tức!        │               │   máy => Bắt buộc Offline!   │
│                             │               │ - Phải qua xét duyệt Store   │
│                             │               │   (Apple duyệt từ 1-3 ngày!) │
└─────────────────────────────┘               └──────────────────────────────┘
```

---

## 2. Cuộc Chiến Nền Tảng: Native vs Cross-Platform

```text
                          CÁC HƯỚNG ĐI CỦA KỸ SƯ MOBILE
┌──────────────────────────────────────────────┬──────────────────────────────────────────────┐
│             LẬP TRÌNH NATIVE                 │         LẬP TRÌNH ĐA NỀN TẢNG (CROSS)        │
├──────────────────────┬───────────────────────┼──────────────────────┬───────────────────────┤
│    ANDROID NATIVE    │      iOS NATIVE       │     REACT NATIVE     │        FLUTTER        │
│    Ngôn ngữ: Kotlin  │    Ngôn ngữ: Swift    │  Ngôn ngữ: TypeScript│    Ngôn ngữ: Dart     │
│  Framework: Compose  │  Framework: SwiftUI   │  Nền tảng: Meta (FB) │  Nền tảng: Google     │
└──────────────────────┴───────────────────────┴──────────────────────┴───────────────────────┘
```

### 2.1. So sánh chi tiết các công nghệ:

| Tiêu chí | Native (Kotlin / Swift) | React Native (TS) | Flutter (Dart) |
| :--- | :--- | :--- | :--- |
| **Hiệu năng** | **Tối đa 100%**, tận dụng tối đa GPU/NPU | Rất cao (với New Architecture) | Cực cao (Vẽ trực tiếp bằng Impeller) |
| **Tái sử dụng code** | 0% (Phải viết 2 dự án riêng biệt) | **~85 - 95%** dùng chung giữa iOS & Android | **~90 - 98%** dùng chung giữa iOS & Android |
| **Tốc độ làm sản phẩm** | Chậm (Tốn gấp đôi nhân sự) | **Siêu nhanh** (Tận dụng kiến thức React Web) | **Rất nhanh** (Hot Reload siêu tốc) |
| **Giao diện (UI)** | Mang đậm ngôn ngữ gốc của hệ điều hành | Sử dụng các thành phần Native UI gốc | Tự vẽ toàn bộ pixel lên màn hình (Canvas) |
| **Khi nào nên chọn?** | Ứng dụng Game nặng 3D, AR/VR, xử lý camera/audio chuyên sâu, ngân hàng lớn | Đội ngũ đã biết React/TypeScript, ứng dụng thương mại, mạng xã hội, SaaS | Startup cần giao diện lung linh đồng nhất 100% trên mọi loại màn hình |

---

## 3. Kiến Trúc Bên Trong: React Native vs Flutter

### 3.1. React Native New Architecture (Kiến Trúc Mới)
Trước đây, React Native dùng một "Cầu nối" (Bridge) để tuần tự hóa dữ liệu JSON giữa JavaScript và Native C++, gây giật lag khi cuộn nhanh. 
Hiện nay, **New Architecture** đã loại bỏ hoàn toàn Bridge cũ:
- **JSI (JavaScript Interface):** Cho phép mã JavaScript gọi trực tiếp các phương thức C++ và Native mà không cần ép kiểu JSON.
- **Fabric Renderer:** Hệ thống render UI mới đa luồng, hỗ trợ đồng bộ mượt mà 120Hz.
- **TurboModules:** Chỉ tải các module phần cứng (Camera, GPS) khi thực sự cần dùng (Lazy loading), giúp app khởi động nhanh hơn 40%.

### 3.2. Flutter Engine
Flutter không dùng bất kỳ thành phần UI gốc nào của iOS hay Android. Flutter hoạt động giống như một **Game Engine**:
- Toàn bộ giao diện được vẽ trực tiếp từng pixel bằng **Engine đồ họa Impeller (hoặc Skia)** thông qua Metal (trên iOS) hoặc Vulkan (trên Android).
- Mọi thành phần trên màn hình đều là **Widget** (`StatelessWidget` và `StatefulWidget`).

---

## 4. Các Bài Toán Sống Còn Của Lập Trình Viên Mobile

### 4.1. Chiến Lược Dữ Liệu Ngoại Tuyến (Offline-First Architecture)
Người dùng di động thường xuyên rơi vào tình trạng mất mạng (đi vào hầm gửi xe, thang máy, máy bay). Ứng dụng chuyên nghiệp **không bao giờ được hiện màn hình lỗi**:

```text
[ GIAO DIỆN APP (UI) ] ◄── Luôn luôn đọc và hiển thị dữ liệu từ CSDL CỤC BỘ!
          │
          ▼ (Khi người dùng tạo mới dữ liệu)
[ CƠ SỞ DỮ LIỆU CỤC BỘ TRÊN MÁY ] (SQLite / WatermelonDB / MMKV)
          │
          ▼ (Chạy ngầm ở nền - Background Sync)
   Kiểm tra có kết nối Internet không?
        /              \
     (CÓ MẠNG)       (MẤT MẠNG)
        │                 │
  Đẩy dữ liệu lên Server   Đợi khi nào có mạng thì tự động đồng bộ tiếp!
```

### 4.2. Cơ Chế Thông Báo Đẩy (Push Notifications)
Làm sao để khi có tin nhắn mới hoặc có đơn hàng, điện thoại của người dùng nhận được thông báo rung và kêu chuông kể cả khi ứng dụng đã bị tắt hoàn toàn?

```text
[ SERVER CỦA BẠN ] ── 1. Có tin nhắn mới ──► [ DỊCH VỤ THÔNG BÁO HỆ ĐIỀU HÀNH ]
                                               - FCM (Firebase Cloud Messaging - Android)
                                               - APNs (Apple Push Notification - iOS)
                                                         │
                                                         ▼ 2. Gửi tín hiệu đánh thức
                                               [ THIẾT BỊ ĐIỆN THOẠI ]
                                               (Hệ điều hành nhận sóng, hiện popup)
```

---

## 5. Quy Trình Xuất Bản Ứng Dụng Lên App Store & Google Play

Xuất bản một ứng dụng di động phức tạp hơn việc deploy web rất nhiều:

```text
[ MÃ NGUỒN HOÀN THIỆN ]
           │
           ▼
[ 1. KÝ MÃ XÁC THỰC (CODE SIGNING) ]
- Android: Tạo file Keystore bảo mật (chữ ký số)
- iOS: Tạo Certificate & Provisioning Profile trên Apple Developer Portal
           │
           ▼
[ 2. ĐÓNG GÓI BẢN BUILD (RELEASE BUILD) ]
- Android: Tạo file .aab (Android App Bundle)
- iOS: Build file .ipa qua Xcode trên máy tính Mac
           │
           ▼
[ 3. XÉT DUYỆT CỬA HÀNG (STORE REVIEW) ]
- Google Play Console: Kiểm duyệt tự động + thủ công (khoảng 1 - 2 ngày)
- Apple App Store: Kiểm duyệt nhân sự cực kỳ khắt khe về quyền riêng tư,
  hướng dẫn giao diện (HIG) và tài khoản thanh toán in-app (1 - 3 ngày)
           │
           ▼
[ CHÍNH THỨC PHÁT HÀNH TỚI NGƯỜI DÙNG TOÀN CẦU! 🚀 ]
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. Lập trình Mobile đòi hỏi tư duy tối ưu hóa pin, bộ nhớ RAM và thiết kế **Offline-First**.
2. **Native (Kotlin/Swift)** mang lại hiệu năng cao nhất; **Cross-Platform (React Native/Flutter)** giúp tiết kiệm 50% chi phí và thời gian phát triển.
3. Nắm vững **Vòng đời ứng dụng (App Lifecycle)** để lưu trạng thái trước khi bị hệ điều hành giải phóng bộ nhớ.
4. Triển khai **Push Notification** qua cầu nối **FCM (Android)** và **APNs (iOS)**.
5. Luôn chuẩn bị kỹ lưỡng quy trình **Ký mã số (Code Signing)** và đọc kỹ chính sách quyền riêng tư trước khi nộp ứng dụng lên App Store.
