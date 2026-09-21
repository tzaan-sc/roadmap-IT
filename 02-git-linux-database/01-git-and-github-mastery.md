# 01 - Làm Chủ Git & GitHub (Git & GitHub Mastery)

> **Mục tiêu bài học:** Thấu hiểu bản chất kiến trúc 4 vùng làm việc của Git, làm chủ các kỹ thuật phân nhánh, phân biệt sâu sắc giữa `merge` và `rebase`, tự tin xử lý xung đột (conflict), sử dụng các lệnh cứu nguy (`reset`, `revert`, `stash`) và thành thạo quy trình làm việc nhóm chuẩn quốc tế qua GitHub Pull Request.

---

## 1. Git Là Gì? Tại Sao Git Thống Trị Thế Giới?

**Git** là hệ thống quản lý phiên bản phân tán (**Distributed Version Control System - DVCS**) được Linus Torvalds tạo ra vào năm 2005 để quản lý mã nguồn hệ điều hành Linux.

```text
HỆ THỐNG TẬP TRUNG CŨ (SVN, CVS)             HỆ THỐNG PHÂN TÁN (GIT)
       ┌────────────────┐                          ┌────────────────┐
       │ Máy chủ Server │                          │ Máy chủ Remote │
       │ (Chứa Lịch sử) │                          │ (GitHub/GitLab)│
       └───────┬────────┘                          └───────┬────────┘
        ▲      │      ▲                             ▲      │      ▲
        │      │      │                             │      │      │ Clone toàn bộ
   Commit      │    Commit                     Push │      │ Pull │ Lịch sử về máy!
        │      ▼      │                             │      ▼      │
   ┌────┴──┐ ┌───┴───┐ ┌─────┴──┐              ┌────┴──┐ ┌───┴───┐ ┌─────┴──┐
   │ Dev 1 │ │ Dev 2 │ │ Dev 3  │              │ Dev 1 │ │ Dev 2 │ │ Dev 3  │
   │ (Chỉ  │ │ (Chỉ  │ │ (Chỉ   │              │(Kho đầy│ │(Kho đầy│ │(Kho đầy│
   │có code│ │có code│ │có code)│              │ đủ Repo│ │ đủ Repo│ │ đủ Repo│
   └───────┘ └───────┘ └────────┘              └───────┘ └───────┘ └────────┘
 (Server sập là ngừng làm việc!)             (Mỗi máy tính là 1 bản backup hoàn chỉnh!)
```

### Ưu điểm vượt trội của Git:
1. **Phân tán (Distributed):** Mỗi lập trình viên đều có một bản sao hoàn chỉnh của toàn bộ lịch sử dự án trên máy tính cá nhân. Bạn có thể commit, tạo branch, xem log hoàn toàn offline khi mất mạng.
2. **Hiệu năng siêu tốc:** Hầu hết mọi thao tác đều thực hiện trên đĩa cứng cục bộ, không phụ thuộc vào độ trễ mạng.
3. **Tính toàn vẹn dữ liệu:** Mọi tệp tin và commit đều được mã hóa bằng hàm băm mật mã học **SHA-1 / SHA-256**. Lịch sử dự án không thể bị sửa đổi lén lút mà không làm thay đổi mã hash.

---

## 2. Kiến Trúc 4 Vùng Nhớ Cốt Lõi Của Git

Đây là mô hình quan trọng nhất bạn cần khắc ghi trong đầu:

```text
                                  CHU TRÌNH LÀM VIỆC CỦA GIT
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│ WORKING DIRECTORY│       │  STAGING AREA   │       │LOCAL REPOSITORY │       │REMOTE REPOSITORY│
│ (Thư mục làm    │       │     (INDEX)     │       │     (HEAD)      │       │ (GitHub/GitLab) │
│  việc thực tế)  │       │ (Khu chuẩn bị)  │       │(Kho commit máy) │       │ (Kho trên mây)  │
└────────┬────────┘       └────────┬────────┘       └────────┬────────┘       └────────┬────────┘
         │                         │                         │                         │
         │──── git add <file> ────►│                         │                         │
         │                         │──── git commit -m ────►│                         │
         │                         │                         │─────── git push ───────►│
         │                         │                         │◄────── git fetch ───────│
         │◄─────────────────────── git pull ───────────────────────────────────────────│
         │◄── git checkout/restore ──────────────────────────│                         │
```

### 2.1. Giải thích 4 khu vực:
1. **Working Directory (Thư mục làm việc):** Nơi bạn mở VS Code, gõ phím, thêm sửa xóa file thực tế trên ổ cứng.
2. **Staging Area / Index (Khu vực chuẩn bị):** Một file đánh chỉ mục lưu trữ tạm thời những thay đổi mà bạn chọn lọc để chuẩn bị đưa vào lần commit tiếp theo.
3. **Local Repository (`.git` folder):** Kho lưu trữ chính thức nằm ngay trên máy của bạn, chứa toàn bộ các commit (ảnh chụp trạng thái dự án theo thời gian).
4. **Remote Repository:** Kho lưu trữ mã nguồn chia sẻ trên đám mây (GitHub, GitLab, Bitbucket) để làm việc nhóm.

### 2.2. Vòng đời của một tệp tin (File Lifecycle)

```text
Untracked (Chưa theo dõi) ──► git add ──► Staged (Đã chuẩn bị)
                                                │
                                           git commit
                                                ▼
Modified (Bị chỉnh sửa)   ◄── Sửa file ── Unmodified (Chưa sửa)
```

---

## 3. Các Lệnh Git Căn Bản Hàng Ngày

### 3.1. Khởi tạo và Thiết lập ban đầu
```bash
# Thiết lập danh tính (Chỉ cần làm 1 lần khi cài Git)
git config --global user.name "Nguyen Van A"
git config --global user.email "nguyenvana@gmail.com"

# Khởi tạo một Git repository mới trong thư mục hiện tại
git init

# Hoặc tải một dự án có sẵn từ GitHub về máy
git clone https://github.com/user/project.git
```

### 3.2. Chu trình lưu trữ code
```bash
# Xem trạng thái các file (đang ở vùng nào: untracked, modified, hay staged)
git status

# Đưa 1 file cụ thể vào Staging Area
git add index.html

# Đưa toàn bộ các file thay đổi vào Staging Area
git add .

# Tạo một commit chính thức với thông điệp rõ ràng
git commit -m "feat: thêm giao diện đăng nhập cho người dùng"

# Đẩy các commit từ máy cá nhân lên GitHub nhánh main
git push origin main
```

### 3.3. Đồng bộ dữ liệu từ Remote: `git fetch` vs `git pull`
- **`git fetch`:** Tải toàn bộ các commit mới từ GitHub về máy cá nhân nhưng **chưa hợp nhất vào mã nguồn bạn đang viết**. Rất an toàn để kiểm tra xem đồng đội đã làm gì.
- **`git pull`:** Thực hiện liên tiếp 2 việc: `git fetch` rồi tự động `git merge` ngay vào nhánh hiện tại của bạn.

---

## 4. Quản Lý Nhánh (Branching) & Hợp Nhất (Merging)

Nhánh (Branch) trong Git thực chất chỉ là **một con trỏ 40-byte siêu nhẹ** trỏ vào mã hash của commit cuối cùng. Do đó, việc tạo một nhánh mới trong Git diễn ra tức thì trong $O(1)$.

```text
                      (Nhánh feature/login)
                               ┌───► [ Commit C ] ───► [ Commit D ]
                              │
[ Commit A ] ───► [ Commit B ] ◄── HEAD trỏ vào main ban đầu
                      (Nhánh main)
```

### 4.1. Thao tác với nhánh
```bash
# Xem danh sách nhánh hiện có (* là nhánh đang đứng)
git branch

# Tạo nhánh mới và nhảy sang nhánh đó ngay lập tức (Cú pháp hiện đại)
git switch -c feature/login
# Hoặc cú pháp cũ:
git checkout -b feature/login

# Quay trở lại nhánh main
git switch main
```

### 4.2. Hai cơ chế Hợp Nhất (Merge Types)

1. **Fast-Forward Merge:**
   - Xảy ra khi nhánh `main` chưa có bất kỳ commit mới nào kể từ khi nhánh `feature` được rẽ ra. Git chỉ cần "kéo dịch" con trỏ `main` tiến về phía trước. Lịch sử commit là một đường thẳng tắp.
2. **3-Way Merge (True Merge):**
   - Xảy ra khi cả `main` lẫn `feature` đều có những commit mới riêng rẽ. Git sẽ tìm commit tổ tiên chung (Common Ancestor) và tự động tạo ra một commit đặc biệt gọi là **Merge Commit** (có 2 commit cha).

```bash
# Đang đứng ở nhánh main, hợp nhất nhánh feature vào main:
git merge feature/login
```

---

## 5. So Sánh Kinh Điển: `git merge` vs `git rebase`

Đây là chủ đề phỏng vấn kỹ thuật phổ biến bậc nhất và là nguồn cơn của nhiều tranh cãi trong các đội ngũ phát triển:

```text
TÌNH HUỐNG BAN ĐẦU:
            C ─── D (feature)
           /
A ─── B ─── E (main có commit E mới)

CÁCH 1: git merge feature vào main
            C ─── D
           /       \
A ─── B ─── E ────── M (Tạo một Merge Commit M, lịch sử phân nhánh rõ ràng)

CÁCH 2: git rebase main trên nhánh feature
A ─── B ─── E ─── C' ─── D' (Bứng toàn bộ commit C, D ghép vào sau E)
                             (Lịch sử biến thành MỘT ĐƯỜNG THẲNG HOÀN HẢO!)
```

| Tiêu chí | `git merge` | `git rebase` |
| :--- | :--- | :--- |
| **Bản chất** | Giữ nguyên mọi commit, tạo thêm 1 Merge Commit kết nối | Viết lại lịch sử (Rewrite history) bằng cách tạo các commit mới ($C', D'$) |
| **Ưu điểm** | Bảo toàn 100% lịch sử nguyên gốc, không can thiệp hash cũ | Lịch sử Git sạch sẽ, thẳng tắp, cực kỳ dễ đọc bằng `git log` |
| **Nhược điểm** | Lịch sử trông rối như "mạng nhện" khi có hàng chục dev cùng merge | Có thể làm sai lệch thời gian commit thật, nguy hiểm nếu dùng sai |

> [!CAUTION]
> **Quy tắc vàng của Git Rebase (The Golden Rule of Rebase):**
> **TUYỆT ĐỐI KHÔNG REBASE trên các nhánh công khai (Public Branches như `main` hoặc `develop`)** mà người khác đang cùng làm việc. Chỉ rebase trên nhánh cá nhân của riêng bạn trước khi tạo Pull Request!

---

## 6. Xử Lý Xung Đột (Merge Conflicts)

Xung đột xảy ra khi hai lập trình viên **cùng chỉnh sửa trên cùng một dòng code trong cùng một file** và Git không thể tự đoán biết nên giữ lại code của ai.

Khi đó, Git sẽ dừng lại và đánh dấu file bị xung đột như sau:

```text
<<<<<<< HEAD (Mã nguồn hiện tại của bạn)
const API_URL = "https://api.staging.mysite.com";
=======
const API_URL = "https://api.production.mysite.com";
>>>>>>> feature/update-api (Mã nguồn từ nhánh đang muốn gộp vào)
```

### Các bước giải quyết Conflict:
1. Mở file bị báo lỗi trong VS Code (VS Code sẽ hiển thị các nút bấm: *Accept Current Change*, *Accept Incoming Change*, hoặc *Accept Both*).
2. Thảo luận với đồng đội hoặc tự quyết định chọn đoạn code đúng.
3. Xóa sạch các ký tự đánh dấu `<<<<<<<`, `=======`, `>>>>>>>`.
4. Lưu file lại.
5. Đánh dấu đã sửa xong và hoàn tất merge:
   ```bash
   git add <tên-file-vừa-sửa>
   git commit -m "fix: giải quyết xung đột cấu hình API URL"
   ```

---

## 7. Bộ Lệnh "Cứu Hộ" Trong Git

### 7.1. `git stash` - Cất tạm đồ nghề
Khi bạn đang làm dở tính năng thì sếp yêu cầu sửa gấp một bug nghiêm trọng trên `main`, nhưng bạn chưa muốn commit những dòng code dở dang:
```bash
# Cất toàn bộ thay đổi chưa commit vào ngăn kéo bí mật
git stash

# Chuyển nhánh sửa bug, commit và deploy xong...
# Quay lại nhánh cũ và lấy lại code dang dở:
git stash pop
```

### 7.2. Hủy bỏ commit: `git reset` vs `git revert`

```text
git reset --soft HEAD~1  ──► Hủy commit, giữ nguyên code ở STAGING AREA
git reset --mixed HEAD~1 ──► Hủy commit, đưa code về WORKING DIRECTORY (Mặc định)
git reset --hard HEAD~1  ──► HỦY SẠCH SẼ TOÀN BỘ CODE, KHÔNG THỂ KHÔI PHỤC!
```

- **`git reset` (Tua ngược thời gian):** Xóa bỏ các commit khỏi lịch sử. Chỉ an toàn khi commit đó **chưa hề push lên GitHub**.
- **`git revert` (Bảo đảm an toàn):** Tạo ra một **commit mới có nội dung đảo ngược lại hoàn toàn commit lỗi**. Lịch sử vẫn nguyên vẹn, cực kỳ an toàn cho các nhánh đã push lên server công khai.

---

## 8. Quy Trình Làm Việc Nhóm Chuẩn Qua GitHub Pull Request (PR)

Trong các công ty chuyên nghiệp, không ai được phép push code thẳng vào nhánh `main`. Mọi tính năng đều phải đi qua quy trình:

```text
[ GITHUB: Repo Công Ty ]
           │
           ▼ (1. Fork hoặc Tạo nhánh mới)
[ Nhánh: feature/shopping-cart ]
           │
           ├─► Viết code, commit theo chuẩn Conventional Commits
           ├─► git push origin feature/shopping-cart
           │
           ▼ (2. Mở PULL REQUEST - PR trên GitHub)
[ PULL REQUEST #42 ]
           │
           ├─► Chạy kiểm thử tự động (CI/CD Pipeline)
           ├─► Đồng đội vào Review Code & Comment góp ý
           ├─► Sửa code theo yêu cầu review cho đến khi được Approve ✅
           │
           ▼ (3. Bấm MERGE PULL REQUEST)
[ GITHUB: Nhánh main được tích hợp tính năng mới an toàn! ]
```

### Chuẩn viết thông điệp Commit chuyên nghiệp (Conventional Commits):
- `feat: thêm chức năng lọc sản phẩm theo giá`
- `fix: sửa lỗi crash ứng dụng khi người dùng nhập email rỗng`
- `docs: cập nhật hướng dẫn cài đặt trong README.md`
- `refactor: tái cấu trúc hàm tính thuế không làm thay đổi hành vi`
- `test: bổ sung unit test cho chức năng thanh toán`

---

## 9. Tóm Tắt & Ghi Nhớ Nhanh

1. Nắm vững **4 khu vực**: Working Directory → Staging Area → Local Repo → Remote Repo.
2. Dùng `git switch -c <name>` để tạo nhánh; tách biệt mỗi tính năng vào một nhánh riêng.
3. Hiểu rõ **Merge** (giữ nguyên lịch sử, an toàn) và **Rebase** (lịch sử thẳng tắp, không dùng cho public branch).
4. Khi gặp **Conflict**, bình tĩnh mở file, xóa các ký hiệu `<<<`, `===`, `>>>`, kiểm tra lại code rồi `git add` và commit.
5. Luôn làm việc theo quy trình **Pull Request** và tuân thủ quy tắc viết commit thông điệp rõ ràng.
