# 03 - Giải Thuật & Độ Phức Tạp Thuật Toán (Algorithms & Big-O)

> **Mục tiêu bài học:** Nắm vững thước đo hiệu năng thuật toán (Big-O Notation), làm chủ các thuật toán kinh điển về tìm kiếm (Search), sắp xếp (Sort), đệ quy (Recursion) và các kỹ thuật giải quyết vấn đề đỉnh cao (Two Pointers, Sliding Window, Dynamic Programming).

---

## 1. Độ Phức Tạp Thuật Toán & Ký Pháp Big-O (Big-O Notation)

Khi đánh giá một giải thuật, ta **không đo bằng giây hay mili-giây** (bởi vì thời gian chạy phụ thuộc vào máy tính mạnh hay yếu). Thay vào đó, ta đo **tốc độ tăng trưởng của số phép tính (hoặc dung lượng RAM)** khi kích thước dữ liệu đầu vào ($N$) tăng dần lên vô cùng.

```text
               THANG ĐO ĐỘ PHỨC TẠP THỜI GIAN (TIME COMPLEXITY)
  Số phép tính
      ▲
      │                                                O(N!)  O(2^N)
      │                                                  │      │
      │                                                  │      │   O(N^2)
      │                                                  │      │     │
      │                                                  │      │     │
      │                                                  │      │     │     O(N log N)
      │                                                  │      │     │      /
      │                                                  │      │     │     /   O(N)
      │                                                  │      │     │    /   /
      │                                                  │      │     │   /   /
      │                                                  │      │     │  /   /
      │──────────────────────────────────────────────────┼──────┼─────┼─/───/────── O(log N)
      │──────────────────────────────────────────────────┴──────┴─────┴/───/─────── O(1)
      └──────────────────────────────────────────────────────────────────────────► Kích thước N
          (Rất tốt) ◄─────────────────────────────────────────────► (Rất tệ/Chết máy)
```

### 1.1. Bảng ý nghĩa các cấp độ Big-O

| Ký pháp Big-O | Tên gọi | Số phép tính khi $N = 1,000,000$ | Ví dụ thực tế |
| :--- | :--- | :--- | :--- |
| **$O(1)$** | Thời gian hằng số (Constant) | 1 phép tính (Tức thì) | Đọc phần tử mảng theo index `arr[5]`, tra cứu Hash Map |
| **$O(\log N)$**| Thời gian logarit (Logarithmic) | ~20 phép tính (Siêu tốc) | Tìm kiếm nhị phân (Binary Search), tra cứu cây BST cân bằng |
| **$O(N)$** | Thời gian tuyến tính (Linear) | 1,000,000 phép tính | Một vòng lặp `for` duyệt qua từng phần tử của mảng |
| **$O(N \log N)$**| Tuyến tính - Logarit | ~20,000,000 phép tính | Các thuật toán sắp xếp tối ưu (Merge Sort, Quick Sort) |
| **$O(N^2)$** | Thời gian bình phương (Quadratic)| 1,000,000,000,000 (1 nghìn tỷ!) | Hai vòng lặp lồng nhau (Nested loops), Bubble Sort |
| **$O(2^N)$** | Thời gian số mũ (Exponential) | Vượt quá số nguyên tử trong vũ trụ! | Vét cạn (Brute-force) tập con, Fibonacci đệ quy ngây thơ |
| **$O(N!)$** | Thời gian giai thừa (Factorial) | Treo máy hoàn toàn | Bài toán Người du lịch (Traveling Salesman) vét cạn hoán vị |

### 1.2. Ba quy tắc vàng khi tính Big-O
1. **Bỏ qua hằng số:** $O(2N) \rightarrow O(N)$; $O(500) \rightarrow O(1)$.
2. **Chỉ giữ lại số hạng có bậc tăng trưởng cao nhất:**
   $$O(N^2 + 100N + 5000) \rightarrow O(N^2)$$
3. **Các vòng lặp nối tiếp thì CỘNG, lồng nhau thì NHÂN:**
   ```javascript
   // 2 vòng lặp độc lập => O(N + M)
   for (let i = 0; i < N; i++) { ... }
   for (let j = 0; j < M; j++) { ... }

   // 2 vòng lặp lồng nhau => O(N * N) = O(N^2)
   for (let i = 0; i < N; i++) {
     for (let j = 0; j < N; j++) { ... }
   }
   ```

---

## 2. Thuật Toán Tìm Kiếm (Searching Algorithms)

### 2.1. Tìm kiếm tuyến tính (Linear Search) — $O(N)$
Duyệt lần lượt từ phần tử đầu tiên đến phần tử cuối cùng cho tới khi thấy giá trị cần tìm.
- Áp dụng được cho **mọi loại mảng**, kể cả mảng chưa sắp xếp.

### 2.2. Tìm kiếm nhị phân (Binary Search) — $O(\log N)$
> [!IMPORTANT]
> **Điều kiện tiên quyết:** Dữ liệu **BẮT BUỘC ĐÃ ĐƯỢC SẮP XẾP** (tăng dần hoặc giảm dần).

```text
Mảng đã sắp xếp: [ 2, 5, 8, 12, 16, 23, 38, 56, 72, 91 ]  (Cần tìm x = 23)

Bước 1: left = 0, right = 9 => mid = 4 (Giá trị: 16)
        Vì 23 > 16 => Chắc chắn 23 nằm ở nửa bên phải!
        Loại bỏ hoàn toàn nửa bên trái [2, 5, 8, 12].
        left = mid + 1 = 5

Bước 2: left = 5, right = 9 => mid = 7 (Giá trị: 56)
        Vì 23 < 56 => 23 nằm ở nửa bên trái của vùng còn lại.
        right = mid - 1 = 6

Bước 3: left = 5, right = 6 => mid = 5 (Giá trị: 23)
        Tìm thấy 23 tại index 5 chỉ sau 3 lần so sánh! (Trong khi Linear Search mất 6 lần).
```

#### Code mẫu Python chuẩn tránh tràn số:
```python
def binary_search(arr, target):
    left = 0
    right = len(arr) - 1
    
    while left <= right:
        # Cách tính mid an toàn tránh integer overflow trong C++/Java:
        mid = left + (right - left) // 2
        
        if arr[mid] == target:
            return mid  # Tìm thấy, trả về vị trí index
        elif arr[mid] < target:
            left = mid + 1   # Thu hẹp nửa bên phải
        else:
            right = mid - 1  # Thu hẹp nửa bên trái
            
    return -1  # Không tìm thấy
```

---

## 3. Thuật Toán Sắp Xếp (Sorting Algorithms)

### 3.1. Các thuật toán cơ bản ($O(N^2)$) - Dễ hiểu nhưng chậm
- **Bubble Sort (Nổi bọt):** Liên tục so sánh 2 phần tử kề nhau, nếu sai thứ tự thì hoán đổi (swap). Phần tử lớn nhất sẽ dần dần "nổi" về cuối mảng như bọt khí.
- **Selection Sort (Chọn):** Tìm phần tử nhỏ nhất trong mảng chưa sắp xếp rồi đưa về vị trí đầu tiên. Lặp lại với các vị trí tiếp theo.
- **Insertion Sort (Chèn):** Giống như cách sắp xếp bài tây trên tay: lấy từng quân bài mới và chèn vào đúng vị trí của các quân bài đã sắp trước đó.

### 3.2. Các thuật toán nâng cao ($O(N \log N)$) - Chuẩn công nghiệp

#### A. Merge Sort (Sắp xếp trộn) - Tư tưởng "Chia để trị" (Divide and Conquer)
1. **Chia (Divide):** Cắt đôi mảng ra làm 2 nửa liên tục cho đến khi mỗi mảng con chỉ còn đúng 1 phần tử (mảng 1 phần tử coi như đã sắp xếp).
2. **Trị (Conquer):** Ghép (Merge) 2 mảng con đã sắp xếp lại thành 1 mảng lớn hơn có thứ tự.

```text
               [ 38, 27, 43, 3, 9, 82, 10 ]
                       /            \
             [ 38, 27, 43, 3 ]     [ 9, 82, 10 ]
               /         \            /       \
           [ 38, 27 ]  [ 43, 3 ]   [ 9, 82 ]  [ 10 ]
             /    \      /    \      /    \      │
           [38]  [27]  [43]   [3]   [9]  [82]   [10]   (Đã chia nhỏ nhất)
             \    /      \    /      \    /      │
           [ 27, 38 ]  [ 3, 43 ]   [ 9, 82 ]    [10]   (Bắt đầu Merge)
               \         /            \         /
             [ 3, 27, 38, 43 ]     [ 9, 10, 82 ]
                       \            /
               [ 3, 9, 10, 27, 38, 43, 82 ]
```

- **Ưu điểm:** Luôn luôn đảm bảo thời gian $O(N \log N)$ trong mọi trường hợp (kể cả trường hợp xấu nhất). Là thuật toán ổn định (**Stable Sort** - giữ nguyên thứ tự tương đối của các phần tử bằng nhau).
- **Nhược điểm:** Cần tốn thêm một vùng nhớ phụ **$O(N)$** trong RAM để làm mảng tạm khi ghép.

#### B. Quick Sort (Sắp xếp nhanh)
- Chọn một phần tử làm chốt (**Pivot**).
- Phân hoạch (Partition): Đưa toàn bộ các số nhỏ hơn Pivot sang bên trái, các số lớn hơn Pivot sang bên phải.
- Đệ quy áp dụng cho 2 nửa trái và phải.
- **Ưu điểm:** Sắp xếp trực tiếp tại chỗ (**In-place**, Space complexity chỉ là $O(\log N)$ trên Call Stack), cực kỳ thân thiện với bộ nhớ đệm CPU Cache.
- **Nhược điểm:** Nếu chọn Pivot kém (ví dụ mảng đã sắp xếp mà chọn trúng phần tử đầu tiên), thuật toán có thể thoái hóa về $O(N^2)$.

---

## 4. Đệ Quy (Recursion) - Nghệ Thuật Tự Gọi Chính Mình

Một hàm được gọi là đệ quy khi bên trong thân hàm có câu lệnh gọi lại chính nó để giải một bài toán con tương tự có quy mô nhỏ hơn.

```text
ĐỆ QUY TÍNH 4! (Giai thừa của 4)
factorial(4) = 4 * factorial(3)
                   │
                   ▼
                   factorial(3) = 3 * factorial(2)
                                      │
                                      ▼
                                      factorial(2) = 2 * factorial(1)
                                                         │
                                                         ▼
                                                         factorial(1) = 1 (BASE CASE!)
                                                         ▲ Trả ngược kết quả lên
```

### 2 Thành phần sinh tử của hàm đệ quy:
1. **Base Case (Điểm dừng):** Điều kiện biên để dừng lại và trả về giá trị trực tiếp. **Nếu thiếu Base Case, chương trình sẽ gọi mãi mãi dẫn tới lỗi tràn ngăn xếp (Stack Overflow)!**
2. **Recursive Step (Bước đệ quy):** Gọi lại chính hàm đó nhưng với đối số nhỏ hơn, tiến dần về phía Base Case.

```javascript
// Tính giai thừa n!
function factorial(n) {
  // 1. Base case: 0! = 1 và 1! = 1
  if (n <= 1) return 1;

  // 2. Recursive step
  return n * factorial(n - 1);
}
```

---

## 5. Các Mẫu Tư Duy Thuật Toán Thực Chiến (Algorithmic Patterns)

### 5.1. Kỹ thuật Hai Con Trỏ (Two Pointers)
Thay vì dùng 2 vòng lặp lồng nhau $O(N^2)$, ta dùng 2 con trỏ dịch chuyển trên mảng để đạt độ phức tạp tuyến tính **$O(N)$**.

- **Con trỏ tiến lại gần nhau (Left & Right):** Áp dụng trên mảng đã sắp xếp. Một con trỏ ở đầu (`left = 0`), một con trỏ ở cuối (`right = len - 1`). Tùy vào tổng hoặc điều kiện, ta cho `left++` hoặc `right--`.
- **Con trỏ Nhanh & Chậm (Fast & Slow Pointers / Floyd's Cycle):** Phát hiện vòng lặp (chu trình) trong Danh sách liên kết.

### 5.2. Kỹ thuật Cửa Sổ Trượt (Sliding Window)
Áp dụng cho các bài toán liên quan đến **chuỗi con (substring) hoặc mảng con liên tiếp (subarray)**:
- Mở rộng biên phải của cửa sổ (`right++`) để nạp thêm dữ liệu.
- Khi điều kiện bị vi phạm, thu hẹp biên trái của cửa sổ (`left++`) cho đến khi điều kiện hợp lệ trở lại.
- Tránh việc phải tính toán lại từ đầu các phần tử nằm trong vùng giao nhau giữa các cửa sổ.

### 5.3. Quy Hoạch Động (Dynamic Programming - DP) Căn Bản
DP không phải là một thuật toán cụ thể, mà là một **phương pháp tối ưu hóa**: Chia bài toán lớn thành các bài toán con trùng lặp, giải từng bài toán con một lần duy nhất và **lưu kết quả vào bộ nhớ** để tái sử dụng (không tính lại lần 2).

```text
Tính Fibonacci f(5) không dùng DP:
             f(5)
           /      \
        f(4)       f(3)
       /    \     /    \
     f(3)  f(2)  f(2)  f(1)   ◄── f(3) và f(2) bị tính đi tính lại nhiều lần!
     (Độ phức tạp bùng nổ: O(2^N))

Tính Fibonacci dùng Memoization (Ghi nhớ vào Hash Map / Array):
Khi f(3) tính xong lần đầu, lưu vào RAM. Lần sau cần f(3), lấy ra dùng ngay O(1)!
=> Giảm độ phức tạp từ O(2^N) xuống O(N)!
```

---

## 6. Tóm Tắt & Ghi Nhớ Nhanh

1. **Big-O** đo tốc độ tăng trưởng của số phép tính khi $N$ tăng lên vô cực. Luôn hướng tới $O(1), O(\log N), O(N)$ hoặc $O(N \log N)$.
2. **Binary Search** đạt tốc độ siêu việt $O(\log N)$ nhưng dữ liệu bắt buộc phải sắp xếp trước.
3. Các thuật toán sắp xếp tốt nhất dựa trên so sánh (Merge Sort, Quick Sort) có cận dưới lý thuyết là **$O(N \log N)$**.
4. Mọi hàm **đệ quy** đều cần có **Base Case** rõ ràng để tránh sập Call Stack.
5. Học thuộc các mẫu tư duy: **Two Pointers**, **Sliding Window** và **Memoization** để tối ưu hóa thuật toán từ bậc hai $O(N^2)$ xuống tuyến tính $O(N)$.
