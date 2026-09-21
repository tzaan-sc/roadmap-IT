# 04 - Chiến Lược Kiểm Thử Phần Mềm & TDD (Testing Strategies & TDD)

> *"Mã nguồn mà không có kiểm thử tự động không phải là mã nguồn sạch. Nó là mã nguồn di sản (Legacy Code) ngay từ giây phút bạn vừa viết xong."* — **Michael Feathers**

---

## 1. Tại Sao Kỹ Sư Phần Mềm Bắt Buộc Phải Viết Automated Tests?

Trong các công ty phần mềm chuyên nghiệp:
- **Kiểm thử thủ công (Manual Testing):** Bạn click chuột 20 bước để test chức năng mua hàng. Mỗi lần sửa code lại phải click lại 20 bước đó. Tốn thời gian, dễ bỏ sót lỗi và không thể mở rộng.
- **Kiểm thử tự động (Automated Testing):** Bạn viết một đoạn mã script để máy tính tự động kiểm tra hàng ngàn kịch bản chỉ trong **vài giây**.
- **Tấm lưới bảo hiểm (Safety Net):** Cho phép bạn tự tin tái cấu trúc code (Refactoring) vào bất kỳ ngày thứ Sáu nào mà không sợ làm sập hệ thống.

---

## 2. Kim Tự Tháp Kiểm Thử (The Testing Pyramid)

Mô hình kiểm thử chuẩn mực của ngành công nghệ phần mềm (do Mike Cohn đề xuất):

```text
                           KIM TỰ THÁP KIỂM THỬ
                                   ▲
                                  / \
                                 /   \
                                / E2E \          ◄── 10% (End-to-End Tests)
                               /───────\              Mô phỏng người dùng click thật
                              /         \             (Chậm nhất, Đắt đỏ nhất)
                             /INTEGRATION\       ◄── 20% (Integration Tests)
                            /─────────────\           Kiểm tra các tầng phối hợp
                           /     UNIT      \     ◄── 70% (Unit Tests)
                          /      TESTS      \         Kiểm tra từng hàm độc lập
                         /───────────────────\        (Siêu nhanh, Rẻ nhất!)
```

### 2.1. Unit Tests (Kiểm thử đơn vị - 70%)
- Kiểm tra một đơn vị mã nguồn nhỏ nhất (một hàm, một phương thức của Class) trong điều kiện hoàn toàn cô lập.
- Thời gian chạy: **vài mili-giây**.
- Không gọi Database thật, không gọi mạng Internet (tất cả các thành phần phụ thuộc đều được làm giả bằng Mocking).

### 2.2. Integration Tests (Kiểm thử tích hợp - 20%)
- Kiểm tra xem hai hoặc nhiều thành phần có phối hợp ăn ý với nhau hay không.
- Ví dụ: Kiểm tra tầng `UserService` có thực sự gọi xuống CSDL PostgreSQL (dùng Docker test container) để lưu và đọc dữ liệu chính xác hay không.

### 2.3. End-to-End Tests (Kiểm thử đầu-cuối - 10%)
- Bật một trình duyệt Chromium thật (dùng **Playwright** hoặc **Cypress**), tự động gõ tài khoản, click nút "Đăng nhập", thêm hàng vào giỏ và kiểm tra màn hình thanh toán.
- Rất gần với trải nghiệm thực tế, nhưng chạy chậm (mất vài phút) và dễ bị lỗi do độ trễ mạng (Flaky tests).

---

## 3. Nghệ Thuật Làm Giả Đối Tượng (Test Doubles / Mocking)

Khi viết Unit Test cho một hàm nghiệp vụ, làm sao để hàm chạy được mà không cần kết nối tới Database thật hay gọi API thanh toán trừ tiền thẻ tín dụng thật? Ta sử dụng **Test Doubles**:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        5 DẠNG TEST DOUBLES KINH ĐIỂN                   │
└───────┬──────────────────┬───────────────────┬──────────────────┬──────┘
        │                  │                   │                  │
        ▼                  ▼                   ▼                  ▼
     1. DUMMY           2. STUB             3. SPY             4. MOCK
  Đối tượng bù bù     Trả về dữ liệu      Ghi lại lịch sử    Được cài đặt sẵn
  cho đủ tham số,     cố định tính sẵn    gọi hàm: đã gọi    kỳ vọng: BẮT BUỘC
  không dùng tới.     (Hardcoded data).   mấy lần, param gì? phải được gọi đúng!
```

- **Fake:** Là một bản cài đặt thu nhỏ có logic thật nhưng chạy đơn giản và nhẹ (ví dụ: dùng một mảng trong RAM `InMemoryUserRepository` thay cho CSDL PostgreSQL).

---

## 4. Quy Trình Phát Triển Hướng Kiểm Thử (TDD - Test-Driven Development)

TDD đảo ngược hoàn toàn quy trình lập trình truyền thống: **Viết Test trước khi viết Code!**

```text
                  CHU TRÌNH TDD: RED ──► GREEN ──► REFACTOR
┌────────────────────────────────────────────────────────────────────────┐
│ 1. RED (Đỏ):                                                           │
│ Viết một bài Unit Test mô tả hành vi bạn mong muốn.                    │
│ Chạy test: BÀI TEST CHẮC CHẮN THẤT BẠI (vì code thực tế chưa có)!      │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 2. GREEN (Xanh):                                                       │
│ Viết lượng code THỰC TẾ TỐI THIỂU NHẤT CÓ THỂ để bài test vượt qua.   │
│ (Thậm chí code xấu, hardcode tạm giá trị cũng được, miễn là Pass!)     │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 3. REFACTOR (Tái cấu trúc):                                            │
│ Dọn dẹp mã nguồn: Áp dụng Clean Code, xóa trùng lặp, áp dụng SOLID.    │
│ Tự tin 100% vì hệ thống Test xanh sẽ báo ngay nếu bạn lỡ làm hỏng code!│
└────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Code Mẫu Thực Chiến Với Vitest / Jest

Giả sử ta cần viết hàm tính chiết khấu đơn hàng:
- Đơn dưới 100$: Không giảm giá (0%).
- Đơn từ 100$ đến 500$: Giảm giá 10%.
- Đơn trên 500$: Giảm giá 20%.

### 5.1. Viết Unit Test trước (TDD - Bước RED):
File `discount.test.ts`:
```typescript
import { describe, it, expect } from 'vitest';
import { calculateDiscount } from './discount';

describe('calculateDiscount()', () => {
  it('không giảm giá cho đơn hàng dưới 100$', () => {
    const result = calculateDiscount(80);
    expect(result).toBe(0);
  });

  it('giảm 10% cho đơn hàng từ 100$ đến 500$', () => {
    const result = calculateDiscount(200);
    expect(result).toBe(20); // 200 * 10% = 20$
  });

  it('giảm 20% cho đơn hàng trên 500$', () => {
    const result = calculateDiscount(1000);
    expect(result).toBe(200); // 1000 * 20% = 200$
  });

  it('ném ra ngoại lệ nếu số tiền âm', () => {
    expect(() => calculateDiscount(-50)).toThrowError('Số tiền không hợp lệ');
  });
});
```

### 5.2. Viết Code thực tế để Test vượt qua (Bước GREEN):
File `discount.ts`:
```typescript
export function calculateDiscount(amount: number): number {
  if (amount < 0) {
    throw new Error('Số tiền không hợp lệ');
  }
  if (amount >= 500) {
    return amount * 0.2;
  }
  if (amount >= 100) {
    return amount * 0.1;
  }
  return 0;
}
```

---

## 6. Độ Bao Phủ Kiểm Thử (Code Coverage) Là Gì?

**Code Coverage** là tỷ lệ phần trăm các dòng code được chạy qua trong quá trình thực thi bộ kiểm thử:
- **Ngưỡng vàng trong ngành:** **75% - 85%**.
- **Cảnh báo ngụy biện 100% Coverage:** Đạt 100% độ bao phủ dòng code không có nghĩa là phần mềm không có bug! Nó chỉ chứng minh là code đã được chạy qua, nhưng có thể bạn chưa kiểm tra các kịch bản biên (**Edge Cases** như truyền `null`, số cực lớn, chuỗi rỗng hay dữ liệu độc hại).

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. Tuân thủ **Kim tự tháp kiểm thử**: Đầu tư 70% công sức vào **Unit Tests**, 20% cho **Integration Tests** và 10% cho **E2E Tests**.
2. Sử dụng **Mocking / Stubbing** để cô lập hoàn toàn Unit Test khỏi Database và mạng Internet.
3. Làm chủ chu trình TDD: **Red** (Viết test lỗi) ──► **Green** (Viết code tối thiểu để pass) ──► **Refactor** (Dọn sạch mã).
4. Viết các bài test theo mẫu cấu trúc **AAA (Arrange - Act - Assert)**: Chuẩn bị dữ liệu ──► Thực hiện hành động ──► Kiểm tra kết quả kỳ vọng.
