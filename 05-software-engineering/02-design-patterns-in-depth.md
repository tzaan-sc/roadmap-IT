# 02 - Các Mẫu Thiết Kế Thực Chiến (Design Patterns In-Depth)

> **Mục tiêu bài học:** Nắm vững bản chất của các mẫu thiết kế kinh điển (Design Patterns - Gang of Four) thường xuyên xuất hiện trong các dự án thực tế và framework lớn: Factory, Builder, Adapter, Decorator, Strategy, Observer, Repository và Dependency Injection. Hiểu rõ bài toán mà mỗi pattern giải quyết và tránh bẫy lạm dụng pattern (Over-engineering).

---

## 1. Design Patterns Là Gì? Tại Sao Cần Học?

**Design Pattern (Mẫu thiết kế)** không phải là một đoạn code có thể copy-paste. Nó là **một giải pháp tổng quát đã được kiểm nghiệm qua thời gian** để giải quyết các vấn đề thường gặp trong thiết kế phần mềm hướng đối tượng.

```text
                                 PHÂN LOẠI DESIGN PATTERNS
┌──────────────────────────────┬──────────────────────────────┬──────────────────────────────┐
│ 1. CREATIONAL (Khởi tạo)     │ 2. STRUCTURAL (Cấu trúc)     │ 3. BEHAVIORAL (Hành vi)      │
│ Cơ chế tạo đối tượng linh    │ Cách lắp ráp các Class và    │ Cách các đối tượng giao tiếp │
│ hoạt, giảm phụ thuộc cứng.   │ Object thành cấu trúc lớn.   │ và phân chia trách nhiệm.    │
│ - Factory Method             │ - Adapter                    │ - Strategy                   │
│ - Builder                    │ - Decorator                  │ - Observer                   │
│ - Singleton                  │ - Facade                     │ - Command                    │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
```

---

## 2. Nhóm Mẫu Khởi Tạo (Creational Patterns)

### 2.1. Factory Method Pattern (Nhà máy sản xuất)
- **Vấn đề:** Thay vì gọi trực tiếp `new PaypalPayment()` hay `new StripePayment()` rải rác khắp các controller, ta muốn tập trung toàn bộ logic khởi tạo vào một nơi duy nhất.

```typescript
interface PaymentGateway {
  processPayment(amount: number): void;
}

class PaypalGateway implements PaymentGateway {
  processPayment(amount: number) { console.log(`Thanh toán ${amount}$ qua PayPal`); }
}

class StripeGateway implements PaymentGateway {
  processPayment(amount: number) { console.log(`Thanh toán ${amount}$ qua Stripe`); }
}

// LỚP FACTORY: Tập trung logic khởi tạo
class PaymentFactory {
  public static create(type: "PAYPAL" | "STRIPE"): PaymentGateway {
    switch (type) {
      case "PAYPAL": return new PaypalGateway();
      case "STRIPE": return new StripeGateway();
      default: throw new Error("Cổng thanh toán không hợp lệ!");
    }
  }
}

// Sử dụng sạch sẽ:
const gateway = PaymentFactory.create("STRIPE");
gateway.processPayment(100);
```

---

### 2.2. Builder Pattern (Người thợ xây)
- **Vấn đề:** Khi một đối tượng có quá nhiều thuộc tính tùy chọn (Optional Fields). Nếu dùng constructor thông thường, bạn sẽ gặp lỗi **Telescoping Constructor** (hàm khởi tạo có 10 tham số dài dằng dặc, dễ truyền nhầm thứ tự).
- **Giải pháp:** Xây dựng đối tượng từng bước một bằng chuỗi phương thức liên tiếp (**Method Chaining**):

```typescript
class UserProfile {
  public name!: string;
  public email!: string;
  public age?: number;
  public phone?: string;
  public address?: string;
}

class UserProfileBuilder {
  private profile = new UserProfile();

  constructor(name: string, email: string) {
    this.profile.name = name;
    this.profile.email = email;
  }

  public setAge(age: number): this {
    this.profile.age = age;
    return this; // Trả về 'this' để xâu chuỗi method chaining!
  }

  public setPhone(phone: string): this {
    this.profile.phone = phone;
    return this;
  }

  public build(): UserProfile {
    return this.profile;
  }
}

// Sử dụng siêu trực quan:
const myUser = new UserProfileBuilder("Nguyen Nam", "nam@gmail.com")
  .setAge(25)
  .setPhone("0901234567")
  .build();
```

---

### 2.3. Singleton Pattern (Độc bản duy nhất)
- **Mục đích:** Đảm bảo một Class chỉ có **duy nhất một Instance** trong toàn bộ vòng đời ứng dụng (ví dụ: Database Connection Pool, App Configuration, Logger).

```typescript
class DatabaseConnection {
  private static instance: DatabaseConnection | null = null;

  // BẮT BUỘC: Khóa constructor bằng 'private' để không ai gọi 'new DatabaseConnection()' được từ bên ngoài!
  private constructor() {
    console.log("Khởi tạo kết nối CSDL...");
  }

  public static getInstance(): DatabaseConnection {
    if (!DatabaseConnection.instance) {
      DatabaseConnection.instance = new DatabaseConnection();
    }
    return DatabaseConnection.instance;
  }
}

const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2); // true (Cả 2 cùng trỏ chung một ô nhớ duy nhất!)
```

> [!WARNING]
> **Cảnh báo Anti-Pattern:** Không lạm dụng Singleton vì nó thực chất là một biến toàn cục ngầm, tạo sự phụ thuộc chặt chẽ và gây rất nhiều khó khăn khi viết Unit Test. Các framework hiện đại như NestJS hay Spring Boot ưu tiên dùng **Dependency Injection với Singleton Scope** thay vì viết Class Singleton thủ công.

---

## 3. Nhóm Mẫu Cấu Trúc (Structural Patterns)

### 3.1. Adapter Pattern (Phích cắm chuyển đổi)
- **Vấn đề:** Hệ thống hiện tại của bạn yêu cầu một Interface chuẩn, nhưng bạn phải tích hợp một thư viện bên thứ 3 có cách đặt tên hàm và tham số hoàn toàn khác.
- **Giải pháp:** Tạo một lớp Adapter làm trung gian phiên dịch:

```typescript
// Giao diện hệ thống của bạn yêu cầu:
interface NotificationService {
  send(message: string, recipient: string): void;
}

// Thư viện bên thứ 3 (Ví dụ SendGrid SDK cũ) có giao diện khác:
class LegacySendGridSDK {
  public dispatchEmail(targetEmail: string, contentBody: string) {
    console.log(`SendGrid gửi tới ${targetEmail}: ${contentBody}`);
  }
}

// LỚP ADAPTER: Cầu nối tương thích
class SendGridAdapter implements NotificationService {
  constructor(private legacySdk: LegacySendGridSDK) {}

  public send(message: string, recipient: string): void {
    // Phiên dịch từ hàm 'send' sang hàm 'dispatchEmail'
    this.legacySdk.dispatchEmail(recipient, message);
  }
}
```

---

### 3.2. Decorator Pattern (Trang trí bổ sung tính năng)
- **Mục đích:** Bổ sung hành vi mới cho một đối tượng tại thời điểm thực thi (Runtime) **mà không làm thay đổi mã nguồn của lớp gốc và không cần dùng kế thừa**.

```typescript
interface Coffee {
  getCost(): number;
  getDescription(): string;
}

class SimpleCoffee implements Coffee {
  getCost() { return 20; }
  getDescription() { return "Cà phê đen"; }
}

// Lớp Decorator thêm sữa
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  getCost() { return this.coffee.getCost() + 5; } // Cộng thêm tiền sữa
  getDescription() { return `${this.coffee.getDescription()} + Sữa tươi`; }
}

const myDrink = new MilkDecorator(new SimpleCoffee());
console.log(`${myDrink.getDescription()}: ${myDrink.getCost()}k`); // Cà phê đen + Sữa tươi: 25k
```

---

## 4. Nhóm Mẫu Hành Vi (Behavioral Patterns)

### 4.1. Strategy Pattern (Chiến lược linh hoạt)
- **Vấn đề:** Bạn có nhiều thuật toán tương đương nhau (ví dụ: các cách tính phí giao hàng: Tiết kiệm, Hỏa tốc, GrabBike).
- **Giải pháp:** Đóng gói mỗi giải thuật vào một Class riêng và cho phép hoán đổi chiến lược lúc runtime:

```typescript
interface ShippingStrategy {
  calculate(weightKg: number): number;
}

class StandardShipping implements ShippingStrategy {
  calculate(weightKg: number) { return weightKg * 10; }
}

class ExpressShipping implements ShippingStrategy {
  calculate(weightKg: number) { return weightKg * 25 + 50; }
}

// Context: Áp dụng chiến lược
class OrderDelivery {
  constructor(private strategy: ShippingStrategy) {}

  public setStrategy(strategy: ShippingStrategy) {
    this.strategy = strategy; // Cho phép đổi chiến lược bất kỳ lúc nào!
  }

  public getShippingFee(weight: number): number {
    return this.strategy.calculate(weight);
  }
}
```

---

### 4.2. Observer Pattern (Người quan sát / Pub-Sub)
- **Vấn đề:** Khi trạng thái của một đối tượng thay đổi, ta cần tự động thông báo cho hàng loạt đối tượng khác mà không muốn các đối tượng đó phụ thuộc cứng vào nhau.

```typescript
interface Observer {
  update(news: string): void;
}

class NewsletterPublisher {
  private subscribers: Observer[] = [];

  public subscribe(obs: Observer) { this.subscribers.push(obs); }
  public unsubscribe(obs: Observer) { this.subscribers = this.subscribers.filter(s => s !== obs); }

  public notify(news: string) {
    // Phát thông báo tới tất cả người theo dõi
    this.subscribers.forEach(sub => sub.update(news));
  }
}

class UserSubscriber implements Observer {
  constructor(private name: string) {}
  update(news: string) { console.log(`[${this.name}] Nhận tin tức mới: ${news}`); }
}

const publisher = new NewsletterPublisher();
const alice = new UserSubscriber("Alice");
const bob = new UserSubscriber("Bob");

publisher.subscribe(alice);
publisher.subscribe(bob);
publisher.notify("Khóa học IT mới vừa ra mắt!");
```

---

## 5. Mẫu Kiến Trúc: Repository Pattern

Đây là mẫu thiết kế xuất hiện trong **99% các dự án Backend chuyên nghiệp**, nhằm tách biệt hoàn toàn Tầng nghiệp vụ (Business Logic / Service) khỏi Tầng truy xuất dữ liệu (Data Access / Database):

```text
[ ORDER SERVICE ] ──► [ IOrderRepository (Interface) ]
                             ▲
              ┌──────────────┴──────────────┐
              │                             │
[ PostgresOrderRepository ]      [ MongoOrderRepository ]
```

```typescript
interface IUserRepository {
  findById(id: number): Promise<User | null>;
  save(user: User): Promise<void>;
}

// Tầng Service chỉ tương tác với Interface Repository, không quan tâm DB dùng SQL hay NoSQL:
class UserService {
  constructor(private userRepo: IUserRepository) {}

  async getUserDetails(id: number) {
    const user = await this.userRepo.findById(id);
    if (!user) throw new Error("User không tồn tại!");
    return user;
  }
}
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **Factory Method:** Tập trung việc khởi tạo đối tượng phức tạp vào một chỗ.
2. **Builder:** Tạo đối tượng có nhiều thuộc tính tùy biến bằng method chaining rõ ràng.
3. **Singleton:** Đảm bảo duy nhất 1 instance, nhưng cẩn trọng tránh biến thành Anti-Pattern.
4. **Adapter:** Cầu nối tương thích giữa 2 giao diện không khớp nhau.
5. **Decorator:** Mở rộng tính năng động cho đối tượng lúc runtime thay vì dùng kế thừa.
6. **Strategy:** Đóng gói các giải thuật thay thế nhau, xóa bỏ các chuỗi `if/else` dài dòng.
7. **Observer:** Cơ chế phát thông báo sự kiện 1-N (Publisher - Subscriber).
8. **Repository:** Chuẩn mực cách ly tầng Business Logic khỏi các câu lệnh Database.
