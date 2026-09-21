# 01 - Cú Pháp Cốt Lõi & Tư Duy Lập Trình Cơ Bản (Syntax & Core Concepts)

> **Mục tiêu bài học:** Làm chủ các khối xây dựng nền tảng của mọi ngôn ngữ lập trình: biến số, kiểu dữ liệu nguyên thủy & tham chiếu, toán tử, cấu trúc điều khiển rẽ nhánh, vòng lặp, cùng với tư duy thiết kế hàm (Functions) và phạm vi biến (Scope).

---

## 1. Biến Số (Variables) & Hằng Số (Constants)

Biến số là **tên định danh đại diện cho một ô nhớ trong RAM** dùng để lưu trữ dữ liệu có thể thay đổi trong quá trình chương trình chạy.

```text
       Tên biến (Identifier): "age"
                 │
                 ▼
       ┌──────────────────┐
       │   Giá trị: 25    │  ◄── Nằm tại địa chỉ ô nhớ RAM: 0x7FFEEA04
       └──────────────────┘
```

### 1.1. Các giai đoạn của một biến
1. **Declaration (Khai báo):** Thông báo cho trình biên dịch/thông dịch biết sự tồn tại của biến.
2. **Initialization (Khởi tạo):** Gán giá trị lần đầu tiên cho biến khi vừa tạo.
3. **Assignment (Gán lại):** Cập nhật giá trị mới cho biến đã tồn tại.

```javascript
// JavaScript
let score;        // Declaration (chưa có giá trị -> undefined)
score = 100;      // Initialization
score = 150;      // Re-assignment

const PI = 3.14;  // Constant (Hằng số - không thể gán lại giá trị)
```

```python
# Python (Tự động gán kiểu lúc runtime - Dynamic Typing)
score = 100       # Vừa khai báo vừa khởi tạo
score = 150       # Gán lại giá trị mới
PI = 3.14         # Quy ước viết hoa là Hằng số (Constants)
```

### 1.2. Static Typing (Định kiểu tĩnh) vs Dynamic Typing (Định kiểu động)

| Tiêu chí | Static Typing (C, C++, Java, Rust, TypeScript) | Dynamic Typing (Python, JavaScript, Ruby, PHP) |
| :--- | :--- | :--- |
| **Kiểm tra kiểu** | Tại thời điểm biên dịch (Compile-time) | Tại thời điểm thực thi (Runtime) |
| **Cú pháp khai báo** | `int age = 20;` hoặc `let age: number = 20;` | `age = 20` hoặc `let age = 20;` |
| **Ưu điểm** | Bắt lỗi kiểu dữ liệu ngay khi viết code; IDE gợi ý code cực tốt; chạy nhanh | Viết code nhanh, linh hoạt, ngắn gọn |
| **Nhược điểm** | Viết code dài hơn; phải học cú pháp kiểu chặt chẽ | Dễ phát sinh lỗi ngầm lúc chạy (Runtime TypeError) |

---

## 2. Kiểu Dữ Liệu: Primitive (Nguyên Thủy) vs Reference (Tham Chiếu)

Hiểu được sự phân chia này sẽ giúp bạn tránh 80% các lỗi "bỗng dưng dữ liệu bị thay đổi" ngoài ý muốn.

```text
PRIMITIVE TYPES (Lưu trực tiếp giá trị vào STACK)
Biến a: [ 10 ]
Biến b = a ──► Tạo bản sao độc lập: [ 10 ] (Sửa b không ảnh hưởng tới a)

REFERENCE TYPES (Lưu con trỏ ở STACK, dữ liệu thật nằm ở HEAP)
Biến obj1: [ Con trỏ 0x1A2B ] ──────┐
                                   ▼
Biến obj2 = obj1                   [ Dữ liệu Object thật trên HEAP ]
           [ Con trỏ 0x1A2B ] ──────┘ (Cả 2 cùng trỏ chung một địa chỉ!)
```

### 2.1. Kiểu nguyên thủy (Primitive Types / Value Types)
- Đại diện cho các giá trị đơn lẻ, kích thước cố định:
  - Số nguyên, số thực: `number` (JS), `int`, `float` (Python, C++).
  - Chuỗi ký tự: `string` (hoặc `str`).
  - Giá trị đúng/sai: `boolean` (`true` / `false`).
  - Không có giá trị: `null`, `undefined` (JS), `None` (Python).
- **Tính chất:** Truyền theo giá trị (**Pass by Value**). Khi gán biến này cho biến khác, hệ thống sao chép một bản copy riêng biệt.

### 2.2. Kiểu tham chiếu (Reference Types)
- Đại diện cho các cấu trúc dữ liệu phức tạp:
  - Mảng (Array / List), Đối tượng (Object / Dictionary), Hàm (Function).
- **Tính chất:** Biến chỉ lưu **địa chỉ con trỏ**. Khi gán `b = a`, cả hai biến đều cùng trỏ vào cùng một vùng nhớ trên Heap. Nếu sửa thuộc tính qua `b`, dữ liệu của `a` cũng bị thay đổi!

```javascript
// Minh họa Reference Type trong JavaScript
let list1 = [1, 2, 3];
let list2 = list1; // Cả hai trỏ chung vùng nhớ

list2.push(99);
console.log(list1); // In ra: [1, 2, 3, 99] -> list1 bị thay đổi theo!
```

---

## 3. Toán Tử & Đánh Giá Ngắn Mạch (Short-Circuit Evaluation)

### 3.1. Phép so sánh lỏng lẻo vs So sánh nghiêm ngặt (Strict Equality)
Trong JavaScript:
- `==` (Loose equality): Tự động ép kiểu (Type coercion) rồi mới so sánh. Gây ra các hiện tượng khó lường:
  ```javascript
  "5" == 5;      // true (ép chuỗi "5" thành số 5)
  0 == false;    // true
  null == undefined; // true
  ```
- `===` (Strict equality): So sánh cả giá trị **lẫn kiểu dữ liệu** mà không ép kiểu. **Luôn luôn sử dụng `===` trong JavaScript**:
  ```javascript
  "5" === 5;     // false (khác kiểu: string vs number)
  ```
*(Trong Python, toán tử `==` luôn so sánh giá trị tương đương, và `is` dùng để so sánh hai biến có cùng trỏ vào một ô nhớ hay không).*

### 3.2. Đánh giá ngắn mạch (Short-Circuit Evaluation with AND / OR)
- **`A && B` (AND):** Nếu `A` là **falsy**, trả về ngay `A` mà không thèm tính tiếp `B`.
- **`A || B` (OR):** Nếu `A` là **truthy**, trả về ngay `A` mà không thèm tính tiếp `B`.

```javascript
// Ứng dụng gán giá trị mặc định kinh điển:
let username = inputName || "Khách vô danh";

// Ứng dụng gọi hàm an toàn khi biến tồn tại:
user && user.sendNotification();
```

---

## 4. Cấu Trúc Điều Khiển Luồng (Control Flow)

### 4.1. Câu lệnh rẽ nhánh `if / else if / else`
Dùng khi cần đưa ra quyết định dựa trên điều kiện logic:

```python
score = 85

if score >= 90:
    grade = "Xuất sắc"
elif score >= 75:
    grade = "Giỏi"
elif score >= 50:
    grade = "Trung bình"
else:
    grade = "Yếu"
```

### 4.2. Khớp mẫu & Lựa chọn nhiều nhánh (`switch/case` hoặc `match/case`)
Khi có quá nhiều nhánh so sánh một biến với các giá trị hằng cụ thể:

```javascript
const status = "PENDING";

switch (status) {
  case "SUCCESS":
    console.log("Giao dịch thành công");
    break; // BẮT BUỘC có break để tránh rơi xuống case dưới (fall-through)
  case "PENDING":
    console.log("Đang chờ xử lý...");
    break;
  case "FAILED":
    console.log("Thất bại!");
    break;
  default:
    console.log("Trạng thái không xác định");
}
```

---

## 5. Vòng Lặp (Loops): Lặp Lại Công Việc Tự Động

### 5.1. Vòng lặp `for` (Biết trước số lần lặp)
```javascript
// Duyệt từ 0 đến 4
for (let i = 0; i < 5; i++) {
  console.log(`Lần lặp thứ: ${i}`);
}
```

```python
# Trong Python dùng hàm range()
for i in range(5):
    print(f"Lần lặp thứ: {i}")
```

### 5.2. Vòng lặp `while` (Lặp theo điều kiện logic chưa biết trước số lần)
```python
password = ""
# Tiếp tục hỏi cho đến khi người dùng nhập đúng
while password != "secret123":
    password = input("Nhập mật mã: ")
```

### 5.3. Kiểm soát vòng lặp: `break` vs `continue`
- **`break`:** Lập tức thoát hoàn toàn ra khỏi vòng lặp gần nhất.
- **`continue`:** Bỏ qua các câu lệnh còn lại của lần lặp hiện tại, nhảy ngay sang lần lặp kế tiếp.

```javascript
for (let i = 1; i <= 10; i++) {
  if (i % 2 === 0) continue; // Bỏ qua số chẵn
  if (i > 7) break;          // Dừng hẳn khi vượt quá 7
  console.log(i);            // Chỉ in ra: 1, 3, 5, 7
}
```

---

## 6. Hàm (Functions) - Trái Tim Của Kiến Trúc Code

Hàm là một khối mã lệnh có thể tái sử dụng, nhận đầu vào (Inputs), xử lý và trả về đầu ra (Output).

```text
  ĐẦU VÀO (Parameters)       ┌────────────────────────┐       ĐẦU RA (Return Value)
  a = 5, b = 10        ───►  │  HÀM: add(a, b)        │ ───►  15
                             │  return a + b;         │
                             └────────────────────────┘
```

### 6.1. Tham số (Parameters) vs Đối số (Arguments)
- **Parameter (Tham số):** Tên biến được liệt kê trong định nghĩa hàm (ví dụ: `a`, `b`).
- **Argument (Đối số):** Giá trị thực tế được truyền vào khi gọi hàm (ví dụ: `5`, `10`).

### 6.2. Giá trị mặc định (Default Parameters) & Rest Parameter
```javascript
// Default parameter: taxRate mặc định là 0.1 nếu không truyền
function calculateTotal(price, taxRate = 0.1) {
  return price + (price * taxRate);
}

// Rest parameter (...numbers): gom vô số đối số thành một mảng
function sumAll(...numbers) {
  return numbers.reduce((acc, curr) => acc + curr, 0);
}
console.log(sumAll(1, 2, 3, 4, 5)); // 15
```

### 6.3. Pure Functions (Hàm Thuần Khiết) - Tư Duy Viết Code Đẳng Cấp
Một hàm được coi là Pure Function khi thỏa mãn 2 điều kiện:
1. **Tính tất định (Deterministic):** Cùng một đầu vào (Inputs) luôn luôn trả về cùng một đầu ra (Output), không phụ thuộc vào thời gian, random hay biến toàn cục.
2. **Không có hiệu ứng phụ (No Side Effects):** Không tự ý sửa đổi biến toàn cục bên ngoài, không ghi file, không làm thay đổi các tham số truyền vào dạng tham chiếu.

```javascript
// ❌ Impure Function (Phụ thuộc và làm biến đổi trạng thái bên ngoài)
let total = 0;
function addToTotal(amount) {
  total += amount; // Side effect!
  return total;
}

// ✅ Pure Function (Dễ test, độc lập, không bug ngầm)
function calculateNewTotal(currentTotal, amount) {
  return currentTotal + amount;
}
```

---

## 7. Phạm Vi Biến (Scope) & Bao Đóng (Closures)

### 7.1. Các tầng Scope
- **Global Scope (Toàn cục):** Biến được khai báo ngoài cùng, mọi nơi trong file đều truy cập được.
- **Function Scope (Hàm):** Biến khai báo trong hàm chỉ sống bên trong hàm đó.
- **Block Scope (Khối lệnh `{ ... }`):** Các từ khóa hiện đại như `let`, `const` trong JS giữ biến chỉ tồn tại trong cặp ngoặc nhọn chứa nó (ví dụ bên trong lệnh `if` hoặc vòng lặp `for`).

### 7.2. Khái niệm Bao đóng (Closure)
**Closure** là khả năng một hàm con bên trong có thể "ghi nhớ" và truy cập các biến thuộc phạm vi của hàm cha bên ngoài, ngay cả khi hàm cha đã kết thúc thực thi và biến mất khỏi Call Stack!

```javascript
function createCounter() {
  let count = 0; // Biến này nằm trong private scope của createCounter

  return function() {
    count++; // Vẫn ghi nhớ biến count này!
    return count;
  };
}

const counterA = createCounter();
console.log(counterA()); // 1
console.log(counterA()); // 2

const counterB = createCounter(); // Tạo một counter mới độc lập
console.log(counterB()); // 1
```

---

## 8. Tóm Tắt & Ghi Nhớ Nhanh

1. **Biến** là định danh trỏ tới ô nhớ; ưu tiên dùng hằng số (`const`) nếu giá trị không cần thay đổi.
2. Phân biệt **Primitive Type** (sao chép giá trị) và **Reference Type** (sao chép con trỏ tham chiếu tới Heap).
3. Trong JavaScript, luôn dùng `===` để tránh cạm bẫy ép kiểu của `==`.
4. Viết các **Pure Function** bất cứ khi nào có thể để mã nguồn dễ bảo trì, dễ viết kiểm thử (unit test).
5. Nắm vững **Scope** và **Closure** để bảo vệ dữ liệu nội bộ không bị ghi đè từ bên ngoài.
