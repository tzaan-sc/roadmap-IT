# Level 2 — Git + Linux + Database (Bộ Ba Công Cụ Cốt Lõi)

> *"Một lập trình viên giỏi không chỉ biết viết code. Họ biết cách lưu trữ lịch sử mã nguồn (Git), điều khiển cỗ máy thực thi ứng dụng (Linux) và tổ chức dữ liệu một cách bền vững (Database)."*

Chào mừng bạn đến với **Level 2** trong lộ trình [Roadmap IT](../README.md). Đây là giai đoạn bạn bước ra khỏi các bài tập thuật toán cục bộ trên máy tính cá nhân để làm quen với **môi trường kỹ thuật thực tế** mà mọi dự án phần mềm chuyên nghiệp trên thế giới đều dựa vào.

---

## 🗺️ Tam Giác Sức Mạnh Kỹ Sư Phần Mềm (The Core Engineering Triangle)

```text
                                  [ GIT & GITHUB ]
                            (Quản lý & Hợp tác mã nguồn)
                                    ▲        ▲
                                   /          \
                       Pull Request            Triển khai code
                       Code Review             CI/CD Pipeline
                                 /              \
                                ▼                ▼
       [ LINUX SYSTEM ] ◄────────────────────────────────► [ DATABASE / SQL ]
   (Môi trường máy chủ Server,                     (Nơi lưu trữ bền vững,
    Systemd, SSH, Nginx, Docker)                    Toàn vẹn ACID, B-Tree Index)
```

- **Git & GitHub:** Giúp bạn bảo vệ mã nguồn, quay ngược thời gian khi có sự cố và phối hợp nhịp nhàng trong đội ngũ hàng chục lập trình viên.
- **Linux:** Là nền tảng hệ điều hành máy chủ và đám mây (Cloud) nơi mã nguồn của bạn được triển khai thực tế.
- **Database (SQL):** Trái tim của mọi ứng dụng thương mại, nơi tiền bạc, tài khoản và thông tin người dùng được lưu trữ an toàn tuyệt đối.

---

## 📚 Danh Sách Các Bài Học Chi Tiết

Dưới đây là 5 chuyên đề chuyên sâu đã được biên soạn hoàn chỉnh trong thư mục này:

| STT | Tên bài học | Nội dung trọng tâm | Đường dẫn |
| :---: | :--- | :--- | :---: |
| **01** | **Làm Chủ Git & GitHub** | Kiến trúc 4 vùng làm việc (Working Dir, Staging, Local Repo, Remote), Branching, Fast-forward vs 3-Way Merge, So sánh sâu `merge` vs `rebase`, Xử lý xung đột (Merge Conflict), Bộ lệnh cứu hộ (`stash`, `reset`, `revert`), Quy trình GitHub Pull Request chuyên nghiệp. | [Xem bài viết](./01-git-and-github-mastery.md) |
| **02** | **Quản Trị Hệ Thống Linux Thực Chiến** | Quản lý dịch vụ với `systemd` & `systemctl`, Tự tạo file `.service` cho ứng dụng chạy ngầm 24/7 tự phục hồi, Kết nối an toàn không cần pass qua `SSH Key` (`ed25519`), Quản lý tiến trình ngầm (`nohup`, `&`), Tìm và giải phóng Port mạng (`lsof`, `ss`), Lệnh `curl` và Tường lửa `ufw`. | [Xem bài viết](./02-linux-system-administration.md) |
| **03** | **Cơ Sở Dữ Liệu Quan Hệ & SQL** | So sánh RDBMS vs NoSQL, Cú pháp SQL chuẩn (DDL & DML/CRUD), Lọc và gom nhóm (`WHERE`, `GROUP BY`, `HAVING`), Làm chủ các phép nối bảng (`INNER JOIN`, `LEFT JOIN`, `FULL JOIN`), Thiết kế quan hệ (1-1, 1-N, N-N với bảng trung gian), Chuẩn hóa dữ liệu 1NF - 3NF. | [Xem bài viết](./03-sql-and-relational-database.md) |
| **04** | **Giao Dịch, Chỉ Mục & Tối Ưu Hóa** | Bản chất của Transaction và 4 thuộc tính sống còn **ACID**, Cú pháp `COMMIT`/`ROLLBACK`, Cấu trúc cây B-Tree Indexing, Phân tích cái giá của Index (đánh đổi giữa Đọc và Ghi), Chẩn đoán hiệu năng truy vấn bằng `EXPLAIN ANALYZE`, Các lỗi kinh điển làm vô hiệu hóa Index. | [Xem bài viết](./04-transactions-indexing-optimization.md) |
| **05** | **Thực Hành & Ôn Tập Phỏng Vấn** | **3 Bài Lab Thực Chiến:** Lab tự tạo và giải quyết Git Conflict, Lab tạo Systemd Daemon chạy ngầm trên Linux, Lab thiết kế CSDL E-commerce & viết truy vấn doanh thu nhiều bảng. **Bộ 15 câu hỏi phỏng vấn kỹ thuật cốt lõi** có lời giải chi tiết và Checklist tự đánh giá. | [Xem bài viết](./05-practice-labs-and-interview.md) |

---

## 🎯 Mục Tiêu Đạt Được Sau Khi Hoàn Thành Level 2

1. **Không còn run sợ khi gặp Git Conflict:** Tự tin tạo nhánh tính năng, viết commit chuẩn mực và thành thạo quy trình làm việc nhóm qua GitHub PR.
2. **Làm chủ máy chủ Linux:** Tự tin SSH vào bất kỳ server Ubuntu nào trên AWS/GCP, biết cách thiết lập service tự khởi động, xem log thời gian thực và quản lý tường lửa.
3. **Thành thạo ngôn ngữ SQL:** Tự tin thiết kế bảng chuẩn hóa 3NF, kết nối dữ liệu nhiều bảng bằng JOIN và viết các báo cáo tổng hợp phức tạp.
4. **Tư duy tối ưu hóa Backend:** Hiểu được cơ chế Index B-Tree hoạt động thế nào để tăng tốc ứng dụng, và biết cách dùng Transaction để bảo vệ sự toàn vẹn của dữ liệu tiền tệ.

---

## ⏭️ Bước Tiếp Theo
Sau khi vượt qua các bài lab và tích trọn bộ Checklist trong bài 05, bạn đã có một bệ phóng cực kỳ vững chãi để bước sang:
👉 **[Level 3: Web / Application Development](../03-web-application-development/README.md)**
