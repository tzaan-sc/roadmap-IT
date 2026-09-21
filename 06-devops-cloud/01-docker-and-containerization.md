# 01 - Docker & Công Nghệ Ảo Hóa Container (Docker & Containerization)

> **Mục tiêu bài học:** Chấm dứt vĩnh viễn nỗi ám ảnh kinh điển *"Nó chạy được trên máy của tôi nhưng lỗi trên server!"*, hiểu sâu bản chất cô lập của Container (Linux Namespaces & Cgroups) so với Máy ảo (VM), viết `Dockerfile` chuẩn công nghiệp với kỹ thuật Multi-stage build giúp giảm 90% dung lượng, và quản lý lưu trữ bền vững với Docker Volumes.

---

## 1. Vấn Đề Lịch Sử: "It Works On My Machine!"

Trước khi có Docker:
1. Bạn viết code trên máy tính cá nhân dùng Windows 11, Node.js v20.5, PostgreSQL v15. Mọi thứ chạy mượt mà.
2. Bạn bàn giao code cho đội vận hành (Ops) để đưa lên máy chủ Linux Ubuntu chạy Node.js v18 và PostgreSQL v13.
3. Ứng dụng lập tức bị sập vì **xung đột phiên bản thư viện, biến môi trường bị thiếu, hoặc khác biệt về hệ điều hành**.

```text
       MÃ NGUỒN CỦA BẠN                         MÔI TRƯỜNG MÁY CHỦ
┌─────────────────────────────┐           ┌─────────────────────────────┐
│ Code + Node v20 + Postgres15│           │ Server Linux + Node v18...  │
└──────────────┬──────────────┘           └──────────────▲──────────────┘
               │                                         │
               └─────────── DEPLOY THỦ CÔNG ─────────────┘
                             (BÙNG NỔ LỖI XUNG ĐỘT!)
```

**Docker (ra đời năm 2013)** đã thay đổi hoàn toàn cuộc chơi: Thay vì chỉ gửi mã nguồn, bạn đóng gói **TOÀN BỘ MÃ NGUỒN + MÔI TRƯỜNG CHẠY + TẤT CẢ THƯ VIỆN + CẤU HÌNH HỆ ĐIỀU HÀNH** thành một gói tiêu chuẩn gọi là **Docker Container**. 
> *"Nếu nó chạy được trong Container trên máy bạn, nó chắc chắn sẽ chạy giống hệt 100% trên bất kỳ máy chủ nào trên thế giới!"*

---

## 2. Bản Chất: Máy Ảo (Virtual Machine) vs Container (Docker)

Nhiều người thường nhầm lẫn Docker là một loại máy ảo. Sự khác biệt ở tầng sâu kiến trúc hệ điều hành:

```text
       KIẾN TRÚC MÁY ẢO (VIRTUAL MACHINE)                KIẾN TRÚC DOCKER CONTAINER
┌──────────────────────────────────────────────┐ ┌──────────────────────────────────────────────┐
│  App A           App B           App C       │ │  App A           App B           App C       │
│ (Libs/Bins)     (Libs/Bins)     (Libs/Bins)  │ │ (Libs/Bins)     (Libs/Bins)     (Libs/Bins)  │
├───────────────┬───────────────┬──────────────┤ ├───────────────┬───────────────┬──────────────┤
│ Guest OS      │ Guest OS      │ Guest OS     │ │                                              │
│ (Ubuntu 2GB)  │ (CentOS 2GB)  │ (Debian 2GB) │ │            DOCKER ENGINE                     │
├───────────────┴───────────────┴──────────────┤ │ (Chia sẻ chung Linux Kernel của máy chủ!)    │
│            HYPERVISOR                        │ ├──────────────────────────────────────────────┤
├──────────────────────────────────────────────┤ │             HOST OPERATING SYSTEM            │
│         HOST OS / PHẦN CỨNG VẬT LÝ           │ ├──────────────────────────────────────────────┤
└──────────────────────────────────────────────┘ │             PHẦN CỨNG VẬT LÝ                 │
                                                 └──────────────────────────────────────────────┘
```

| Tiêu chí | Máy ảo (Virtual Machine - VM) | Docker Container |
| :--- | :--- | :--- |
| **Cơ chế hoạt động** | Giả lập toàn bộ phần cứng ảo và **chạy một hệ điều hành riêng biệt (Guest OS)** | Dùng chung **Nhân Kernel của máy chủ**, cô lập bằng tính năng của Linux |
| **Dung lượng** | Rất nặng: Từ **vài GB đến hàng chục GB** cho mỗi máy ảo | Siêu nhẹ: Từ **vài chục MB đến vài trăm MB** |
| **Thời gian khởi động** | Chậm: Mất từ **1 đến 3 phút** để boot hệ điều hành | **Tức thì: Vài phần trăm giây** ($< 1$ giây) |
| **Tiêu tốn tài nguyên** | Cấp phát cứng RAM và CPU (VM không dùng hết RAM vẫn bị giữ) | Cấp phát động linh hoạt (dùng bao nhiêu ăn bấy nhiêu) |

### Công nghệ bên dưới của Linux giúp Docker cô lập Container:
1. **Linux Namespaces (Không gian tên):** Tạo ra bức tường cô lập:
   - *PID Namespace:* Tiến trình bên trong container nghĩ rằng nó là tiến trình PID 1 duy nhất trên máy.
   - *NET Namespace:* Container có card mạng, địa chỉ IP và bảng định tuyến riêng.
   - *MNT Namespace:* Cây thư mục tệp tin độc lập hoàn toàn với máy chủ.
2. **Cgroups (Control Groups):** Kiểm soát và giới hạn lượng tài nguyên phần cứng (ví dụ: Container này chỉ được dùng tối đa 1 Core CPU và 512MB RAM).

---

## 3. Các Khái Niệm Cốt Lõi Của Docker

```text
┌───────────────────────────┐      docker build      ┌───────────────────────────┐
│        DOCKERFILE         │ ─────────────────────► │       DOCKER IMAGE        │
│ (Bản hướng dẫn nấu ăn)    │                        │  (Món ăn đã đóng hộp)     │
└───────────────────────────┘                        └─────────────┬─────────────┘
                                                                   │
                                                docker run ────────┴──────── docker push
                                                   │                            │
                                                   ▼                            ▼
                                     ┌───────────────────────────┐┌───────────────────────────┐
                                     │     DOCKER CONTAINER      ││      DOCKER REGISTRY      │
                                     │ (Instance sống đang chạy) ││ (Docker Hub / AWS ECR)    │
                                     └───────────────────────────┘└───────────────────────────┘
```

- **Dockerfile:** Một file văn bản chứa các chỉ thị từng bước để đóng gói ứng dụng.
- **Docker Image:** Ảnh chụp chỉ đọc (Read-only Snapshot) chứa mã nguồn, thư viện và môi trường. Có thể coi Image như một **Class** trong lập trình hướng đối tượng.
- **Docker Container:** Một thực thể sống đang chạy được tạo ra từ Image. Có thể coi Container như một **Object (Instance)** được đúc ra từ Class.
- **Docker Registry:** Kho chia sẻ Image toàn cầu (giống như GitHub cho mã nguồn). Phổ biến nhất là **Docker Hub**.

---

## 4. Kỹ Nghệ Viết `Dockerfile` Chuẩn Production (Multi-Stage Build)

### 4.1. Tối ưu hóa Layer Caching
Docker xây dựng Image theo từng lớp (**Layers**). Nếu một lớp không có sự thay đổi, Docker sẽ lấy lại từ bộ nhớ Cache:

```dockerfile
# ❌ CÁCH VIẾT NGÂY THƠ (Làm mất sạch Cache mỗi khi sửa code):
FROM node:20
WORKDIR /app
COPY . .                      # Mỗi lần bạn sửa 1 dòng code, lệnh này bị đổi
RUN npm install               # => Khiến Docker phải tải lại toàn bộ thư viện npm (Mất 5 phút!)
CMD ["node", "server.js"]

# ✅ CÁCH VIẾT TỐI ƯU CỦA SENIOR (Tận dụng Layer Caching):
FROM node:20
WORKDIR /app
COPY package*.json ./         # 1. Chỉ copy file khai báo thư viện trước
RUN npm install               # 2. Cài đặt thư viện (Nếu package.json không đổi, bước này chạy trong 0.1s!)
COPY . .                      # 3. Bây giờ mới copy mã nguồn (Sửa code chỉ tốn vài giây để build lại)
CMD ["node", "server.js"]
```

---

### 4.2. Multi-Stage Builds: Tuyệt Kỹ Giảm 95% Dung Lượng Image

Trong các ngôn ngữ biên dịch (Go, Rust, Java, hoặc build frontend React/Next.js), ta cần trình biên dịch nặng hàng Gigabytes để build code. Nhưng khi chạy thực tế trên server, ta **chỉ cần file thành phẩm siêu nhẹ**!

```text
GIAI ĐOẠN 1: BUILD STAGE (node:20 khổng lồ 1.2 GB)
Tải devDependencies, chạy TypeScript Compiler 'tsc', build thư mục 'dist/'
                              │
                              ▼ (Chỉ vác file build 'dist/' sang giai đoạn 2!)
GIAI ĐOẠN 2: RUNTIME STAGE (node:20-alpine siêu nhẹ chỉ 80 MB!)
Chỉ cài dependencies sản xuất và chạy file 'dist/server.js'. Vứt bỏ toàn bộ rác thừa!
```

#### File `Dockerfile` chuẩn mực cho ứng dụng Node.js/TypeScript:
```dockerfile
# -------------------------------------------------------------
# STAGE 1: Môi trường Build
# -------------------------------------------------------------
FROM node:20-alpine AS builder
WORKDIR /app

# Copy dependencies manifest
COPY package*.json ./
RUN npm ci # Cài đặt chính xác theo package-lock.json

# Copy mã nguồn và tiến hành biên dịch TypeScript
COPY . .
RUN npm run build

# -------------------------------------------------------------
# STAGE 2: Môi trường Chạy Thật (Production Runtime)
# -------------------------------------------------------------
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Tạo user không có đặc quyền root để tăng cường bảo mật
USER node

# Chỉ copy các file cần thiết từ Stage 1 sang
COPY --chown=node:node package*.json ./
RUN npm ci --only=production

COPY --chown=node:node --from=builder /app/dist ./dist

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## 5. Quản Lý Dữ Liệu & Mạng Trong Docker

### 5.1. Bản Chất Phù Du (Ephemeral) & Docker Volumes
Mặc định, khi một Container bị xóa bỏ (`docker rm`), **toàn bộ dữ liệu được sinh ra bên trong container đó sẽ biến mất vĩnh viễn**!
- Nếu bạn chạy CSDL PostgreSQL trong container, khi nâng cấp container sang phiên bản mới, toàn bộ tài khoản và đơn hàng của khách hàng sẽ bị xóa sạch!

#### Giải pháp: Docker Volumes
Volume là một vùng lưu trữ nằm trên ổ đĩa của máy chủ host (ngoài container), do Docker quản lý:
```bash
# Tạo volume có tên 'pgdata'
docker volume create pgdata

# Gắn volume vào thư mục lưu trữ dữ liệu của PostgreSQL
docker run -d \
  --name my-postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16
```
Bây giờ, kể cả khi bạn xóa container `my-postgres` đi, dữ liệu vẫn nằm nguyên vẹn trong `pgdata`!

---

### 5.2. Mạng Trong Docker (Docker Networks)
Khi bạn chạy một container Backend và một container Database trên cùng một máy chủ, làm sao Backend có thể kết nối tới Database?
- Hãy đưa cả hai container vào cùng một mạng **Bridge Network**:
```bash
# 1. Tạo mạng nội bộ
docker network create my-app-network

# 2. Khởi chạy Database trong mạng đó với tên '--name db'
docker run -d --name db --network my-app-network postgres:16

# 3. Khởi chạy Backend trong cùng mạng:
# Backend có thể kết nối thẳng tới Database bằng tên miền 'db:5432'
# thông qua hệ thống DNS nội bộ của Docker!
docker run -d --name api --network my-app-network -p 3000:3000 my-api-image
```

---

## 6. Cheatsheet Các Câu Lệnh Docker Thần Thánh

| Lệnh | Ý nghĩa |
| :--- | :--- |
| `docker build -t my-app:v1 .` | Đóng gói mã nguồn thư mục hiện tại thành Image có tên `my-app` |
| `docker run -d -p 8080:80 --name web nginx` | Khởi chạy container ngầm (`-d`), ánh xạ cổng host 8080 vào cổng 80 của container |
| `docker ps` | Xem danh sách các container đang chạy (`-a` để xem cả container đã tắt) |
| `docker logs -f <tên-container>` | Xem log thời gian thực của container |
| `docker exec -it <tên-container> sh` | Mở Terminal tương tác (`-it`) đi thẳng vào bên trong container |
| `docker stop <tên-container>` | Dừng hoạt động của một container |
| `docker rm <tên-container>` | Xóa bỏ một container đã dừng |
| `docker rmi <tên-image>` | Xóa bỏ một image khỏi máy tính |
| `docker system prune -a --volumes` | Dọn dẹp sạch sẽ toàn bộ các container chết, image rác và volume thừa để giải phóng ổ cứng |

---

## 7. Tóm Tắt & Ghi Nhớ Nhanh

1. **Docker Container** dùng chung Linux Kernel với máy chủ, khởi động tức thì trong vài mili-giây và nhẹ hơn Máy ảo (VM) gấp hàng chục lần.
2. Nắm vững mối quan hệ: **`Dockerfile`** (Công thức) ──► **Image** (Bản đóng gói chỉ đọc) ──► **Container** (Thực thể đang chạy).
3. Luôn tận dụng **Layer Caching** bằng cách `COPY package.json` và cài thư viện trước khi copy mã nguồn.
4. Sử dụng **Multi-stage builds** để tách biệt môi trường build và runtime, giúp giảm dung lượng image từ hàng Gigabyte xuống còn vài chục Megabyte.
5. Luôn gắn **Docker Volumes** cho các dịch vụ có trạng thái lưu trữ (Database, Redis, MinIO) để không bao giờ bị mất dữ liệu.
