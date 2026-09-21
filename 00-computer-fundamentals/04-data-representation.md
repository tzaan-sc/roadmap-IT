# 04 - Biểu Diễn Dữ Liệu Trong Máy Tính (Data Representation)

> **Mục tiêu bài học:** Nắm vững cách phần cứng máy tính chỉ với tín hiệu điện 0 và 1 (nhị phân) có thể biểu diễn được số nguyên, số thực dấu phẩy động, văn bản chữ viết tiếng Việt, hình ảnh và âm thanh. Hiểu cội nguồn của các hiện tượng kinh điển như tràn số (Integer Overflow) và sai số dấu phẩy động (`0.1 + 0.2 != 0.3`).

---

## 1. Đơn Vị Đo Lường: Bit, Byte và Tiền Tố Bộ Nhớ

### 1.1. Bit và Byte
- **Bit (Binary digit):** Đơn vị thông tin nhỏ nhất trong máy tính. Chỉ nhận 1 trong 2 trạng thái: `0` (điện áp thấp / tắt) hoặc `1` (điện áp cao / bật).
- **Nibble:** Một nhóm gồm 4 bits (ví dụ: `1011`).
- **Byte:** Một nhóm gồm **8 bits**.
  - Với 8 bits, một byte có thể biểu diễn được $2^8 = 256$ trạng thái khác nhau (từ `00000000` đến `11111111`).
  - Byte là đơn vị địa chỉ hóa nhỏ nhất mà CPU có thể truy cập trực tiếp trong RAM.

### 1.2. Tại sao mua ổ cứng 1TB cắm vào máy tính chỉ thấy ~931 GB?

Sự khác biệt giữa hệ thập phân (SI) của nhà sản xuất phần cứng và hệ nhị phân (IEC) của hệ điều hành:

```text
HỆ THẬP PHÂN - SI (Nhà sản xuất ổ cứng)       HỆ NHỊ PHÂN - IEC (Hệ điều hành Windows)
1 KB = 10^3  = 1,000 Bytes                   1 KiB (Kibibyte) = 2^10 = 1,024 Bytes
1 MB = 10^6  = 1,000,000 Bytes               1 MiB (Mebibyte) = 2^20 = 1,048,576 Bytes
1 GB = 10^9  = 1,000,000,000 Bytes           1 GiB (Gibibyte) = 2^30 = 1,073,741,824 Bytes
1 TB = 10^12 = 1,000,000,000,000 Bytes       1 TiB (Tebibyte) = 2^40 = 1,099,511,627,776 Bytes
```

- Nhà sản xuất bán ổ 1 TB = $1,000,000,000,000$ Bytes.
- Hệ điều hành tính dung lượng theo lũy thừa của 2:
  $$\frac{1,000,000,000,000}{1,024 \times 1,024 \times 1,024} \approx 931.32 \text{ GiB (GB)}$$

---

## 2. Các Hệ Đếm Thường Dùng Trong Khoa Học Máy Tính

| Hệ đếm | Cơ số (Base) | Các ký số sử dụng | Tiền tố trong code | Ví dụ giá trị 42 |
| :--- | :--- | :--- | :--- | :--- |
| **Nhị phân (Binary)** | 2 | `0, 1` | `0b` | `0b00101010` |
| **Bát phân (Octal)** | 8 | `0, 1, 2, 3, 4, 5, 6, 7` | `0o` | `0o52` |
| **Thập phân (Decimal)** | 10 | `0, 1, 2, 3, 4, 5, 6, 7, 8, 9` | Không có | `42` |
| **Thập lục phân (Hexadecimal)** | 16 | `0-9` và `A, B, C, D, E, F` (A=10, F=15) | `0x` | `0x2A` |

### Tại sao Hexadecimal (Hệ 16) lại cực kỳ phổ biến trong lập trình?
Vì **1 ký số Hex tương ứng chính xác với 4 bits (1 nibble)**:
- `0x0` = `0000`
- `0xF` = `1111`
- **1 Byte (8 bits)** được viết gọn thành đúng **2 ký số Hex** (từ `0x00` đến `0xFF`).
- Ví dụ: Địa chỉ ô nhớ RAM `0x7FFEED40`, mã màu web CSS `#FF5733` (Đỏ: FF, Xanh lục: 57, Xanh lam: 33).

---

## 3. Biểu Diễn Số Nguyên (Integers) & Số Bù 2 (Two's Complement)

### 3.1. Số nguyên không dấu (Unsigned Integer)
Tất cả các bits đều dùng để biểu diễn độ lớn:
- Với 8-bit unsigned: biểu diễn từ $0$ đến $2^8 - 1 = 255$.
- Với 32-bit unsigned: biểu diễn từ $0$ đến $2^{32} - 1 \approx 4.29$ tỷ.

### 3.2. Số nguyên có dấu (Signed Integer) - Chuẩn Bù 2 (Two's Complement)
Hầu hết các CPU ngày nay dùng **Bù 2** để lưu số âm vì nó giúp mạch điện ALU dùng chung một bộ cộng cho cả phép cộng và phép trừ:

- **Bit dấu (Sign bit):** Bit có trọng số cao nhất (nằm ngoài cùng bên trái - MSB).
  - Bit `0`: Số dương.
  - Bit `1`: Số âm.
- **Cách tìm dạng biểu diễn Bù 2 của một số âm:**
  1. Viết dạng nhị phân của số dương tương ứng.
  2. Đảo toàn bộ các bit (`0` thành `1`, `1` thành `0` - gọi là Bù 1).
  3. Cộng thêm `1`.

*Ví dụ: Biểu diễn số `-5` trong 8-bit:*
1. Số `+5` = `0000 0101`
2. Đảo bit = `1111 1010`
3. Cộng `1` = `1111 1011` (Đây chính là số `-5`)

### 3.3. Hiện tượng tràn số nguyên (Integer Overflow)
Điều gì xảy ra khi bạn cộng thêm 1 vào giá trị lớn nhất mà kiểu dữ liệu có thể chứa?

```c
// Ví dụ với 8-bit signed (phạm vi từ -128 đến 127):
int8_t x = 127;   // Nhị phân: 0111 1111
x = x + 1;        // Nhị phân biến thành: 1000 0000 => Giá trị trở thành -128!
```

> [!CAUTION]
> **Bài học xương máu trong lịch sử:** Tên lửa Ariane 5 của châu Âu phát nổ năm 1996 chỉ sau 37 giây phóng, gây thiệt hại 370 triệu USD, nguyên nhân cốt lõi là do code chuyển đổi số thực 64-bit sang số nguyên có dấu 16-bit gây hiện tượng tràn số (Integer Overflow)!

---

## 4. Biểu Diễn Số Thực - Chuẩn IEEE 754 & Bí Ẩn `0.1 + 0.2 != 0.3`

Trong máy tính, số thực có phần thập phân được biểu diễn dưới dạng **Dấu phẩy động (Floating-Point)** theo chuẩn **IEEE 754**:

```text
Chuẩn IEEE 754 Single Precision (Float - 32 bits):
┌──────────┬────────────────────┬──────────────────────────────────────┐
│  1 bit   │       8 bits       │               23 bits                │
│ Bit dấu  │ Số mũ (Exponent)   │  Phần định trị (Mantissa / Fraction) │
└──────────┴────────────────────┴──────────────────────────────────────┘
```

Công thức toán học: $\text{Giá trị} = (-1)^{\text{Sign}} \times (1 + \text{Mantissa}) \times 2^{\text{Exponent - Bias}}$

### Tại sao trong JavaScript hay Python: `0.1 + 0.2 == 0.30000000000000004`?
- Giống như trong hệ thập phân, ta không thể biểu diễn chính xác số $\frac{1}{3}$ (nó là số vô hạn tuần hoàn `0.333333...`).
- Trong hệ nhị phân, số $0.1$ ($\frac{1}{10}$) và $0.2$ ($\frac{1}{5}$) là các **số vô hạn tuần hoàn**:
  $$0.1_{10} = 0.00011001100110011..._2$$
- Do số lượng bit của CPU là hữu hạn (32-bit float hoặc 64-bit double), máy tính buộc phải cắt cụt và làm tròn. Khi cộng hai số đã bị làm tròn, sai số tích lũy xuất hiện!

> [!IMPORTANT]
> **Quy tắc làm phần mềm tài chính / ngân hàng / thương mại điện tử:**
> **TUYỆT ĐỐI KHÔNG** dùng kiểu dữ liệu `float` hoặc `double` để lưu số tiền hay tính toán số dư ví. Hãy dùng kiểu `Decimal` (trong C#/Python), `BigDecimal` (trong Java) hoặc quy đổi toàn bộ số tiền ra đơn vị nhỏ nhất (ví dụ lưu đồng xu / cents dưới dạng số nguyên `int` hoặc `long`).

---

## 5. Biểu Diễn Ký Tự & Văn Bản: ASCII vs Unicode vs UTF-8

Máy tính chỉ hiểu số, vì vậy mỗi chữ cái cần có một con số quy ước đại diện (**Character Code**).

```text
KÝ TỰ        ASCII (Hệ 10)       NHỊ PHÂN 1 BYTE (Hex)
 'A'    ───►      65       ───►  0100 0001 (0x41)
 'B'    ───►      66       ───►  0100 0010 (0x42)
 'a'    ───►      97       ───►  0110 0001 (0x61)
 '0'    ───►      48       ───►  0011 0000 (0x30)
```

### 5.1. Bảng mã ASCII (1963)
- Dùng 7 bits để biểu diễn 128 ký tự (chữ cái tiếng Anh A-Z, a-z, số 0-9, dấu câu và các ký tự điều khiển như `\n`, `\t`).
- **Hạn chế:** Không thể biểu diễn các ngôn ngữ khác như tiếng Việt, tiếng Trung, Nhật, Hàn, tiếng Ả Rập hay Emoji.

### 5.2. Unicode (Tiêu chuẩn quốc tế)
- Unicode không phải là định dạng file, Unicode là một **bộ từ điển khổng lồ** gán cho mỗi ký tự trên thế giới một con số định danh duy nhất gọi là **Code Point** (ký hiệu dạng `U+XXXX`).
- Ví dụ:
  - Chữ `A` là `U+0041`
  - Chữ `ệ` tiếng Việt là `U+1EC7`
  - Emoji cười 😂 là `U+1F602`

### 5.3. UTF-8 - Đỉnh Cao Của Thiết Kế Mã Hóa
Làm sao để lưu Code Point Unicode vào các byte trong ổ cứng và truyền qua mạng Internet? Chuẩn **UTF-8** ra đời và hiện chiếm hơn 98% toàn bộ trang web trên thế giới nhờ các ưu điểm vượt trội:

- **Độ dài biến thiên (Variable-length encoding):**
  - Ký tự tiếng Anh chuẩn ASCII: Dùng đúng **1 Byte** (tương thích 100% với mã ASCII cũ).
  - Ký tự tiếng Việt, Latin mở rộng, Hy Lạp: Dùng **2 đến 3 Bytes**.
  - Ký tự chữ tượng hình (Hán tự, Kanji) và Emoji: Dùng **3 đến 4 Bytes**.
- Tiết kiệm dung lượng hơn rất nhiều so với UTF-16 hay UTF-32 (luôn bắt buộc dùng 2 hoặc 4 byte cho mọi ký tự kể cả chữ tiếng Anh).

---

## 6. Thứ Tự Lưu Trữ Byte Trong Bộ Nhớ (Endianness)

Khi một biến số nguyên 32-bit có giá trị chiếm 4 bytes (ví dụ `0x12345678`), nó sẽ được xếp vào 4 ô nhớ RAM liên tiếp theo thứ tự nào?

```text
Giả sử lưu giá trị 0x12345678 vào ô nhớ 0x00, 0x01, 0x02, 0x03:

LITTLE-ENDIAN (Byte thấp lưu ở địa chỉ thấp) - Phổ biến nhất: Intel x86, AMD, ARM
Địa chỉ:   0x00     0x01     0x02     0x03
Dữ liệu:  [ 78 ]   [ 56 ]   [ 34 ]   [ 12 ]
          (LSB)                       (MSB)

BIG-ENDIAN (Byte cao lưu ở địa chỉ thấp) - Đọc thuận mắt, chuẩn mạng Internet (Network Byte Order)
Địa chỉ:   0x00     0x01     0x02     0x03
Dữ liệu:  [ 12 ]   [ 34 ]   [ 56 ]   [ 78 ]
          (MSB)                       (LSB)
```

> [!TIP]
> Khi lập trình mạng (Socket Programming), dữ liệu gửi đi trên đường truyền Internet luôn quy ước theo chuẩn **Big-Endian** (Network Byte Order). Lập trình viên thường dùng các hàm như `htons()` (Host to Network Short) hoặc `ntohl()` (Network to Host Long) để hoán đổi đúng thứ tự byte giữa máy khách và mạng.

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **1 Byte = 8 Bits = 2 ký số Hex.**
2. **Hexadecimal** là ngôn ngữ ngắn gọn để con người đọc các chuỗi nhị phân, địa chỉ bộ nhớ và mã màu.
3. Số âm được biểu diễn bằng phương pháp **Bù 2** (Đảo bit + 1).
4. Cẩn thận với **Integer Overflow** khi giá trị tính toán vượt khỏi biên của kiểu dữ liệu.
5. Số thực dấu phẩy động **luôn có sai số làm tròn**; không bao giờ dùng `float/double` cho bài toán tài chính tiền tệ.
6. **Unicode** định nghĩa ký tự; **UTF-8** là chuẩn nén lưu trữ biến thiên 1-4 byte phổ biến nhất thế giới.
