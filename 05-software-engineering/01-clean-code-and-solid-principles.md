# 01 - Mã Sạch & 5 Nguyên Lý SOLID (Clean Code & SOLID Principles)

> *"Bất kỳ kẻ ngốc nào cũng có thể viết code mà máy tính hiểu được. Lập trình viên giỏi viết code mà con người có thể hiểu được."* — **Martin Fowler**

---

## 1. Clean Code Là Gì? Tại Sao Cần Viết Mã Sạch?

Trong vòng đời của một dự án phần mềm:
- Thời gian **đọc code** chiếm tới **80 - 90%**, trong khi thời gian **gõ code mới** chỉ chiếm khoảng **10 - 20%**.
- Một đoạn mã "chạy được nhưng cẩu thả" (Spaghetti Code) có thể giúp bạn kịp hạn chót (deadline) hôm nay, nhưng sẽ biến thành món nợ kỹ thuật (**Technical Debt**) khổng lồ, khiến các tính năng sau này mất hàng tháng trời để sửa lỗi và làm sụp đổ toàn bộ dự án.

```text
       MÃ NGUỒN CẨU THẢ (DIRTY CODE)                    MÃ NGUỒN SẠCH (CLEAN CODE)
┌────────────────────────────────────────┐       ┌────────────────────────────────────────┐
│ - Đặt tên vô nghĩa: a, b, temp, fn1    │       │ - Tên gọi rõ ràng, tự giải thích nghĩa │
│ - Hàm dài 500 dòng làm 10 việc một lúc │       │ - Hàm ngắn gọn, chỉ làm đúng 1 việc    │
│ - Copy-paste code rải rác khắp nơi     │       │ - Tuân thủ nguyên tắc DRY, SOLID       │
│ - Đầy rẫy Magic Numbers: if (x == 42)  │       │ - Sử dụng Constants / Enums có ý nghĩa │
│ - Sửa chỗ này, hỏng chỗ kia!           │       │ - Dễ kiểm thử (Testable) và dễ mở rộng │
└────────────────────────────────────────┘       └────────────────────────────────────────┘
```

---

## 2. Nghệ Thuật Viết Clean Code Hàng Ngày

### 2.1. Quy tắc đặt tên có ý nghĩa (Intention-Revealing Names)
Tên biến, tên hàm, tên lớp phải trả lời được 3 câu hỏi: *Nó là cái gì? Nó làm nhiệm vụ gì? Tại sao nó tồn tại?*

```typescript
// ❌ Đặt tên tồi: Bí hiểm, buộc người đọc phải đoán
const d = 86400; // Số giây trong ngày?
function check(u: any) {
  return u.st === 1 && u.a > 18;
}

// ✅ Đặt tên sạch: Đọc như một câu văn tiếng Anh tự nhiên
const SECONDS_PER_DAY = 86400;
function isUserEligibleForRegistration(user: User): boolean {
  const isAccountActive = user.status === AccountStatus.ACTIVE;
  const isAdult = user.age >= 18;
  return isAccountActive && isAdult;
}
```

### 2.2. Thiết kế hàm: Hàm chỉ nên làm một việc duy nhất (Do One Thing)
Một hàm tốt phải thỏa mãn:
1. **Quy tắc kích thước:** Hàm nên ngắn gọn, lý tưởng từ **10 đến 20 dòng**.
2. **Một cấp độ trừu tượng (One Level of Abstraction):** Không nên vừa xử lý logic nghiệp vụ cao cấp vừa gọi các câu lệnh thao tác mảng cấp thấp trong cùng 1 hàm.
3. **Số lượng tham số tối ưu:** Đẹp nhất là **0 đến 2 tham số**. Nếu hàm cần truyền từ 3 tham số trở lên, hãy gom chúng lại thành một Object / DTO:
   ```typescript
   // ❌ Quá nhiều tham số rời rạc:
   function createUser(name: string, email: string, age: number, address: string, phone: string) {}

   // ✅ Gom thành DTO rõ ràng:
   function createUser(dto: CreateUserDto) {}
   ```

### 2.3. Bốn triết lý vàng của kỹ sư phần mềm
- **DRY (Don't Repeat Yourself):** Không bao giờ lặp lại cùng một đoạn logic ở hai nơi. Nếu logic thay đổi, bạn chỉ cần sửa ở đúng 1 chỗ duy nhất.
- **KISS (Keep It Simple, Stupid):** Giữ cho giải pháp luôn đơn giản nhất có thể. Đừng cố gắng thể hiện trình độ bằng những thuật toán "thần thánh" mà không ai trong team đọc hiểu được.
- **YAGNI (You Aren't Gonna Need It):** Đừng viết trước những tính năng hay trừu tượng hóa mà bạn nghĩ "biết đâu sau này sẽ cần". Hãy chỉ xây dựng những gì thực sự cần thiết cho yêu cầu hiện tại.
- **Boy Scout Rule (Quy tắc Hướng đạo sinh):** *"Luôn luôn để lại khu cắm trại sạch sẽ hơn lúc bạn vừa bước chân đến"*. Mỗi khi mở một file code ra sửa, hãy dọn dẹp ít nhất một dòng code xấu cũ.

---

## 3. Phân Tích Chuyên Sâu 5 Nguyên Lý SOLID

SOLID là 5 nguyên tắc thiết kế hướng đối tượng được đúc kết bởi Robert C. Martin (Uncle Bob), là nền tảng của mọi kiến trúc phần mềm đẳng cấp:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                         5 NGUYÊN LÝ THIẾT KẾ S.O.L.I.D                 │
└──────┬──────────┬──────────┬───────────────────┬────────────────┬──────┘
       │          │          │                   │                │
       ▼          ▼          ▼                   ▼                ▼
     S - SRP    O - OCP    L - LSP             I - ISP          D - DIP
   (Single     (Open /   (Liskov              (Interface       (Dependency
  Responsibility) Closed) Substitution)       Segregation)      Inversion)
```

---

### 1. S — Single Responsibility Principle (Nguyên lý Đơn trách nhiệm)
> *"Một Class hoặc Module chỉ nên có duy nhất một lý do để thay đổi."*

#### ❌ Code vi phạm SRP (Class "ôm đồm" quá nhiều việc):
```typescript
class OrderService {
  public calculateTotal(order: Order): number {
    // 1. Tính toán tiền hàng
    return order.items.reduce((sum, item) => sum + item.price, 0);
  }

  public saveToDatabase(order: Order): void {
    // 2. Chịu trách nhiệm kết nối CSDL (Vi phạm!)
    console.log("Lưu đơn hàng vào PostgreSQL...");
  }

  public sendEmailReceipt(order: Order): void {
    // 3. Chịu trách nhiệm gửi email (Vi phạm!)
    console.log("Gửi email hóa đơn cho khách hàng...");
  }
}
// Hậu quả: Nếu đổi template email, ta phải sửa OrderService.
// Nếu đổi từ PostgreSQL sang MongoDB, ta cũng phải sửa OrderService!
```

#### ✅ Code chuẩn mực sau khi tách trách nhiệm:
```typescript
class OrderCalculator {
  public calculateTotal(order: Order): number { /* Chỉ tính tiền */ }
}

class OrderRepository {
  public save(order: Order): void { /* Chỉ lo lưu DB */ }
}

class EmailNotificationService {
  public sendReceipt(order: Order): void { /* Chỉ lo gửi email */ }
}
```

---

### 2. O — Open/Closed Principle (Nguyên lý Đóng / Mở)
> *"Phần mềm nên MỞ cho việc mở rộng (Open for Extension), nhưng ĐÓNG đối với việc sửa đổi mã nguồn cũ (Closed for Modification)."*

Khi có yêu cầu nghiệp vụ mới, ta nên **viết thêm code mới** chứ không phải nhảy vào sửa đổi các đoạn code cũ đang chạy ổn định.

#### ❌ Code vi phạm OCP (Dùng chuỗi `if/else` hoặc `switch`):
```typescript
class PaymentProcessor {
  public process(amount: number, method: string): void {
    if (method === "PAYPAL") {
      // Logic PayPal...
    } else if (method === "STRIPE") {
      // Logic Stripe...
    } else if (method === "VNPAY") { // Mỗi lần thêm cổng mới là phải SỬA CODE CŨ!
      // Logic VNPay...
    }
  }
}
```

#### ✅ Code chuẩn OCP (Sử dụng Interface & Đa hình):
```typescript
// 1. Định nghĩa Interface chuẩn chung (Đóng với sửa đổi)
interface PaymentMethod {
  pay(amount: number): void;
}

// 2. Mở rộng bằng cách tạo các Class mới độc lập:
class PaypalPayment implements PaymentMethod {
  pay(amount: number): void { console.log(`Thanh toán ${amount}$ qua PayPal`); }
}

class VNPayPayment implements PaymentMethod {
  pay(amount: number): void { console.log(`Thanh toán ${amount} VNĐ qua VNPay`); }
}

// Khi muốn hỗ trợ thêm Momo, chỉ cần tạo class MomoPayment mà KHÔNG CẦN CHẠM VÀO code cũ!
class CheckoutService {
  public execute(payment: PaymentMethod, amount: number): void {
    payment.pay(amount);
  }
}
```

---

### 3. L — Liskov Substitution Principle (Nguyên lý Thay thế Liskov)
> *"Các đối tượng của Lớp Con phải có thể thay thế hoàn toàn cho Lớp Cha mà không làm thay đổi tính đúng đắn của chương trình."*

#### ❌ Bài toán kinh điển vi phạm Liskov: Hình vuông kế thừa Hình chữ nhật
Về mặt hình học đời thường, Hình vuông là một Hình chữ nhật. Nhưng trong lập trình hướng đối tượng, sự kế thừa này là một **thảm họa**:

```typescript
class Rectangle {
  constructor(protected width: number, protected height: number) {}
  setWidth(w: number) { this.width = w; }
  setHeight(h: number) { this.height = h; }
  getArea(): number { return this.width * this.height; }
}

class Square extends Rectangle {
  // Vì hình vuông có 4 cạnh bằng nhau, nên sửa chiều nào thì sửa cả 2:
  setWidth(w: number) { this.width = w; this.height = w; }
  setHeight(h: number) { this.width = h; this.height = h; }
}

function verifyRectangle(rect: Rectangle) {
  rect.setWidth(5);
  rect.setHeight(4);
  // Người dùng kỳ vọng: 5 * 4 = 20.
  // Nhưng nếu truyền vào Square: chiều rộng bị đổi thành 4, diện tích = 16 => BUG!
  console.assert(rect.getArea() === 20, "Vi phạm Liskov!");
}
```
**Giải pháp:** Tách `Square` và `Rectangle` thành 2 class riêng biệt cùng triển khai interface `Shape`.

---

### 4. I — Interface Segregation Principle (Nguyên lý Phân tách Giao diện)
> *"Thà tạo ra nhiều Interface nhỏ chuyên biệt, còn hơn ép một Class phải phụ thuộc vào một Interface khổng lồ chứa những hàm mà nó không dùng tới."*

#### ❌ Code vi phạm ISP:
```typescript
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
}

// Robot chỉ làm việc, không ăn không ngủ nhưng vẫn bị bắt implement:
class RobotWorker implements Worker {
  work(): void { console.log("Robot đang làm việc..."); }
  eat(): void { throw new Error("Robot không biết ăn!"); } // Lỗi vô nghĩa!
  sleep(): void { throw new Error("Robot không biết ngủ!"); }
}
```

#### ✅ Code chuẩn ISP:
```typescript
interface Workable { work(): void; }
interface Feedable { eat(): void; }

class HumanWorker implements Workable, Feedable {
  work() {}
  eat() {}
}

class RobotWorker implements Workable {
  work() {} // Chỉ implement đúng cái nó thực sự làm được!
}
```

---

### 5. D — Dependency Inversion Principle (Nguyên lý Đảo ngược Phụ thuộc)
> *"1. Các module cấp cao không nên phụ thuộc vào các module cấp thấp. Cả hai nên phụ thuộc vào sự trừu tượng hóa (Abstraction/Interface).*
> *2. Sự trừu tượng hóa không nên phụ thuộc vào chi tiết. Chi tiết nên phụ thuộc vào sự trừu tượng hóa."*

#### ❌ Code vi phạm DIP (Module cấp cao phụ thuộc cứng vào chi tiết cấp thấp):
```typescript
class MySQLDatabase {
  public saveOrder(order: any): void { /* Lưu vào MySQL */ }
}

class OrderManager {
  private db: MySQLDatabase; // Phụ thuộc cứng vào MySQL!
  
  constructor() {
    this.db = new MySQLDatabase(); // Nếu sau này muốn đổi sang Postgres hay Mongo thì phải đập đi xây lại!
  }
}
```

#### ✅ Code chuẩn DIP (Phụ thuộc qua Interface / Abstraction):
```typescript
// 1. Tầng Abstraction trung gian
interface IDatabase {
  saveOrder(order: any): void;
}

// 2. Các module chi tiết cấp thấp triển khai Abstraction:
class PostgreSQLDatabase implements IDatabase {
  saveOrder(order: any): void { console.log("Lưu vào PostgreSQL"); }
}

class MongoDatabase implements IDatabase {
  saveOrder(order: any): void { console.log("Lưu vào MongoDB"); }
}

// 3. Module cấp cao chỉ phụ thuộc vào Interface, nhận instance qua Constructor (Dependency Injection):
class OrderManager {
  constructor(private db: IDatabase) {} // Cực kỳ linh hoạt, muốn đổi DB nào chỉ cần truyền vào lúc khởi tạo!
}
```

---

## 4. Tóm Tắt & Ghi Nhớ Nhanh

1. **Clean Code** ưu tiên khả năng đọc hiểu của con người; đặt tên rõ ràng, hàm ngắn gọn dưới 20 dòng và chỉ làm đúng 1 việc.
2. Luôn nhớ 4 triết lý: **DRY** (không lặp code), **KISS** (đơn giản hóa), **YAGNI** (không làm thừa), và **Boy Scout Rule** (dọn dẹp mã liên tục).
3. **S (Single Responsibility):** Một lý do duy nhất để thay đổi class.
4. **O (Open/Closed):** Mở rộng tính năng bằng cách tạo class mới, không sửa code cũ.
5. **L (Liskov Substitution):** Lớp con không được làm thay đổi hành vi kỳ vọng của lớp cha.
6. **I (Interface Segregation):** Chia nhỏ Interface thành các giao diện chuyên biệt.
7. **D (Dependency Inversion):** Phụ thuộc vào Interface, không phụ thuộc vào Concrete Class.
