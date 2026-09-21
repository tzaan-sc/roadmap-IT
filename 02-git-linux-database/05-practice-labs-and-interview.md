# 05 - Bài Tập Thực Hành & Bộ Câu Hỏi Phỏng Vấn (Practice Labs & Interview Questions)

> **Mục tiêu bài học:** Thực chiến toàn diện kiến thức Git, Linux và Database thông qua 3 bài lab chuẩn môi trường sản xuất thực tế, kèm bộ 15 câu hỏi phỏng vấn kỹ thuật cốt lõi có lời giải chi tiết giúp bạn tự tin ứng tuyển vị trí Kỹ sư Phần mềm / Backend / DevOps.

---

## PHẦN 1: CÁC BÀI LAB THỰC CHIẾN

### LAB 1: Mô Phỏng & Tự Tay Giải Quyết Git Conflict

#### Đề bài:
Tạo một kho Git cục bộ, tạo 2 nhánh cùng sửa một dòng trong file `config.js` để gây ra xung đột (Merge Conflict) có chủ đích, sau đó tự tay giải quyết và hoàn tất hợp nhất.

#### Các bước thực hiện:
```bash
# 1. Tạo thư mục thử nghiệm và khởi tạo git
mkdir git-conflict-lab && cd git-conflict-lab
git init

# 2. Tạo file ban đầu trên nhánh main
echo "const API_PORT = 3000;" > config.js
git add config.js
git commit -m "feat: khởi tạo cổng API mặc định 3000"

# 3. Tạo nhánh mới feature/port-8080 và đổi cổng thành 8080
git switch -c feature/port-8080
echo "const API_PORT = 8080;" > config.js
git commit -am "feat: nâng cổng API lên 8080 cho môi trường dev"

# 4. Quay lại nhánh main và đổi cổng thành 9000
git switch main
echo "const API_PORT = 9000;" > config.js
git commit -am "feat: đổi cổng API thành 9000 theo yêu cầu production"

# 5. Hợp nhất nhánh feature/port-8080 vào main -> BÙNG NỔ CONFLICT!
git merge feature/port-8080
```

Git sẽ báo lỗi:
`CONFLICT (content): Merge conflict in config.js. Automatic merge failed; fix conflicts and then commit the result.`

#### Xử lý xung đột:
Mở file `config.js`, bạn sẽ thấy:
```javascript
<<<<<<< HEAD
const API_PORT = 9000;
=======
const API_PORT = 8080;
>>>>>>> feature/port-8080
```
- Chọn giữ lại giá trị đúng (ví dụ quyết định dùng cổng `9000`).
- Xóa sạch các dòng ký hiệu `<<<<<<<`, `=======`, `>>>>>>>`.
- Nội dung file sau khi sửa:
```javascript
const API_PORT = 9000;
```
- Hoàn tất merge:
```bash
git add config.js
git commit -m "fix: giải quyết xung đột cổng API, thống nhất sử dụng cổng 9000"
git log --oneline --graph
```

---

### LAB 2: Xây Dựng & Cấu Hình Systemd Service Tự Động Chạy 24/7 Trên Linux

#### Đề bài:
Viết một script server mini bằng Python hoặc Node.js, sau đó cấu hình nó thành một dịch vụ hệ thống (**systemd daemon**) tự động chạy nền và tự hồi sinh khi server reboot.

#### Các bước thực hiện:
1. Tạo thư mục ứng dụng tại `/var/www/ping-app` và tạo file `app.py`:
```python
# /var/www/ping-app/app.py
import time
from http.server import HTTPServer, BaseHTTPRequestHandler

class PingHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        self.wfile.write(b'{"status": "UP", "timestamp": %d}' % int(time.time()))

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8080), PingHandler)
    print("Server dang chay tai port 8080...")
    server.serve_forever()
```

2. Tạo file service cấu hình tại `/etc/systemd/system/pingapp.service`:
```ini
[Unit]
Description=Ping Health Check Python Daemon
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/var/www/ping-app
ExecStart=/usr/bin/python3 /var/www/ping-app/app.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

3. Nạp và khởi động dịch vụ:
```bash
sudo systemctl daemon-reload
sudo systemctl start pingapp
sudo systemctl enable pingapp
sudo systemctl status pingapp
```

4. Kiểm tra kết quả bằng lệnh `curl`:
```bash
curl http://localhost:8080
# Kết quả trả về: {"status": "UP", "timestamp": 1726912345}
```

---

### LAB 3: Thiết Kế Schema E-commerce & Viết Truy Vấn Báo Cáo Doanh Thu Bằng SQL

#### Đề bài:
Thiết kế CSDL quản lý bán hàng gồm: `users`, `categories`, `products`, `orders`, `order_items`. Viết câu lệnh tính **Top 3 danh mục sản phẩm mang lại doanh thu cao nhất năm 2024**.

```sql
-- 1. Tạo các bảng với ràng buộc Khóa chính và Khóa ngoại
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    category_id INT REFERENCES categories(id) ON DELETE RESTRICT,
    title VARCHAR(200) NOT NULL,
    price DECIMAL(10, 2) NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'PAID'
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INT REFERENCES orders(id) ON DELETE CASCADE,
    product_id INT REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL
);

-- 2. Đánh Index tối ưu cho việc lọc và nối bảng
CREATE INDEX idx_order_items_product_id ON order_items(product_id);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
CREATE INDEX idx_orders_date_status ON orders(order_date, status);

-- 3. CÂU TRUY VẤN BÁO CÁO DOANH THU KẾT HỢP 4 BẢNG
SELECT 
    c.name AS category_name,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(oi.quantity * oi.unit_price) AS total_revenue
FROM categories c
INNER JOIN products p ON c.id = p.category_id
INNER JOIN order_items oi ON p.id = oi.product_id
INNER JOIN orders o ON o.id = oi.order_id
WHERE o.status = 'PAID' 
  AND o.order_date >= '2024-01-01' 
  AND o.order_date < '2025-01-01'
GROUP BY c.id, c.name
ORDER BY total_revenue DESC
LIMIT 3;
```

---

## PHẦN 2: BỘ 15 CÂU HỎI PHỎNG VẤN KỸ THUẬT CỐT LÕI (CÓ LỜI GIẢI)

### Câu 1: Phân biệt sự khác nhau giữa `git merge` và `git rebase`? Khi nào KHÔNG được dùng rebase?
- **Trả lời:**
  - `git merge` kết hợp hai nhánh bằng cách tạo ra một **Merge Commit** mới. Nó giữ nguyên 100% lịch sử gốc và thứ tự thời gian của các commit, nhưng làm cây lịch sử phân nhánh nhiều nhánh đan xen.
  - `git rebase` viết lại lịch sử bằng cách lấy toàn bộ commit của nhánh hiện tại và "đặt lại gốc" nối tiếp vào đuôi của nhánh đích, tạo ra lịch sử là **một đường thẳng duy nhất**.
  - **Quy tắc vàng:** **Không bao giờ rebase trên nhánh công khai (Public Branch như `main` hoặc `develop`)** vì nó thay đổi mã băm commit, gây thảm họa xung đột lịch sử cho toàn bộ các thành viên khác trong nhóm.

### Câu 2: Phân biệt `git reset` (soft, mixed, hard) và `git revert`?
- **Trả lời:**
  - `git reset` tua ngược lại con trỏ HEAD về commit quá khứ:
    - `--soft`: Giữ nguyên mã nguồn đã sửa trong **Staging Area**.
    - `--mixed` (mặc định): Giữ nguyên mã nguồn đã sửa trong **Working Directory**.
    - `--hard`: **Xóa sạch hoàn toàn** mọi thay đổi code. (Không nên dùng nếu commit đã push lên server).
  - `git revert`: **Tạo một commit hoàn toàn mới** có nội dung đảo ngược lại các thay đổi của commit cũ. Rất an toàn để sửa lỗi trên nhánh đã push lên GitHub vì nó không viết lại lịch sử cũ.

### Câu 3: SSH Key hoạt động như thế nào? Sự khác biệt giữa Public Key và Private Key?
- **Trả lời:**
  - SSH sử dụng thuật toán **Mã hóa bất đối xứng (Asymmetric Cryptography)** với một cặp khóa:
    - **Private Key (`id_ed25519`):** Lưu trữ bí mật tuyệt đối trên máy tính cá nhân của bạn, không bao giờ được gửi qua mạng hay chia sẻ cho ai.
    - **Public Key (`id_ed25519.pub`):** Đưa lên máy chủ từ xa lưu vào file `~/.ssh/authorized_keys`.
  - Khi bạn kết nối, server gửi một thông điệp thử thách ngẫu nhiên. Máy bạn dùng Private Key để ký xác thực. Server dùng Public Key giải mã để đối chiếu. Nếu khớp, server cho phép đăng nhập mà không cần truyền mật khẩu thô qua mạng.

### Câu 4: Làm thế nào để tìm và tiêu diệt tiến trình đang chiếm dụng cổng 8080 trên Linux?
- **Trả lời:**
  ```bash
  # 1. Tìm PID của tiến trình đang chiếm cổng 8080:
  sudo lsof -i :8080
  # Hoặc dùng ss:
  sudo ss -tulpn | grep :8080

  # 2. Tiêu diệt tiến trình bằng PID vừa tìm được (ví dụ PID là 1234):
  sudo kill -15 1234   # Gửi tín hiệu đóng an toàn (SIGTERM)
  # Nếu không tắt được:
  sudo kill -9 1234    # Buộc dừng ngay lập tức (SIGKILL)
  ```

### Câu 5: Lệnh `nohup` và ký tự `&` trong Linux có ý nghĩa gì?
- **Trả lời:**
  - Khi đăng xuất khỏi phiên SSH, terminal sẽ gửi tín hiệu `SIGHUP` để ngắt các tiến trình con.
  - **`nohup` (No Hangup)** giúp tiến trình bỏ qua tín hiệu `SIGHUP`, giúp chương trình tiếp tục chạy ngay cả khi tắt cửa sổ Terminal.
  - Ký tự **`&`** ở cuối câu lệnh chỉ định đẩy tiến trình chạy ngầm dưới nền (background) thay vì chiếm giữ con trỏ gõ lệnh của Shell.

### Câu 6: Phân biệt `WHERE` và `HAVING` trong SQL?
- **Trả lời:**
  - **`WHERE`**: Dùng để lọc các hàng dữ liệu thô **trước khi** dữ liệu được gom nhóm hoặc tính toán bởi các hàm tổng hợp (`SUM`, `COUNT`, `AVG`).
  - **`HAVING`**: Dùng để lọc kết quả của các nhóm **sau khi** mệnh đề `GROUP BY` đã gom nhóm và tính toán xong.

### Câu 7: Phân biệt `INNER JOIN` và `LEFT JOIN`?
- **Trả lời:**
  - **`INNER JOIN`**: Chỉ trả về các bản ghi khi có sự khớp dữ liệu ở **cả hai bảng**. Những hàng ở bảng A không tìm thấy hàng tương ứng ở bảng B sẽ bị loại bỏ hoàn toàn.
  - **`LEFT JOIN`**: Trả về **tất cả các bản ghi của bảng bên trái (bảng A)**, bất kể bảng bên phải (bảng B) có bản ghi khớp hay không. Nếu không khớp, các cột của bảng B sẽ mang giá trị `NULL`.

### Câu 8: Hãy giải thích 4 tính chất ACID trong Database Transaction?
- **Trả lời:**
  - **A (Atomicity):** Tất cả các câu lệnh trong transaction cùng thành công hoặc cùng bị hủy bỏ (`ROLLBACK`). Không có trạng thái dở dang.
  - **C (Consistency):** Đảm bảo CSDL luôn chuyển từ trạng thái hợp lệ này sang trạng thái hợp lệ khác, không vi phạm các ràng buộc toàn vẹn dữ liệu.
  - **I (Isolation):** Các transaction chạy đồng thời không nhìn thấy hoặc can thiệp vào dữ liệu chưa commit của nhau.
  - **D (Durability):** Một khi đã `COMMIT`, dữ liệu được ghi bền vững vào đĩa (nhờ WAL log), không bị mất mát kể cả khi hệ thống sập nguồn.

### Câu 9: B-Tree Index là gì? Tại sao nó giúp tăng tốc độ đọc nhưng làm chậm tốc độ ghi?
- **Trả lời:**
  - B-Tree Index là cấu trúc cây tự cân bằng lưu trữ các giá trị của một hoặc nhiều cột theo thứ tự đã sắp xếp. Thay vì phải quét toàn bộ bảng ($O(N)$), Database chỉ mất $O(\log N)$ phép tính để tìm thấy con trỏ trỏ tới dữ liệu thực tế trên ổ cứng.
  - Nó làm **chậm tốc độ ghi (`INSERT`, `UPDATE`, `DELETE`)** vì mỗi khi có dữ liệu mới thêm vào, CSDL không chỉ ghi dữ liệu vào bảng mà còn phải tính toán để chèn node mới và cân bằng lại cây B-Tree cho mọi index trên bảng đó.

### Câu 10: `EXPLAIN ANALYZE` trong PostgreSQL dùng để làm gì? Phân biệt `Seq Scan` và `Index Scan`?
- **Trả lời:**
  - `EXPLAIN ANALYZE` là câu lệnh phân tích kế hoạch thực thi của truy vấn SQL bằng cách thực sự chạy câu lệnh và in ra chi phí dự đoán (Cost) cùng thời gian thực thi thực tế (Execution Time).
  - **`Seq Scan` (Sequential / Full Table Scan):** Database phải đọc từng khối dữ liệu của toàn bộ bảng trên ổ cứng. Rất chậm trên bảng lớn.
  - **`Index Scan`:** Database tra cứu trong cây chỉ mục Index để tìm trực tiếp các dòng thỏa mãn điều kiện rồi mới nhảy tới ổ cứng lấy dữ liệu. Nhanh hơn gấp hàng nghìn lần.

### Câu 11: Chuẩn hóa dữ liệu đến dạng chuẩn 3 (3NF) là gì? Tại sao cần áp dụng?
- **Trả lời:**
  - **1NF:** Dữ liệu trong mỗi ô phải có tính nguyên tử (atomic), không chứa mảng hay danh sách.
  - **2NF:** Đạt 1NF và mọi thuộc tính không khóa phải phụ thuộc vào toàn bộ khóa chính (tránh phụ thuộc một phần vào khóa tổ hợp).
  - **3NF:** Đạt 2NF và không có thuộc tính không khóa nào phụ thuộc bắc cầu vào một thuộc tính không khóa khác.
  - **Mục đích:** Loại bỏ sự dư thừa dữ liệu (Data Redundancy), tiết kiệm bộ nhớ và tránh các lỗi dị thường khi cập nhật dữ liệu.

### Câu 12: Khi nào nên chọn NoSQL thay vì RDBMS?
- **Trả lời:**
  - Chọn **RDBMS (PostgreSQL/MySQL)** khi: Dữ liệu có cấu trúc rõ ràng, quan hệ phức tạp giữa các thực thể, yêu cầu toàn vẹn giao dịch tài chính nghiêm ngặt (ACID).
  - Chọn **NoSQL (MongoDB, Cassandra, Redis)** khi: Dữ liệu có cấu trúc phi cấu trúc hoặc thường xuyên thay đổi (Document), yêu cầu lưu trữ và ghi dữ liệu khổng lồ với tốc độ cực cao, hoặc cần mở rộng theo chiều ngang (Horizontal Sharding) trên nhiều cụm máy chủ.

### Câu 13: Làm thế nào để cấu hình dịch vụ trong Linux tự khởi động lại khi server reboot?
- **Trả lời:**
  - Tạo file cấu hình dịch vụ `.service` trong thư mục `/etc/systemd/system/`.
  - Trong phần `[Install]`, cấu hình `WantedBy=multi-user.target`.
  - Chạy lệnh: `sudo systemctl daemon-reload` và `sudo systemctl enable <tên-dịch-vụ>`.

### Câu 14: Pull Request (PR) là gì và quy trình Code Review diễn ra như thế nào?
- **Trả lời:**
  - Pull Request là cơ chế trên GitHub/GitLab thông báo cho các thành viên trong nhóm biết rằng một tính năng trên nhánh cá nhân đã hoàn tất và sẵn sàng được gộp vào nhánh chính (`main`/`develop`).
  - Trong quá trình Code Review, đồng đội sẽ đọc từng dòng thay đổi (diff), để lại nhận xét góp ý về hiệu năng, bảo mật hoặc quy chuẩn code (convention). Người tạo PR sửa đổi bổ sung commit cho đến khi được duyệt (**Approve**) thì mới được phép nhấn nút Merge.

### Câu 15: Các bước cơ bản để tăng cường bảo mật cho một máy chủ Linux mới dựng là gì?
- **Trả lời:**
  1. Cập nhật các gói phần mềm mới nhất: `sudo apt update && sudo apt upgrade -y`.
  2. Tạo user mới không phải root và cấp quyền `sudo`, không sử dụng trực tiếp tài khoản `root`.
  3. Cấu hình đăng nhập bằng **SSH Key** và tắt đăng nhập bằng mật khẩu thường (`PasswordAuthentication no` trong `sshd_config`).
  4. Đổi cổng SSH mặc định từ 22 sang cổng khác (ví dụ 2222) để giảm thiểu các bot tự động dò quét mạng.
  5. Bật tường lửa **`ufw`**, chỉ mở đúng các cổng cần thiết (SSH, HTTP 80, HTTPS 443).
  6. Cài đặt công cụ chống dò quét mật khẩu tự động như **Fail2ban**.

---

## PHẦN 3: CHECKLIST TỰ ĐÁNH GIÁ KỸ NĂNG (LEVEL 2 COMPETENCY)

Hãy tự kiểm tra xem bạn đã thành thạo các kỹ năng này chưa:

- [ ] Tôi hiểu rõ sự khác nhau giữa 4 vùng làm việc của Git và vòng đời của file.
- [ ] Tôi tự tin tạo nhánh, giải quyết xung đột (Merge Conflict) và biết khi nào nên dùng `merge` vs `rebase`.
- [ ] Tôi biết cách cất tạm code dang dở bằng `git stash` và hủy commit an toàn bằng `git revert`.
- [ ] Tôi thành thạo quy trình làm việc nhóm qua GitHub Pull Request.
- [ ] Tôi có thể thiết lập đăng nhập máy chủ Linux bằng SSH Key mà không cần mật khẩu.
- [ ] Tôi tự viết được file cấu hình `systemd service` để quản lý app chạy ngầm tự phục hồi.
- [ ] Tôi biết dùng `lsof` và `kill` để giải phóng port mạng bị chiếm dụng.
- [ ] Tôi thành thạo các câu lệnh SQL CRUD, `INNER JOIN`, `LEFT JOIN` và `GROUP BY / HAVING`.
- [ ] Tôi hiểu rõ 4 tính chất ACID và cách dùng Transaction để bảo vệ tính nhất quán dữ liệu.
- [ ] Tôi hiểu cấu trúc cây B-Tree của Index, biết khi nào nên tạo Index và đánh đổi của nó.
- [ ] Tôi biết dùng `EXPLAIN ANALYZE` để phát hiện câu truy vấn bị `Seq Scan`.
