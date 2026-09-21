# 06 - Bài Tập Thực Hành & Thử Thách Thuật Toán (Practice Problems & Challenges)

> **Mục tiêu bài học:** Vận dụng toàn bộ kiến thức về cấu trúc dữ liệu, giải thuật Big-O, hướng đối tượng (OOP) và xử lý ngoại lệ vào việc giải quyết 5 bài toán kinh điển trong phỏng vấn kỹ thuật của các công ty công nghệ (Google, Meta, Amazon, Shopee, VNG).

---

## BÀI 1: Two Sum — Tối Ưu Hóa Bằng Bảng Băm (Hash Map)

> **Nguồn gốc:** LeetCode #1 (Bài toán phỏng vấn phổ biến nhất lịch sử lập trình)

### Đề bài:
Cho một mảng số nguyên `nums` và một số nguyên `target`. Hãy tìm **chỉ số (indices)** của hai số trong mảng có tổng bằng đúng `target`. Giả định luôn có duy nhất một đáp án và bạn không được dùng cùng một phần tử hai lần.

- **Ví dụ:** `nums = [2, 7, 11, 15]`, `target = 9`
- **Kết quả:** `[0, 1]` (vì `nums[0] + nums[1] = 2 + 7 = 9`).

---

### Cách 1: Vét cạn ngây thơ (Brute-Force) — $O(N^2)$
Dùng 2 vòng lặp lồng nhau duyệt qua mọi cặp số có thể:
- **Thời gian:** $O(N^2)$ (Quá chậm khi mảng có 100,000 phần tử).
- **Bộ nhớ phụ:** $O(1)$.

---

### Cách 2: Tối ưu với Bảng băm (Hash Map) — $O(N)$
Thay vì tìm số thứ hai bằng vòng lặp, khi duyệt qua mỗi số $x$, ta tự hỏi: *"Số bù cần tìm là bao nhiêu?"*
$$\text{Complement} = \text{target} - x$$
Ta tra cứu trong Hash Map xem `complement` đã từng xuất hiện trước đó chưa:
- Nếu **ĐÃ CÓ**: Trả về ngay cặp index tương ứng (chỉ mất $O(1)$ tra cứu)!
- Nếu **CHƯA CÓ**: Lưu giá trị hiện tại và index của nó vào Hash Map rồi đi tiếp.

```text
nums = [ 2, 7, 11, 15 ], target = 9
Duyệt số đầu tiên: x = 2
  - Số cần tìm: 9 - 2 = 7.
  - Map hiện tại: {}. Chưa thấy 7.
  - Lưu vào Map: { 2: 0 } (Giá trị 2 nằm ở index 0).

Duyệt số thứ hai: x = 7
  - Số cần tìm: 9 - 7 = 2.
  - Tra Map: ĐÃ THẤY số 2 tại index 0!
  - Kết quả ngay lập tức: [0, 1]!
```

#### Code mẫu Python:
```python
def two_sum(nums, target):
    seen = {} # Hash Map: { số: index }
    
    for current_index, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], current_index]
        # Lưu vào map
        seen[num] = current_index
        
    return []

# Chạy thử
print(two_sum([2, 7, 11, 15], 9)) # [0, 1]
```

#### Code mẫu JavaScript:
```javascript
function twoSum(nums, target) {
  const seen = new Map(); // Map lưu { giá trị => index }

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (seen.has(complement)) {
      return [seen.get(complement), i];
    }
    seen.set(nums[i], i);
  }
  return [];
}

console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

- **Độ phức tạp thời gian:** $O(N)$ (Chỉ duyệt qua mảng đúng 1 lần).
- **Độ phức tạp không gian:** $O(N)$ (Bộ nhớ lưu Hash Map).

---

## BÀI 2: Valid Parentheses — Kiểm Tra Dấu Ngoặc Hợp Lệ (Stack)

> **Nguồn gốc:** LeetCode #20 (Thường gặp khi viết Parser hoặc Compiler)

### Đề bài:
Cho một chuỗi `s` chỉ chứa các ký tự `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`. Hãy kiểm tra xem chuỗi đó có hợp lệ hay không.
Một chuỗi hợp lệ khi:
1. Các ngoặc mở phải được đóng bằng cùng loại ngoặc.
2. Các ngoặc mở phải được đóng theo đúng thứ tự (cái mở sau phải đóng trước).

- **Ví dụ hợp lệ:** `"()"` , `"()[]{}"` , `"{[()]}"`
- **Ví dụ không hợp lệ:** `"(]"` , `"([)]"`

---

### Phân tích tư duy:
Dấu ngoặc mở xuất hiện sau cùng sẽ là dấu ngoặc đầu tiên cần phải được đóng. Cấu trúc vào sau ra trước (LIFO) hoàn hảo cho bài này chính là **Ngăn xếp (Stack)**!

```text
Kiểm tra chuỗi: "{ [ ( ) ] }"
1. Thấy '{' ──► Push vào Stack: [ '{' ]
2. Thấy '[' ──► Push vào Stack: [ '{', '[' ]
3. Thấy '(' ──► Push vào Stack: [ '{', '[', '(' ]
4. Thấy ')' ──► Pop đỉnh Stack ra so sánh: '(' khớp với ')'! => Stack còn: [ '{', '[' ]
5. Thấy ']' ──► Pop đỉnh Stack ra: '[' khớp với ']'! => Stack còn: [ '{' ]
6. Thấy '}' ──► Pop đỉnh Stack ra: '{' khớp với '}'! => Stack rỗng!
Kết luận: Hợp lệ!
```

#### Code mẫu Python:
```python
def is_valid_parentheses(s: str) -> bool:
    stack = []
    # Từ điển ánh xạ ngoặc đóng sang ngoặc mở tương ứng
    matching = {')': '(', '}': '{', ']': '['}
    
    for char in s:
        if char in matching:
            # Là ngoặc đóng: Lấy phần tử đỉnh stack ra kiểm tra
            top_element = stack.pop() if stack else '#'
            if matching[char] != top_element:
                return False
        else:
            # Là ngoặc mở: Đẩy vào stack
            stack.append(char)
            
    # Hợp lệ khi và chỉ khi stack đã được giải phóng sạch sẽ
    return len(stack) == 0

print(is_valid_parentheses("{[()]}")) # True
print(is_valid_parentheses("([)]"))   # False
```

- **Độ phức tạp:** Thời gian $O(N)$, Bộ nhớ $O(N)$.

---

## BÀI 3: Binary Search — Tìm Kiếm Nhị Phân Chuẩn Xác

> **Nguồn gốc:** LeetCode #704

### Đề bài:
Cho một mảng số nguyên `nums` đã được sắp xếp tăng dần và một giá trị `target`. Hãy viết hàm tìm kiếm `target` trong mảng với độ phức tạp thời gian **$O(\log N)$**. Nếu tìm thấy trả về index, ngược lại trả về `-1`.

#### Code mẫu JavaScript:
```javascript
function binarySearch(nums, target) {
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    // Tránh lỗi tràn số nguyên (Integer Overflow) thay vì dùng (left + right) / 2
    const mid = Math.floor(left + (right - left) / 2);

    if (nums[mid] === target) {
      return mid; // Tìm thấy mục tiêu!
    } else if (nums[mid] < target) {
      left = mid + 1; // Mục tiêu nằm ở nửa bên phải
    } else {
      right = mid - 1; // Mục tiêu nằm ở nửa bên trái
    }
  }

  return -1; // Không tồn tại
}

console.log(binarySearch([-1, 0, 3, 5, 9, 12], 9)); // In ra: 4
```

---

## BÀI 4: Reverse Linked List — Đảo Ngược Danh Sách Liên Kết

> **Nguồn gốc:** LeetCode #206

### Đề bài:
Cho con trỏ `head` của một danh sách liên kết đơn, hãy đảo ngược danh sách và trả về con trỏ `head` mới.

```text
BAN ĐẦU:      1 ──► 2 ──► 3 ──► 4 ──► NULL
SAU KHI ĐẢO:  NULL ◄── 1 ◄── 2 ◄── 3 ◄── 4 (HEAD MỚI)
```

### Phân tích tư duy (Kỹ thuật 3 con trỏ):
Để đảo ngược một mắt xích `curr -> next` thành `curr -> prev`, ta cần 3 con trỏ:
1. `prev`: Con trỏ trỏ tới node phía trước (khởi tạo bằng `None`/`null`).
2. `curr`: Con trỏ đang đứng tại node hiện tại.
3. `next_node`: Lưu tạm node kế tiếp trước khi ta bẻ gãy liên kết.

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_linked_list(head):
    prev = None
    curr = head
    
    while curr is not None:
        next_node = curr.next  # 1. Lưu tạm node kế tiếp
        curr.next = prev       # 2. Đảo ngược mũi tên chỉ về node trước đó
        prev = curr            # 3. Dịch prev tiến lên một bước
        curr = next_node       # 4. Dịch curr tiến lên một bước
        
    return prev  # prev lúc này đang ở đỉnh mới (node cuối ban đầu)
```

---

## BÀI 5: Thiết Kế OOP Thực Chiến — Hệ Thống Quản Lý Thư Viện Mini

### Đề bài:
Xây dựng một hệ thống thư viện hướng đối tượng có đầy đủ:
- **Class `Book`**: Thuộc tính mã sách `isbn`, tiêu đề `title`, tác giả `author`, trạng thái mượn `is_borrowed`.
- **Class `User`**: Tên độc giả `name`, danh sách sách đang mượn `borrowed_books`.
- **Class `Library`**: Quản lý danh mục sách, cung cấp phương thức `borrow_book(user, isbn)` và `return_book(user, isbn)`.
- Áp dụng đầy đủ: **Encapsulation**, **Custom Exceptions** khi cố mượn sách đã có người mượn hoặc sách không tồn tại.

#### Code mẫu Python hoàn chỉnh:
```python
class BookNotFoundError(Exception):
    pass

class BookAlreadyBorrowedError(Exception):
    pass

class Book:
    def __init__(self, isbn: str, title: str, author: str):
        self.isbn = isbn
        self.title = title
        self.author = author
        self.__is_borrowed = False # Đóng gói thuộc tính Private

    def is_borrowed(self) -> bool:
        return self.__is_borrowed

    def mark_as_borrowed(self):
        self.__is_borrowed = True

    def mark_as_returned(self):
        self.__is_borrowed = False

    def __repr__(self):
        status = "Đã mượn" if self.__is_borrowed else "Có sẵn"
        return f"[{self.isbn}] {self.title} - {self.author} ({status})"


class User:
    def __init__(self, user_id: str, name: str):
        self.user_id = user_id
        self.name = name
        self.borrowed_books = []

    def add_book(self, book: Book):
        self.borrowed_books.append(book)

    def remove_book(self, isbn: str):
        self.borrowed_books = [b for b in self.borrowed_books if b.isbn != isbn]


class Library:
    def __init__(self, name: str):
        self.name = name
        self.books = {} # Hash Map lưu { isbn: Book }

    def add_book(self, book: Book):
        self.books[book.isbn] = book

    def borrow_book(self, user: User, isbn: str):
        if isbn not in self.books:
            raise BookNotFoundError(f"Không tìm thấy sách với mã ISBN: {isbn}")
        
        book = self.books[isbn]
        if book.is_borrowed():
            raise BookAlreadyBorrowedError(f"Sách '{book.title}' hiện đã được người khác mượn!")
            
        book.mark_as_borrowed()
        user.add_book(book)
        print(f"✅ Độc giả '{user.name}' đã mượn thành công cuốn sách: {book.title}")

    def return_book(self, user: User, isbn: str):
        if isbn not in self.books:
            raise BookNotFoundError("Mã sách không hợp lệ!")
            
        book = self.books[isbn]
        book.mark_as_returned()
        user.remove_book(isbn)
        print(f"🔄 Độc giả '{user.name}' đã trả thành công cuốn sách: {book.title}")


# KỊCH BẢN KIỂM THỬ THỰC TẾ
if __name__ == "__main__":
    lib = Library("Thư Viện Công Nghệ")
    
    # 1. Thêm sách
    b1 = Book("B01", "Clean Code", "Robert C. Martin")
    b2 = Book("B02", "Design Patterns", "Gang of Four")
    lib.add_book(b1)
    lib.add_book(b2)
    
    # 2. Tạo độc giả
    john = User("U01", "John Doe")
    
    # 3. Mượn sách thành công
    lib.borrow_book(john, "B01")
    
    # 4. Thử mượn lại cuốn sách đã mượn (Bắt ngoại lệ)
    try:
        alice = User("U02", "Alice")
        lib.borrow_book(alice, "B01")
    except BookAlreadyBorrowedError as e:
        print(f"❌ Xảy ra ngoại lệ như mong đợi: {e}")
        
    # 5. Trả sách
    lib.return_book(john, "B01")
```

---

## 6. Checklist Tự Đánh Giá Năng Lực (Level 1 Competency)

Hãy tích chọn những mục bạn đã tự tin làm chủ:

- [ ] Tôi hiểu rõ sự khác biệt giữa Primitive Type và Reference Type trong bộ nhớ (Stack vs Heap).
- [ ] Tôi biết cách viết Pure Functions và hiểu cơ chế Closures.
- [ ] Tôi có thể phân biệt và giải thích khi nào nên dùng Mảng (Array), Danh sách liên kết (Linked List), Ngăn xếp (Stack), Hàng đợi (Queue) và Bảng băm (Hash Map).
- [ ] Tôi có thể phân tích độ phức tạp thời gian và không gian Big-O của một đoạn code.
- [ ] Tôi tự viết được thuật toán Tìm kiếm nhị phân (Binary Search) không bị tràn số.
- [ ] Tôi hiểu cơ chế chia để trị của Merge Sort và Quick Sort.
- [ ] Tôi giải thích được 4 tính chất OOP (Đóng gói, Trừu tượng, Kế thừa, Đa hình) bằng ví dụ thực tế.
- [ ] Tôi biết cách dùng `try-catch-finally` và tự tạo Custom Exception khi cần.
- [ ] Tôi biết cách đặt Breakpoint trong VS Code để theo dõi biến và gỡ lỗi.
- [ ] Tôi đã tự tay giải thành công 3/5 bài toán thuật toán trong bài học này.
