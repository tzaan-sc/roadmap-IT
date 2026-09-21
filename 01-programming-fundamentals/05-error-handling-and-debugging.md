# 05 - Quản Lý Lỗi, Xử Lý Ngoại Lệ & Kỹ Năng Debugging (Error Handling & Debugging)

> **Mục tiêu bài học:** Phân biệt các loại lỗi phần mềm, nắm vững cơ chế bắt ngoại lệ (`try-catch-finally`), tự tạo Exception nghiệp vụ và nâng cấp trình độ từ việc gõ `console.log()` bừa bãi sang kỹ năng Debugging chuyên nghiệp với Breakpoints, Call Stack và hệ thống Logging chuẩn công nghiệp.

---

## 1. Phân Loại 3 Dạng Lỗi Kinh Điển Trong Lập Trình

Mọi lỗi mà bạn gặp phải trong suốt sự nghiệp lập trình đều rơi vào 1 trong 3 nhóm sau:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                           CÁC LOẠI LỖI PHẦN MỀM                         │
└───────┬─────────────────────────┬─────────────────────────────┬─────────┘
        │                         │                             │
        ▼                         ▼                             ▼
  1. SYNTAX ERROR           2. RUNTIME ERROR              3. LOGIC ERROR
  (Lỗi cú pháp)             (Lỗi thực thi / Exception)    (Lỗi nghiệp vụ / Bug)
  - Thiếu ngoặc, sai chính  - Chia cho số 0, gọi hàm      - Code chạy êm ru
    tả từ khóa.               trên null/undefined.          nhưng kết quả sai.
  - Trình biên dịch bắt     - Làm sập (crash) ứng         - Đáng sợ nhất vì máy
    được ngay lập tức.        dụng lúc người dùng dùng.     không hề báo lỗi!
```

1. **Lỗi cú pháp (Syntax / Compile-time Error):**
   - Viết sai ngữ pháp của ngôn ngữ. IDE hoặc trình biên dịch sẽ gạch chân đỏ ngay lúc gõ phím.
2. **Lỗi lúc thực thi (Runtime Error / Exception):**
   - Cú pháp hoàn toàn đúng, nhưng khi chạy thực tế thì phát sinh tình huống không hợp lệ (ví dụ: ổ cứng bị đầy không ghi được file, rớt mạng khi gọi API, truy cập thuộc tính của `undefined`).
3. **Lỗi logic (Logic Bug):**
   - Bạn muốn tính thuế 10% nhưng lại gõ nhầm công thức thành nhân với 0.01. Chương trình vẫn chạy mượt mà, nhưng khách hàng bị tính thiếu tiền! Nhóm lỗi này chỉ phát hiện được bằng **Viết Unit Test** hoặc **Thao tác Debugging**.

---

## 2. Cơ Chế Xử Lý Ngoại Lệ (Exception Handling)

Khi một sự cố xảy ra lúc runtime, nếu bạn không có cơ chế "hứng", chương trình sẽ **sập ngay lập tức (Crash)**. Cơ chế `try-catch` giúp bắt lấy ngoại lệ và xử lý nó một cách an toàn mà không làm sập toàn bộ hệ thống.

```text
       ┌────────────────────────────────────────────────────────┐
       │ try {                                                  │
       │   // Chạy đoạn code có nguy cơ gây lỗi ở đây           │
       │   // (ví dụ: đọc file, gọi API mạng, query database)   │
       │ }                                                      │
       └───────────────────────────┬────────────────────────────┘
                                   │ Có lỗi phát sinh!
                                   ▼
       ┌────────────────────────────────────────────────────────┐
       │ catch (error) {                                        │
       │   // Hứng lấy lỗi, ghi log cảnh báo, hiện thông báo     │
       │   // thân thiện cho người dùng thay vì làm sập app!    │
       │ }                                                      │
       └───────────────────────────┬────────────────────────────┘
                                   │ Luôn luôn chạy qua
                                   ▼
       ┌────────────────────────────────────────────────────────┐
       │ finally {                                              │
       │   // BẮT BUỘC CHẠY (dù có lỗi hay không)!              │
       │   // Chuyên dùng để dọn dẹp: Đóng kết nối DB, đóng file│
       │ }                                                      │
       └────────────────────────────────────────────────────────┘
```

### 2.1. Code mẫu trong JavaScript & Python

```javascript
// JavaScript
function readUserConfigFile(filePath) {
  let fileHandle = null;
  try {
    fileHandle = openFile(filePath);
    const data = fileHandle.read();
    return JSON.parse(data);
  } catch (error) {
    console.error(`Không thể đọc cấu hình: ${error.message}`);
    return { theme: "light" }; // Giá trị dự phòng (Fallback) an toàn
  } finally {
    // Dù thành công hay ném ra lỗi, fileHandle vẫn được đóng sạch sẽ
    if (fileHandle) fileHandle.close();
  }
}
```

```python
# Python
def divide_numbers(a, b):
    try:
        result = a / b
    except ZeroDivisionError as e:
        print(f"Lỗi toán học: Không thể chia cho số 0! Chi tiết: {e}")
        return None
    except TypeError as e:
        print(f"Lỗi kiểu dữ liệu: Đầu vào phải là số thực! Chi tiết: {e}")
        return None
    else:
        # Chạy khi KHÔNG có bất kỳ ngoại lệ nào xảy ra
        print("Phép tính thành công!")
        return result
    finally:
        print("Hoàn tất phiên tính toán.")
```

### 2.2. Chủ động ném ngoại lệ (`throw` / `raise`)
Khi dữ liệu đầu vào vi phạm nghiêm trọng quy tắc logic của hàm, hãy chủ động ném ra lỗi:

```javascript
function registerUser(age) {
  if (typeof age !== "number") {
    throw new TypeError("Tuổi phải là một con số!");
  }
  if (age < 18) {
    throw new RangeError("Người dùng phải từ đủ 18 tuổi trở lên!");
  }
  // Tiến hành đăng ký...
}
```

---

## 3. Tạo Ngoại Lệ Nghiệp Vụ Tùy Chỉnh (Custom Exceptions)

Thay vì bắt các lỗi chung chung, trong các dự án thực tế, bạn nên định nghĩa các lớp Exception riêng biệt cho từng miền nghiệp vụ:

```python
# Tạo lớp lỗi nghiệp vụ kế thừa từ Exception gốc
class InsufficientFundsError(Exception):
    def __init__(self, current_balance, amount_requested):
        super().__init__(
            f"Giao dịch bị từ chối! Số dư hiện tại ({current_balance}$) "
            f"không đủ để rút {amount_requested}$."
        )
        self.current_balance = current_balance
        self.amount_requested = amount_requested

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
        
    def withdraw(self, amount):
        if amount > self.balance:
            # Chủ động ném ra Custom Exception rõ ràng
            raise InsufficientFundsError(self.balance, amount)
        self.balance -= amount
```

---

## 4. Nghệ Thuật Debugging Chuyên Nghiệp

### 4.1. Tại sao lạm dụng `print()` hay `console.log()` lại là thói quen xấu?
- **Làm bẩn mã nguồn:** Bạn rất dễ quên xóa chúng trước khi đẩy code lên môi trường thật (Production).
- **Làm chậm hiệu năng:** Các thao tác in ra màn hình Terminal (I/O) tốn rất nhiều chu kỳ CPU.
- **Tùy biến kém:** Bạn không thể tua ngược thời gian, không kiểm tra được giá trị của 50 biến khác trong cùng một hàm.

### 4.2. Kỹ thuật Đọc Hiểu Stack Trace (Dấu Vết Cuộc Gọi Hàm)
Khi một ngoại lệ không được xử lý ném ra màn hình, nó in ra một **Stack Trace**:

```text
Traceback (most recent call last):
  File "main.py", line 25, in <module>
    checkout_cart(my_cart)
  File "main.py", line 18, in checkout_cart
    charge_credit_card(cart.total)
  File "payment.py", line 42, in charge_credit_card
    response = api_client.post("/charge", amount=amount)
  File "api.py", line 10, in post
    raise ConnectionTimeoutError("Cổng thanh toán không phản hồi sau 30s")
ConnectionTimeoutError: Cổng thanh toán không phản hồi sau 30s
```

> [!TIP]
> **Quy tắc đọc Stack Trace:**
> - Nhìn ngay vào **Dòng cuối cùng**: Cho biết **Tên lỗi** và **Lý do lỗi**.
> - Đọc ngược từ dưới lên trên: Tìm dòng code đầu tiên thuộc về file của bạn (ví dụ dòng 42 trong `payment.py`). Đó chính là nơi phát sinh thảm họa!

### 4.3. Làm chủ Debugger với Breakpoints (Trong VS Code / Chrome DevTools)

```text
    Dòng code:                              Hành động gỡ lỗi:
12: let tax = calculateTax(total);  ───►  ● ĐẶT BREAKPOINT TẠI ĐÂY!
13: let finalAmount = total + tax;        Chương trình sẽ DỪNG ĐỨNG HÌNH tại dòng 12.
14: processPayment(finalAmount);          Bạn có thể soi rọi toàn bộ RAM, biến số!
```

- **Breakpoint (Điểm dừng):** Đánh dấu một dòng code. Khi chạy ở chế độ Debug (`F5`), chương trình sẽ tạm dừng ngay trước khi thực thi dòng này.
- **Step Over (`F10`):** Thực thi dòng hiện tại và nhảy sang dòng kế tiếp (không nhảy vào bên trong ruột hàm con).
- **Step Into (`F11`):** Đi sâu vào bên trong thân của hàm con đang được gọi để xem từng dòng lệnh bên trong.
- **Step Out (`Shift + F11`):** Chạy nốt toàn bộ phần còn lại của hàm con hiện tại và quay trở lại hàm gọi bên ngoài.
- **Variables Window:** Xem trực tiếp giá trị của mọi biến Local và Global lúc runtime.
- **Watch Window:** Nhập các biểu thức phức tạp (ví dụ: `user.orders.length > 5`) để xem giá trị của nó thay đổi ra sao qua từng bước nhảy code.

---

## 5. Ghi Nhật Ký (Logging) Chuẩn Công Nghiệp Thay Vì Print

Trong hệ thống Backend hay Cloud thật sự, ứng dụng chạy ngầm không có màn hình Terminal để bạn nhìn. Mọi hoạt động phải được ghi vào file log theo các cấp độ chuẩn (**Log Levels**):

```text
  DEBUG ──► INFO ──► WARNING ──► ERROR ──► CRITICAL / FATAL
(Chi tiết nhất)                                (Sập toàn bộ hệ thống)
```

1. **`DEBUG`:** Thông tin chi tiết tỉ mỉ phục vụ lập trình viên gỡ lỗi khi phát triển (ví dụ: giá trị payload JSON nhận về).
2. **`INFO`:** Thông tin xác nhận sự kiện bình thường diễn ra thành công (ví dụ: *"Người dùng User_123 đăng nhập thành công lúc 08:30"*).
3. **`WARNING`:** Cảnh báo điều bất thường nhưng hệ thống vẫn tự xoay xở chạy tiếp được (ví dụ: *"Dung lượng ổ đĩa chỉ còn dưới 10%"*).
4. **`ERROR`:** Lỗi nghiêm trọng khiến một chức năng cụ thể bị thất bại (ví dụ: *"Không thể gửi email kích hoạt cho khách hàng"*).
5. **`CRITICAL`:** Lỗi thảm họa khiến toàn bộ ứng dụng bị tê liệt (ví dụ: *"Database chính bị mất kết nối, server ngừng hoạt động"*).

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. Phân biệt rõ **Syntax Error** (trình biên dịch bắt), **Runtime Exception** (xảy ra lúc chạy), và **Logic Bug** (kết quả sai lệch).
2. Luôn dùng `try-catch-finally` cho các tác vụ tiềm ẩn rủi ro (I/O file, kết nối mạng, parsing JSON).
3. Khối `finally` là nơi chuẩn mực nhất để dọn dẹp tài nguyên và đóng kết nối.
4. Đọc **Stack Trace từ dưới lên** để định vị chính xác vị trí dòng code gây lỗi.
5. Học cách đặt **Breakpoints**, sử dụng **Step Over (F10)** và **Step Into (F11)** trong VS Code để biến việc gỡ lỗi thành một trải nghiệm khoa học và trực quan.
