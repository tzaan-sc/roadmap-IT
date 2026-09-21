# 05 - Thực Hành Tái Cấu Trúc Mã & Bộ Câu Hỏi Phỏng Vấn (Practice Refactoring & Interview)

> **Mục tiêu bài học:** Trực tiếp chuyển hóa một đoạn mã "thảm họa Spaghetti Code" thành kiến trúc Clean Code chuẩn mực áp dụng SOLID và Design Patterns; thực hành viết bộ Unit Test với kỹ thuật Mocking chuyên nghiệp; cùng với bộ 15 câu hỏi phỏng vấn kỹ thuật Software Engineering cốt lõi giúp bạn tự tin ứng tuyển các vị trí Mid-level / Senior Engineer.

---

## PHẦN 1: CÁC BÀI LAB THỰC CHIẾN

### LAB 1: Thử Thách Tái Cấu Trúc Mã Nguồn (Refactoring Challenge)

#### Tình huống ban đầu:
Dưới đây là một đoạn code xử lý đơn hàng điển hình của người mới học: Vi phạm nghiêm trọng nguyên lý Single Responsibility (SRP), Open/Closed (OCP), và Dependency Inversion (DIP):

```typescript
// ❌ SPAGHETTI CODE (Thảm họa thiết kế):
class OrderProcessor {
  public process(orderData: any, paymentType: string): void {
    // 1. Vi phạm SRP: Tính toán tiền hàng lẫn lộn
    let total = 0;
    for (let item of orderData.items) {
      total += item.price * item.quantity;
    }
    if (total > 500) {
      total = total * 0.9; // Giảm 10%
    }

    // 2. Vi phạm OCP: Dùng chuỗi if/else cứng nhắc để thanh toán
    if (paymentType === "PAYPAL") {
      console.log(`Kết nối PayPal API trừ ${total}$`);
    } else if (paymentType === "STRIPE") {
      console.log(`Kết nối Stripe API trừ ${total}$`);
    }

    // 3. Vi phạm DIP: Tự tạo và kết nối cứng vào CSDL
    console.log(`Lưu đơn hàng ${orderData.id} vào PostgreSQL Database`);

    // 4. Vi phạm SRP: Gửi email thông báo
    console.log(`Gửi email tới ${orderData.customerEmail}: Đơn hàng thành công!`);
  }
}
```

---

#### Các bước tái cấu trúc chuẩn mực:

**Bước 1: Áp dụng Strategy Pattern cho cổng thanh toán (Giải quyết OCP):**
```typescript
export interface PaymentStrategy {
  pay(amount: number): boolean;
}

export class PaypalStrategy implements PaymentStrategy {
  pay(amount: number): boolean {
    console.log(`✅ Thanh toán ${amount}$ qua cổng PayPal`);
    return true;
  }
}

export class StripeStrategy implements PaymentStrategy {
  pay(amount: number): boolean {
    console.log(`✅ Thanh toán ${amount}$ qua cổng Stripe`);
    return true;
  }
}
```

**Bước 2: Áp dụng Repository Pattern và Service chuyên biệt (Giải quyết SRP & DIP):**
```typescript
export interface OrderItem {
  name: string;
  price: number;
  quantity: number;
}

export interface Order {
  id: string;
  customerEmail: string;
  items: OrderItem[];
  totalAmount: number;
}

export interface OrderRepository {
  save(order: Order): Promise<void>;
}

export interface NotificationService {
  sendReceipt(toEmail: string, orderId: string, amount: number): Promise<void>;
}

// Tầng tính toán độc lập:
export class PricingCalculator {
  public static calculate(items: OrderItem[]): number {
    const rawTotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    return rawTotal > 500 ? rawTotal * 0.9 : rawTotal; // Giảm 10% nếu > 500$
  }
}

// TẦNG NGHIỆP VỤ LÕI (USE CASE): Sạch sẽ, không dính líu chi tiết cấp thấp!
export class CheckoutOrderUseCase {
  constructor(
    private orderRepo: OrderRepository,
    private notifier: NotificationService
  ) {}

  public async execute(
    orderId: string,
    customerEmail: string,
    items: OrderItem[],
    payment: PaymentStrategy
  ): Promise<Order> {
    const totalAmount = PricingCalculator.calculate(items);

    // 1. Thực hiện thanh toán qua Strategy
    const paymentSuccess = payment.pay(totalAmount);
    if (!paymentSuccess) {
      throw new Error("Giao dịch thanh toán thất bại!");
    }

    const order: Order = { id: orderId, customerEmail, items, totalAmount };

    // 2. Lưu vào CSDL qua Repository Abstraction
    await this.orderRepo.save(order);

    // 3. Gửi thông báo
    await this.notifier.sendReceipt(customerEmail, orderId, totalAmount);

    return order;
  }
}
```

---

### LAB 2: Viết Bộ Kiểm Thử Đơn Vị Với Mocking Chuyên Nghiệp

Bây giờ ta sẽ viết Unit Test cho `CheckoutOrderUseCase` ở trên. Nhờ thiết kế chuẩn Dependency Inversion, ta có thể kiểm thử toàn bộ nghiệp vụ mà **không cần đụng tới Database thật hay tài khoản PayPal thật**:

File `checkout.test.ts` (Sử dụng Vitest / Jest):
```typescript
import { describe, it, expect, vi } from 'vitest';
import { CheckoutOrderUseCase, OrderRepository, NotificationService, PaymentStrategy } from './checkout';

describe('CheckoutOrderUseCase', () => {
  it('thanh toán thành công, lưu đơn hàng vào DB và gửi email thông báo', async () => {
    // 1. ARRANGE (Chuẩn bị các Mock Doubles)
    const mockOrderRepo: OrderRepository = {
      save: vi.fn().mockResolvedValue(undefined), // Mock hàm save trả về Promise thành công
    };

    const mockNotifier: NotificationService = {
      sendReceipt: vi.fn().mockResolvedValue(undefined),
    };

    const mockPayment: PaymentStrategy = {
      pay: vi.fn().mockReturnValue(true), // Mock thanh toán luôn thành công
    };

    const useCase = new CheckoutOrderUseCase(mockOrderRepo, mockNotifier);

    const items = [
      { name: 'Bàn phím cơ', price: 200, quantity: 1 },
      { name: 'Màn hình 4K', price: 400, quantity: 1 } // Tổng 600$ -> Giảm 10% còn 540$
    ];

    // 2. ACT (Thực thi)
    const result = await useCase.execute('ORD_001', 'khach@gmail.com', items, mockPayment);

    // 3. ASSERT (Kiểm chứng hành vi)
    expect(result.totalAmount).toBe(540); // Đã áp dụng chiết khấu đúng!
    expect(mockPayment.pay).toHaveBeenCalledWith(540); // Đã gọi cổng thanh toán với số tiền 540$
    expect(mockOrderRepo.save).toHaveBeenCalledOnce(); // Bắt buộc phải lưu DB đúng 1 lần!
    expect(mockNotifier.sendReceipt).toHaveBeenCalledWith('khach@gmail.com', 'ORD_001', 540);
  });

  it('ném ra ngoại lệ và không lưu DB nếu thanh toán bị từ chối', async () => {
    const mockOrderRepo: OrderRepository = { save: vi.fn() };
    const mockNotifier: NotificationService = { sendReceipt: vi.fn() };
    
    // Giả lập thanh toán bị lỗi (ví dụ thẻ hết tiền):
    const mockPayment: PaymentStrategy = {
      pay: vi.fn().mockReturnValue(false),
    };

    const useCase = new CheckoutOrderUseCase(mockOrderRepo, mockNotifier);

    // Kiểm chứng ném ra lỗi
    await expect(
      useCase.execute('ORD_002', 'khach@gmail.com', [{ name: 'Chuột', price: 50, quantity: 1 }], mockPayment)
    ).rejects.toThrow('Giao dịch thanh toán thất bại!');

    // Đảm bảo TUYỆT ĐỐI KHÔNG lưu vào DB nếu chưa trả tiền!
    expect(mockOrderRepo.save).not.toHaveBeenCalled();
    expect(mockNotifier.sendReceipt).not.toHaveBeenCalled();
  });
});
```

---

## PHẦN 2: BỘ 15 CÂU HỎI PHỎNG VẤN KỸ NGHỆ PHẦN MỀM CỐT LÕI (CÓ LỜI GIẢI)

### Câu 1: Phân biệt sự khác nhau giữa "Clean Code" và "Code chạy được"?
- **Trả lời:**
  - **Code chạy được:** Chỉ quan tâm đến việc thỏa mãn máy tính ở thời điểm hiện tại. Nó có thể đầy rẫy biến tắt, hàm dài nghìn dòng, phụ thuộc chằng chịt và không có bài kiểm thử nào.
  - **Clean Code:** Viết để cho con người (đồng nghiệp và chính bạn sau 6 tháng) đọc hiểu và mở rộng dễ dàng. Clean Code có cấu trúc rõ ràng, đặt tên tự giải thích, hàm ngắn gọn chỉ làm 1 việc, tuân thủ nguyên lý SOLID, và được bảo vệ bởi hệ thống kiểm thử tự động.

### Câu 2: Trình bày nguyên lý Single Responsibility Principle (SRP) bằng ví dụ thực tế?
- **Trả lời:**
  - SRP quy định: *"Một class chỉ nên có một lý do duy nhất để thay đổi"*.
  - Ví dụ: Nếu một Class `InvoiceReport` vừa chịu trách nhiệm tính toán tiền bạc, vừa tự format ra chuỗi HTML, vừa tự kết nối mạng để gửi email: Nó có tới 3 lý do để bị sửa (thay đổi công thức tính tiền, thay đổi giao diện HTML, đổi nhà cung cấp email).
  - Giải pháp chuẩn: Tách thành 3 class độc lập: `InvoiceCalculator` (chỉ tính toán), `InvoiceHtmlFormatter` (chỉ hiển thị) và `InvoiceEmailSender` (chỉ truyền tin).

### Câu 3: Tại sao bài toán Hình vuông kế thừa Hình chữ nhật lại vi phạm nguyên lý Liskov Substitution (LSP)?
- **Trả lời:**
  - Về hình học toán học, hình vuông là một hình chữ nhật đặc biệt. Nhưng trong hướng đối tượng, `Rectangle` cho phép thay đổi chiều rộng (`setWidth`) độc lập mà không ảnh hưởng tới chiều cao (`setHeight`).
  - Khi `Square` kế thừa `Rectangle`, nó buộc phải ghi đè: sửa `setWidth` thì tự động đổi luôn `setHeight`. Khi một hàm client nhận vào `Rectangle` và gọi `setWidth(5)` rồi `setHeight(4)`, hàm đó kỳ vọng diện tích là 20. Nhưng nếu truyền `Square` vào, diện tích sẽ là 16. Lớp con đã làm phá vỡ hành vi kỳ vọng của lớp cha, dẫn tới lỗi ngầm vi phạm LSP.

### Câu 4: Phân biệt sự khác nhau giữa Dependency Inversion Principle (DIP), Inversion of Control (IoC), và Dependency Injection (DI)?
- **Trả lời:**
  - **DIP (Nguyên lý):** Là triết lý thiết kế cấp cao quy định rằng module cấp cao không nên phụ thuộc trực tiếp vào module cấp thấp; cả hai nên phụ thuộc vào Interface trừu tượng.
  - **IoC (Mô hình / Cơ chế):** Là mô hình đảo ngược quyền kiểm soát luồng chương trình hoặc việc khởi tạo đối tượng (nhường quyền cho Framework).
  - **DI (Kỹ thuật thực thi):** Là cách thức cụ thể để đưa (tiêm) đối tượng phụ thuộc vào bên trong một class (qua Constructor hoặc Setter) thay vì để class đó tự khởi tạo bằng `new`.

### Câu 5: Khi nào nên sử dụng Strategy Pattern thay vì câu lệnh `if/else` hoặc `switch/case`?
- **Trả lời:**
  - Nên dùng Strategy Pattern khi:
    1. Số lượng thuật toán/nhánh logic có xu hướng tăng dần theo thời gian (ví dụ: các phương thức thanh toán, các thuật toán nén ảnh, các cách tính thuế).
    2. Các thuật toán có logic tính toán phức tạp, nếu gom chung vào một file sẽ biến nó thành file khổng lồ vi phạm OCP và SRP.
    3. Ta cần hoán đổi thuật toán một cách linh hoạt tại thời điểm thực thi (Runtime) dựa trên dữ liệu người dùng.

### Câu 6: Singleton Pattern là gì? Tại sao nhiều chuyên gia coi nó là một Anti-Pattern?
- **Trả lời:**
  - Singleton đảm bảo một Class chỉ có duy nhất một Instance trong toàn bộ ứng dụng và cung cấp điểm truy cập toàn cục tới nó.
  - **Lý do bị coi là Anti-Pattern khi lạm dụng:**
    1. Nó tạo ra trạng thái toàn cục ẩn (**Hidden Global State**), làm các module bị phụ thuộc ngầm vào nhau.
    2. Gây cực kỳ khó khăn khi viết **Unit Test song song**, vì bài test A sửa trạng thái của Singleton có thể làm bài test B bị lỗi ngoài ý muốn.
    3. Vi phạm nguyên lý Single Responsibility (vừa quản lý logic nghiệp vụ vừa tự quản lý vòng đời tạo instance của chính nó).

### Câu 7: Factory Method Pattern giải quyết bài toán gì trong thiết kế hướng đối tượng?
- **Trả lời:**
  - Nó định nghĩa một phương thức chuyên dùng để tạo đối tượng, nhưng để cho các lớp con hoặc logic bên trong quyết định Class cụ thể nào sẽ được khởi tạo.
  - Giúp che giấu sự phức tạp của quá trình khởi tạo (ví dụ khởi tạo đối tượng cần truyền 10 cấu hình phức tạp) và giúp mã nguồn của Client không bị dính chặt vào các Concrete Class cụ thể.

### Câu 8: Phân biệt sự khác nhau giữa Adapter Pattern và Decorator Pattern?
- **Trả lời:**
  - **Adapter Pattern:** Thay đổi **giao diện (Interface)** của một đối tượng có sẵn để nó tương thích với một giao diện khác mà Client yêu cầu (mục đích là "chuyển đổi phích cắm").
  - **Decorator Pattern:** Giữ **nguyên giao diện (Interface)** gốc của đối tượng, nhưng bọc thêm bên ngoài để **bổ sung các tính năng/hành vi mới** lúc runtime mà không cần sửa code cũ (mục đích là "nâng cấp tính năng").

### Câu 9: Phân biệt Modular Monolith và Microservices? Khi nào doanh nghiệp nên chuyển sang Microservices?
- **Trả lời:**
  - **Modular Monolith:** Toàn bộ hệ thống chạy chung một tiến trình duy nhất (1 file deploy), nhưng bên trong mã nguồn được phân chia thành các Module có ranh giới nghiệp vụ độc lập, giao tiếp qua Public Interface.
  - **Microservices:** Hệ thống được chia thành hàng chục tiến trình độc lập, mỗi service sở hữu CSDL riêng và giao tiếp qua mạng (HTTP/gRPC/Kafka).
  - **Khi nào nên chuyển sang Microservices:** Chỉ khi doanh nghiệp đã phát triển tới quy mô rất lớn (hàng trăm kỹ sư chia thành nhiều team độc lập cần release tính năng độc lập mà không chờ đợi nhau) và khi có các phân hệ có đặc thù tải đột biến cần scale phần cứng riêng biệt.

### Câu 10: Trình bày quy tắc phụ thuộc (The Dependency Rule) trong Clean Architecture của Uncle Bob?
- **Trả lời:**
  - Quy tắc cốt lõi: *"Các phần phụ thuộc mã nguồn chỉ được phép trỏ hướng vào bên trong (Inward Dependency)"*.
  - Các tầng nghiệp vụ cốt lõi bên trong (**Entities** và **Use Cases**) không bao giờ được phép biết hay phụ thuộc vào bất kỳ thành phần nào của tầng bên ngoài (**Controllers, Databases, Web Frameworks**). Nhờ đó, logic nghiệp vụ của doanh nghiệp hoàn toàn độc lập, không bị ảnh hưởng khi công nghệ framework bên ngoài thay đổi.

### Câu 11: Kim tự tháp kiểm thử (Testing Pyramid) phân bổ tỷ lệ các loại test như thế nào và tại sao?
- **Trả lời:**
  - Tỷ lệ chuẩn: **70% Unit Tests, 20% Integration Tests, 10% End-to-End (E2E) Tests**.
  - **Lý do:** Unit Tests chạy cực nhanh (vài mili-giây), chi phí xây dựng và bảo trì siêu rẻ, dễ định vị chính xác vị trí lỗi dòng code. Càng lên cao (E2E), test càng chạy chậm, đắt đỏ và dễ dính lỗi chập chờn do mạng (Flaky), do đó chỉ nên giữ một lượng nhỏ để kiểm thử các luồng người dùng cốt lõi nhất.

### Câu 12: Phân biệt sự khác nhau giữa Mock và Stub trong kiểm thử tự động?
- **Trả lời:**
  - **Stub:** Tập trung vào **Trạng thái (State)**: Nó chỉ đơn giản trả về dữ liệu mẫu cố định tính toán sẵn khi được gọi (ví dụ stub trả về danh sách 2 sản phẩm mẫu) để hàm cần test có dữ liệu chạy tiếp.
  - **Mock:** Tập trung vào **Hành vi (Behavior)**: Nó được cài đặt sẵn kỳ vọng và sẽ kiểm chứng xem hàm đó có thực sự được gọi hay không, được gọi mấy lần và tham số truyền vào có chính xác không (ví dụ kiểm chứng hàm `orderRepo.save()` bắt buộc phải được gọi đúng 1 lần).

### Câu 13: Quy trình TDD (Test-Driven Development) gồm những bước nào? Lợi ích cốt lõi của TDD là gì?
- **Trả lời:**
  - Gồm 3 bước lặp nhịp nhàng: **RED** (Viết test lỗi trước) ──► **GREEN** (Viết code thực tế tối thiểu để test pass) ──► **REFACTOR** (Dọn dẹp mã sạch sẽ mà không sợ làm hỏng logic).
  - **Lợi ích cốt lõi:** Buộc lập trình viên phải tư duy sâu sắc về thiết kế giao diện hàm (API Design) từ góc nhìn của người sử dụng trước khi viết code; đảm bảo 100% code viết ra đều có test bảo vệ, loại bỏ hoàn toàn tâm lý sợ hãi khi sửa code.

### Câu 14: Đạt 100% Code Coverage có đồng nghĩa với việc phần mềm không còn bug hay không? Tại sao?
- **Trả lời:**
  - **Không!** Code Coverage chỉ đo xem dòng lệnh đó có được thực thi qua trong quá trình chạy test hay không, chứ **không đo được tính đúng đắn của logic nghiệp vụ**.
  - Bạn có thể đạt 100% coverage bằng những bài test không hề có câu lệnh kiểm chứng (`assert/expect`), hoặc bộ test bỏ quên các kịch bản biên quan trọng (nhập chuỗi rỗng, số âm, giá trị vượt ngưỡng tràn bộ nhớ, tấn công bảo mật).

### Câu 15: Nợ kỹ thuật (Technical Debt) là gì và làm thế nào để quản lý nó trong một dự án phần mềm dài hạn?
- **Trả lời:**
  - **Technical Debt** là cái giá vô hình phải trả trong tương lai khi bạn chọn giải pháp lập trình tạm bợ, cẩu thả để hoàn thành nhanh tính năng ở hiện tại. Giống như nợ tài chính, nợ kỹ thuật tích lũy "tiền lãi": hệ thống càng ngày càng chậm, code càng ngày càng khó sửa và tỷ lệ lỗi tăng vọt.
  - **Cách quản lý:**
    1. Áp dụng quy tắc Hướng đạo sinh (Boy Scout Rule): Dọn dẹp mã liên tục trong mỗi Task hằng ngày.
    2. Dành riêng 15 - 20% thời gian của mỗi chu kỳ Sprint phát triển để chuyên tâm trả nợ kỹ thuật và tái cấu trúc (Refactoring).
    3. Thiết lập hệ thống kiểm tra chất lượng tự động (CI/CD với Linter, SonarQube, Automated Tests) để chặn đứng code rác trước khi merge vào nhánh chính.

---

## PHẦN 3: CHECKLIST TỰ ĐÁNH GIÁ NĂNG LỰC (LEVEL 5 COMPETENCY)

Hãy tự kiểm tra xem bạn đã thực sự đạt tư duy của một Kỹ sư Phần mềm chưa:

- [ ] Tôi luôn đặt tên biến và hàm tự giải thích nghĩa, không dùng tên tắt bí hiểm hay Magic Numbers.
- [ ] Tôi viết hàm ngắn gọn dưới 20 dòng và đảm bảo hàm chỉ làm đúng 1 nhiệm vụ duy nhất.
- [ ] Tôi hiểu và giải thích được cả 5 nguyên lý SOLID kèm ví dụ code thực tế.
- [ ] Tôi tự tay áp dụng được các Design Patterns kinh điển: Factory, Builder, Adapter, Strategy, Observer, Repository.
- [ ] Tôi phân biệt được sự khác biệt giữa Monolith, Modular Monolith và Microservices.
- [ ] Tôi hiểu quy tắc phụ thuộc hướng vào trong của Clean Architecture và Ports & Adapters.
- [ ] Tôi tuân thủ Kim tự tháp kiểm thử và luôn viết Unit Test với kỹ thuật Mocking cho logic nghiệp vụ.
- [ ] Tôi hiểu chu trình TDD (Red - Green - Refactor) và biết cách tái cấu trúc code an toàn.
