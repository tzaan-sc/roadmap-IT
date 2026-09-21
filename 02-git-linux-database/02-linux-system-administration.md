# 02 - Quản Trị Hệ Thống Linux Thực Chiến (Linux System Administration)

> **Mục tiêu bài học:** Nâng tầm kỹ năng Linux từ các lệnh điều hướng cơ bản lên trình độ quản trị máy chủ thực tế: làm chủ bộ công cụ quản lý dịch vụ `systemd`, kết nối không cần mật khẩu qua giao thức bảo mật `SSH`, quản lý tiến trình chạy ngầm (daemons/background jobs), phân quyền bảo mật và thao tác mạng với `curl` và tường lửa `ufw`.

---

## 1. Tại Sao Linux Là Trái Tim Của Backend & Cloud?

Trong thế giới phát triển phần mềm:
- **Hơn 95%** các máy chủ Internet (Web Server, Database, Cloud AWS / GCP / Azure) đều chạy hệ điều hành Linux (Ubuntu, Debian, CentOS, RHEL, Alpine).
- Toàn bộ công nghệ ảo hóa hiện đại như **Docker**, **Kubernetes**, Serverless containers đều được sinh ra từ chính các tính năng của Linux Kernel (Namespaces và Cgroups).

> Nếu không biết Linux, bạn sẽ không thể deploy, debug hay duy trì một ứng dụng thực tế nào trên môi trường Internet.

---

## 2. Quản Lý Dịch Vụ Hệ Thống Với `systemd` & `systemctl`

Trên hầu hết các bản phân phối Linux hiện đại (Ubuntu, Debian, CentOS), **`systemd`** là tiến trình mẹ đầu tiên (PID 1) được Kernel khởi động, chịu trách nhiệm quản lý tất cả các dịch vụ (services/daemons) khác của hệ thống.

```text
                                  LINUX KERNEL
                                       │
                                       ▼
                       systemd (Tiến trình gốc PID = 1)
                                       │
         ┌─────────────────────────────┼─────────────────────────────┐
         ▼                             ▼                             ▼
   nginx.service               mysqld.service              my-node-app.service
  (Web Server)                 (Database)                  (App của bạn!)
```

### 2.1. Các lệnh `systemctl` bắt buộc phải thuộc lòng
```bash
# Kiểm tra trạng thái dịch vụ (đang chạy hay đã chết, xem log lỗi gần nhất)
sudo systemctl status nginx

# Bật dịch vụ
sudo systemctl start nginx

# Tắt dịch vụ
sudo systemctl stop nginx

# Khởi động lại dịch vụ (thường dùng sau khi sửa file cấu hình)
sudo systemctl restart nginx

# Tải lại cấu hình mà không làm gián đoạn kết nối hiện tại của người dùng
sudo systemctl reload nginx

# BẬT TỰ ĐỘNG KHỞI ĐỘNG CÙNG HỆ ĐIỀU HÀNH (Khi server reboot thì app tự bật lại)
sudo systemctl enable nginx

# Tắt tính năng tự khởi động cùng OS
sudo systemctl disable nginx
```

### 2.2. Tự tạo một Systemd Service cho ứng dụng của bạn
Giả sử bạn có một ứng dụng Node.js hoặc Python API nằm tại `/home/ubuntu/app/server.js`. Làm sao để app này tự chạy ngầm 24/7 và tự bật lại khi server bị khởi động lại đột ngột?

Tạo file cấu hình dịch vụ tại `/etc/systemd/system/myapp.service`:
```ini
[Unit]
Description=My Awesome Backend API Service
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/app
ExecStart=/usr/bin/node /home/ubuntu/app/server.js
Restart=always
RestartSec=5
Environment=NODE_ENV=production PORT=3000

[Install]
WantedBy=multi-user.target
```

Kích hoạt và chạy ứng dụng:
```bash
# 1. Báo cho systemd nạp lại các file cấu hình mới
sudo systemctl daemon-reload

# 2. Bật dịch vụ và kích hoạt tự khởi động
sudo systemctl start myapp
sudo systemctl enable myapp

# 3. Theo dõi log thời gian thực của ứng dụng
journalctl -u myapp -f
```

---

## 3. Kết Nối Máy Chủ An Toàn Qua SSH (Secure Shell)

**SSH (Cổng mặc định 22)** là giao thức tiêu chuẩn để kết nối và điều khiển một máy chủ Linux từ xa thông qua kênh truyền mã hóa mạnh.

```text
MÁY CÁ NHÂN CỦA BẠN (CLIENT)                        MÁY CHỦ LINUX (SERVER)
┌────────────────────────┐                          ┌────────────────────────┐
│ PRIVATE KEY (id_ed25519)│                          │ PUBLIC KEY             │
│ (GIỮ BÍ MẬT TUYỆT ĐỐI! │ ─── Thử thách mật mã ──► │ (~/.ssh/authorized_keys│
│  Không chia sẻ cho ai) │ ◄── Xác thực thành công ──│ (Chìa khóa cắm ổ)      │
└────────────────────────┘                          └────────────────────────┘
```

### 3.1. Các bước thiết lập đăng nhập không cần mật khẩu (SSH Key-based Auth)

#### Bước 1: Tạo cặp khóa SSH trên máy cá nhân
Mở Terminal trên máy tính của bạn và gõ:
```bash
# Tạo khóa bằng thuật toán hiện đại Ed25519 (nhanh và an toàn hơn RSA)
ssh-keygen -t ed25519 -C "email-cua-ban@gmail.com"
# Bấm Enter liên tiếp để lưu vào thư mục mặc định (~/.ssh/id_ed25519)
```

#### Bước 2: Đẩy Public Key lên máy chủ
```bash
# Lệnh tự động copy public key vào file ~/.ssh/authorized_keys trên server:
ssh-copy-id ubuntu@192.168.1.100

# Hoặc nếu dùng Windows không có ssh-copy-id:
# Mở file ~/.ssh/id_ed25519.pub, copy toàn bộ nội dung và dán vào
# file ~/.ssh/authorized_keys trên server.
```

#### Bước 3: Đăng nhập tức thì mà không cần gõ mật khẩu
```bash
ssh ubuntu@192.168.1.100
```

### 3.2. Chuyển tệp tin an toàn qua mạng với `scp` và `rsync`
```bash
# Copy file từ máy cá nhân lên server từ xa
scp my-archive.zip ubuntu@192.168.1.100:/home/ubuntu/

# Đồng bộ cả thư mục code thông minh với rsync (chỉ truyền các file có thay đổi)
rsync -avz --exclude 'node_modules' ./my-project ubuntu@192.168.1.100:/var/www/
```

### 3.3. Tăng cường bảo mật tối đa cho máy chủ (`/etc/ssh/sshd_config`)
Sau khi đã thiết lập SSH Key thành công, bạn nên tắt tính năng đăng nhập bằng mật khẩu thường để chống các cuộc tấn công quét dò pass (Brute-force):
```bash
sudo nano /etc/ssh/sshd_config
```
Chỉnh sửa 2 dòng sau:
```ini
PasswordAuthentication no
PermitRootLogin no
```
Khởi động lại SSH daemon: `sudo systemctl restart ssh`

---

## 4. Quản Lý Tiến Trình (Processes) & Chạy Tác Vụ Ngầm (Background Jobs)

### 4.1. Giám sát tài nguyên
- `top`: Trình giám sát tài nguyên dạng bảng kinh điển.
- `htop`: Giao diện đồ họa màu trực quan hơn rất nhiều (cài bằng `sudo apt install htop`).
- `ps aux | grep python`: Tìm tất cả tiến trình Python đang chạy kèm theo PID.

### 4.2. Chạy ứng dụng ngầm không lo bị ngắt với `nohup`
Khi bạn SSH vào server và gõ `python bot.py`, nếu bạn tắt cửa sổ Terminal, hệ điều hành sẽ gửi tín hiệu `SIGHUP` (Hangup Signal) để giết chết tiến trình đó. Để ứng dụng sống sót kể cả khi ngắt kết nối SSH:

```bash
# nohup (No Hangup) + chuyển hướng log + dấu & (chạy ngầm)
nohup python bot.py > bot.log 2>&1 &
```
- `> bot.log`: Ghi toàn bộ `stdout` vào file `bot.log`.
- `2>&1`: Gom cả luồng lỗi `stderr` (kênh 2) vào cùng luồng `stdout` (kênh 1).
- `&`: Đẩy tiến trình xuống nền (Background), trả lại dòng lệnh cho bạn tiếp tục làm việc.

### 4.3. Kiểm tra Port mạng đang bị chiếm dụng
Một trong những lỗi kinh điển nhất của Backend Dev: `Error: listen EADDRINUSE: address already in use :::3000`.

```bash
# Cách 1: Dùng lệnh lsof để tìm tiến trình nào đang chiếm cổng 3000
sudo lsof -i :3000

# Kết quả hiển thị:
# COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# node    12450 ubuntu   23u  IPv6  89452      0t0  TCP *:3000 (LISTEN)

# Tiêu diệt tiến trình đó ngay lập tức:
sudo kill -9 12450

# Cách 2: Dùng lệnh ss để liệt kê toàn bộ các cổng TCP đang lắng nghe
sudo ss -tulpn
```

---

## 5. Thao Tác Mạng Với `curl` & Tường Lửa `ufw`

### 5.1. Thần tài `curl` (Command Line URL)
Lập trình viên backend dùng `curl` liên tục để test API mà không cần mở Postman:

```bash
# 1. Gửi request GET đơn giản
curl https://api.github.com/users/octocat

# 2. Chỉ xem HTTP Response Header (Mã trạng thái 200, 404, Cookies, Content-Type)
curl -I https://google.com

# 3. Gửi request POST kèm JSON Body và Authentication Header
curl -X POST https://api.mysite.com/users \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer my_secret_token_123" \
  -d '{"name": "Nguyen Van A", "email": "a@gmail.com"}'
```

### 5.2. Quản lý Tường lửa (Firewall) với `ufw` (Uncomplicated Firewall)
Bảo vệ server khỏi các cuộc tấn công bằng cách chặn toàn bộ các cổng mạng, chỉ mở những cổng thực sự cần thiết:

```bash
# 1. Cho phép cổng SSH trước (QUAN TRỌNG: Quên bước này là bạn tự khóa cửa ngoài!)
sudo ufw allow 22/tcp

# 2. Cho phép cổng Web HTTP và HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 3. Kích hoạt tường lửa
sudo ufw enable

# 4. Kiểm tra danh sách các cổng đang mở
sudo ufw status verbose
```

---

## 6. Phân Quyền & Quản Lý Người Dùng An Toàn

### 6.1. Nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege)
Tuyệt đối không bao giờ dùng tài khoản `root` để chạy ứng dụng hàng ngày. Nếu ứng dụng có lỗ hổng bảo mật, hacker sẽ chiếm toàn quyền kiểm soát cả máy chủ! Hãy luôn tạo tài khoản người dùng thông thường và cấp quyền `sudo` khi cần thiết:

```bash
# Tạo user mới có tên "deployer"
sudo adduser deployer

# Thêm user vào nhóm sudo (được phép chạy quyền quản trị)
sudo usermod -aG sudo deployer
```

### 6.2. Thay đổi quyền sở hữu tệp (`chown`)
```bash
# Gán thư mục /var/www/html cho user www-data và nhóm www-data
sudo chown -R www-data:www-data /var/www/html
```

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **`systemd`** và **`systemctl`** là công cụ chuẩn để biến code thành dịch vụ chạy ngầm 24/7 tự khởi động cùng hệ thống.
2. Dùng **SSH Key** (thuật toán `ed25519`) để đăng nhập an toàn, vô hiệu hóa đăng nhập bằng mật khẩu gốc trên server.
3. Dùng `nohup ... &` để chạy ngầm tác vụ; dùng `lsof -i :port` hoặc `ss -tulpn` để tìm và giải phóng port bị chiếm dụng.
4. Nắm vững cú pháp **`curl`** với `-X`, `-H`, `-d` để kiểm thử API ngay từ dòng lệnh.
5. Luôn kích hoạt tường lửa **`ufw`** và tuân thủ nguyên tắc không dùng trực tiếp tài khoản `root`.
