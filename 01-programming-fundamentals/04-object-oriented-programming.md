# 04 - Lập Trình Hướng Đối Tượng (Object-Oriented Programming - OOP)

> **Mục tiêu bài học:** Chuyển đổi tư duy lập trình từ việc viết mã tuần tự thủ tục (Procedural) sang mô hình hóa bài toán thực tế thành các đối tượng (Objects). Nắm vững bản chất của Class, Object và 4 trụ cột vàng của OOP: Đóng gói (Encapsulation), Trừu tượng (Abstraction), Kế thừa (Inheritance), Đa hình (Polymorphism).

---

## 1. Tư Duy OOP Là Gì? Tại Sao Cần OOP?

Trong thế giới thực, mọi vật thể xung quanh chúng ta đều là **Đối tượng (Object)**: chiếc xe hơi, tài khoản ngân hàng, người dùng, đơn đặt hàng...
Mỗi đối tượng đều có:
1. **Trạng thái (State / Attributes):** Dữ liệu đặc trưng của đối tượng (ví dụ: màu sắc, số dư, tên, tuổi).
2. **Hành vi (Behavior / Methods):** Những hành động đối tượng có thể làm (ví dụ: nổ máy, rút tiền, đăng nhập, thanh toán).

```text
                  LẬP TRÌNH HƯỚNG THỦ TỤC                 LẬP TRÌNH HƯỚNG ĐỐI TƯỢNG (OOP)
                   (Procedural Paradigm)                           (OOP Paradigm)

                  ┌─────────────────────┐                   ┌───────────────────────────┐
                  │ DỮ LIỆU TỰ DO       │                   │          OBJECT           │
                  │ (Biến toàn cục,     │                   │  ┌─────────────────────┐  │
                  │  mảng rải rác)      │                   │  │ Dữ liệu (Attributes)│  │
                  └──────────┬──────────┘                   │  ├─────────────────────┤  │
                             ▼                              │  │ Hành vi (Methods)   │  │
                  ┌─────────────────────┐                   │  └─────────────────────┘  │
                  │ CÁC HÀM XỬ LÝ       │                   └───────────────────────────┘
                  │ (Hàm nào cũng chạm  │                     Dữ liệu và hành vi được
                  │  vào được dữ liệu)  │                     đóng gói chung một chỗ!
                  └─────────────────────┘
```

OOP giúp giải quyết bài toán phức tạp của các hệ thống phần mềm lớn bằng cách gom nhóm dữ liệu và các hành vi liên quan lại với nhau, che chắn để chúng không bị các đoạn mã bên ngoài sửa đổi bừa bãi.

---

## 2. Lớp (Class) vs Đối Tượng (Object)

- **Class (Lớp):** Là một **bản thiết kế (Blueprint)** hoặc khuôn đúc. Nó định nghĩa xem một đối tượng thuộc loại này sẽ có những thuộc tính và phương thức nào, nhưng bản thân Class chưa chiếm không gian bộ nhớ của một đối tượng cụ thể.
- **Object (Đối tượng / Instance):** Là **một thực thể cụ thể** được sinh ra (khởi tạo) từ Class đó và chiếm giữ một vùng nhớ thực sự trên Heap.

```text
       BẢN THIẾT KẾ (CLASS: Car)
       ┌────────────────────────┐
       │ - brand (Hãng)         │
       │ - color (Màu)          │
       │ - speed (Tốc độ)       │
       ├────────────────────────┤
       │ + drive()              │
       │ + brake()              │
       └───────────┬────────────┘
                   │ Đúc ra các đối tượng cụ thể (Instantiate)
         ┌─────────┴─────────┐
         ▼                   ▼
  OBJECT 1 (carA)     OBJECT 2 (carB)
  brand: "Toyota"     brand: "Tesla"
  color: "Trắng"      color: "Đen"
  speed: 0            speed: 0
```

### Minh họa bằng code:
```python
# Định nghĩa Class trong Python
class Car:
    # Hàm khởi tạo (Constructor): chạy tự động khi tạo đối tượng mới
    def __init__(self, brand, color):
        self.brand = brand   # Thuộc tính (Attribute)
        self.color = color
        self.speed = 0
        
    def drive(self, target_speed):
        self.speed = target_speed
        print(f"Chiếc xe {self.brand} màu {self.color} đang chạy với vận tốc {self.speed} km/h.")

# Khởi tạo các Object (Instances)
car1 = Car("Toyota", "Trắng")
car2 = Car("Tesla", "Đen")

car1.drive(80) # Gọi phương thức
```

```javascript
// Cú pháp tương đương trong JavaScript (ES6)
class Car {
  constructor(brand, color) {
    this.brand = brand;
    this.color = color;
    this.speed = 0;
  }

  drive(targetSpeed) {
    this.speed = targetSpeed;
    console.log(`Chiếc xe ${this.brand} màu ${this.color} đang chạy với vận tốc ${this.speed} km/h.`);
  }
}

const car1 = new Car("Toyota", "Trắng");
car1.drive(80);
```

---

## 3. Bốn Trụ Cột Vàng Của OOP

```text
                 ┌──────────────────────────────────────────────┐
                 │             4 TRỤ CỘT CỦA OOP                │
                 └──────┬──────────┬──────────┬──────────┬──────┘
                        │          │          │          │
         ┌──────────────┘          │          │          └─────────────┐
         ▼                         ▼          ▼                        ▼
  1. ENCAPSULATION          2. ABSTRACTION  3. INHERITANCE      4. POLYMORPHISM
     (Đóng gói)             (Trừu tượng)     (Kế thừa)             (Đa hình)
  Che giấu dữ liệu nội     Ẩn chi tiết cài  Tái sử dụng code,    Một giao diện,
  bộ, bảo vệ bằng          đặt, chỉ cung    mở rộng từ lớp       nhiều cách thực
  Getter & Setter.         cấp giao diện.   cha sang con.        hiện khác nhau.
```

---

### Trụ Cột 1: Đóng Gói (Encapsulation) & Che Giấu Thông Tin (Data Hiding)

Đóng gói là việc **ngăn chặn mã bên ngoài can thiệp trực tiếp vào các thuộc tính bên trong đối tượng**, mà bắt buộc phải thông qua các phương thức công khai (Public Methods / Getters / Setters).

#### Tại sao không nên cho phép truy cập trực tiếp biến?
Giả sử bạn có thuộc tính `balance` (số dư tài khoản ngân hàng):
Nếu ai cũng có thể ghi `account.balance = -999999999`, hệ thống sẽ sập và thất thoát tiền bạc!

```python
class BankAccount:
    def __init__(self, owner, initial_balance):
        self.owner = owner
        # Dùng tiền tố 2 dấu gạch dưới '__' để biến thành private trong Python
        self.__balance = initial_balance 

    # Getter: Chỉ cho xem số dư, không cho tự tiện sửa
    def get_balance(self):
        return self.__balance

    # Setter / Method: Kiểm soát chặt chẽ quy tắc nghiệp vụ khi nạp/rút tiền
    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("Số tiền nạp vào phải lớn hơn 0!")
        self.__balance += amount

    def withdraw(self, amount):
        if amount > self.__balance:
            raise ValueError("Số dư không đủ để thực hiện giao dịch!")
        self.__balance -= amount
```

---

### Trụ Cột 2: Trừu Tượng Hóa (Abstraction)

Trừu tượng hóa là việc **tập trung vào những gì đối tượng có thể làm (Interface) thay vì cách nó làm điều đó ở bên trong (Implementation details)**.

*Ví dụ đời thực:* Khi bạn bật tivi bằng chiếc điều khiển (Remote), bạn chỉ cần ấn nút nguồn `ON/OFF`. Bạn không cần biết sóng hồng ngoại được mã hóa ra sao, mạch điện tử bên trong tivi giải mã xung điện thế nào. Điều khiển tivi chính là một tầng trừu tượng!

```python
from abc import ABC, abstractmethod

# Lớp trừu tượng (Abstract Class) làm khuôn mẫu chuẩn
class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount):
        """Mọi cổng thanh toán bắt buộc phải có hàm này!"""
        pass

# Các lớp cụ thể tự cài đặt chi tiết:
class PaypalPayment(PaymentProcessor):
    def pay(self, amount):
        print(f"Xử lý chuyển {amount}$ qua cổng API PayPal.")

class VNPayPayment(PaymentProcessor):
    def pay(self, amount):
        print(f"Tạo mã QR VNPay cho giao dịch trị giá {amount} VNĐ.")
```

---

### Trụ Cột 3: Kế Thừa (Inheritance)

Kế thừa cho phép một lớp con (**Subclass**) sở hữu lại tất cả thuộc tính và phương thức của một lớp cha (**Superclass**), giúp tái sử dụng mã nguồn và tránh việc viết lặp lại code (DRY - Don't Repeat Yourself).

```text
                    [ LỚP CHA: Employee ]
                    - name
                    - salary
                    + work()
                       ▲
         ┌─────────────┴─────────────┐
         │                           │
  [ LỚP CON: Developer ]      [ LỚP CON: Designer ]
  - programming_language      - design_tool (Figma)
  + code()                    + draw_ui()
```

```python
class Employee:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        
    def show_info(self):
        print(f"Nhân viên: {self.name}, Mức lương: {self.salary}$")

# Lớp Developer kế thừa từ Employee
class Developer(Employee):
    def __init__(self, name, salary, language):
        # Gọi constructor của lớp cha bằng hàm super()
        super().__init__(name, salary)
        self.language = language
        
    def code(self):
        print(f"{self.name} đang lập trình bằng ngôn ngữ {self.language}.")

dev = Developer("Nam", 2500, "Go")
dev.show_info() # Tái sử dụng phương thức của lớp cha!
dev.code()
```

> [!WARNING]
> **Nguyên tắc vàng của kiến trúc phần mềm:**
> **"Favor Object Composition over Class Inheritance" (Ưu tiên Cấu thành hơn Kế thừa).**
> - Kế thừa biểu diễn mối quan hệ **"Là Một" (IS-A)**: `Developer IS-A Employee`. Nếu lạm dụng kế thừa quá 3-4 tầng, mã nguồn sẽ trở nên cực kỳ cứng nhắc và khó sửa đổi.
> - Cấu thành biểu diễn mối quan hệ **"Có Một" (HAS-A)**: `Car HAS-AN Engine`. Thay vì kế thừa `Engine`, chiếc xe chỉ cần chứa một đối tượng `Engine` bên trong là đủ.

---

### Trụ Cột 4: Đa Hình (Polymorphism)

Từ Hy Lạp: *Poly* (nhiều) + *Morph* (hình thái). Đa hình cho phép **các đối tượng thuộc các lớp khác nhau cùng phản hồi một lời gọi phương thức theo cách riêng của chúng**.

```text
               Lời gọi hàm: animal.make_sound()
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
   Đối tượng DOG        Đối tượng CAT        Đối tượng DUCK
   Kêu: "Gâu gâu!"      Kêu: "Meo meo!"      Kêu: "Quạc quạc!"
```

#### Hai dạng đa hình chính:
1. **Method Overriding (Ghi đè - Runtime Polymorphism):** Lớp con viết lại phần thân của một phương thức đã có ở lớp cha cho phù hợp với bản thân nó.
2. **Method Overloading (Nạp chồng - Compile-time Polymorphism):** Nhiều hàm cùng tên trong một lớp nhưng khác nhau về số lượng hoặc kiểu dữ liệu của tham số (rất phổ biến trong C++, Java, C#).

```python
class Dog:
    def speak(self):
        return "Gâu gâu!"

class Cat:
    def speak(self):
        return "Meo meo!"

class Bird:
    def speak(self):
        return "Líu lo!"

# Hàm này có thể làm việc với BẤT KỲ con vật nào có phương thức speak()
# (Đây là kỹ thuật Duck Typing kinh điển trong OOP hiện đại)
def make_animal_talk(animal):
    print(animal.speak())

zoo = [Dog(), Cat(), Bird()]
for pet in zoo:
    make_animal_talk(pet) # Mỗi con vật phản hồi một kiểu hoàn toàn tự nhiên!
```

---

## 4. Giới Thiệu 5 Nguyên Lý Thiết Kế SOLID Cốt Lõi

Khi bạn xây dựng các dự án lớn, 4 tính chất OOP là chưa đủ. Bạn cần nắm vững 5 nguyên lý vàng **SOLID** (do Robert C. Martin - Uncle Bob đề xuất):

- **S - Single Responsibility Principle (SRP):** Mỗi Class chỉ nên chịu **một trách nhiệm duy nhất** và chỉ có đúng một lý do để thay đổi.
- **O - Open/Closed Principle (OCP):** Module nên **Mở cho việc mở rộng (Open for extension)** nhưng **Đóng đối với việc sửa đổi (Closed for modification)**.
- **L - Liskov Substitution Principle (LSP):** Lớp con phải có thể thay thế hoàn toàn cho lớp cha mà không làm hỏng tính đúng đắn của chương trình.
- **I - Interface Segregation Principle (ISP):** Thà chia nhỏ thành nhiều Interface chuyên biệt còn hơn tạo ra một Interface "khổng lồ" bắt lớp con phải cài đặt những hàm không dùng tới.
- **D - Dependency Inversion Principle (DIP):** Các module cấp cao không nên phụ thuộc trực tiếp vào module cấp thấp; cả hai nên phụ thuộc vào sự trừu tượng hóa (Abstraction/Interface).

---

## 5. Tóm Tắt & Ghi Nhớ Nhanh

1. **Class** là bản vẽ thiết kế; **Object** là ngôi nhà thực tế được xây dựng trên bộ nhớ Heap.
2. **Đóng gói (Encapsulation):** Che giấu dữ liệu bằng `private`, truy cập an toàn qua getter/setter.
3. **Trừu tượng (Abstraction):** Đơn giản hóa vấn đề, tập trung vào "làm cái gì" thay vì "làm như thế nào".
4. **Kế thừa (Inheritance):** Tái sử dụng mã nguồn lớp cha (`super`), nhưng nhớ ưu tiên Composition over Inheritance.
5. **Đa hình (Polymorphism):** Cùng một tên gọi phương thức nhưng các đối tượng khác nhau tự thực thi theo phong cách riêng.
