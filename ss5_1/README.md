# Báo Cáo Bài Tập 1: Khôi Phục Commit Đã Mất Bằng Git Reflog

- **Môn học / Khóa học:** DevOps Engineering
- **Phiên học (Session):** Session 05
- **Bài tập:** Bài 1 - Khôi phục commit đã mất bằng Git Reflog
- **Đường dẫn nộp bài dự kiến:** `homework/session_05/ex1/`

---

## 1. Mục Tiêu Bài Học
- **Hiểu cơ chế Reflog:** Nắm vững cách Git ghi nhận và lưu trữ lịch sử dịch chuyển con trỏ `HEAD` cục bộ thông qua cơ chế Reference Logs (Reflog).
- **Truy vết lịch sử ẩn:** Sử dụng thành thạo lệnh `git reflog` để truy tìm các commit mồ côi (dangling / unreachable commit) vốn không còn hiển thị trên `git log`.
- **Phục hồi an toàn:** Khôi phục thành công mã nguồn và trạng thái commit đã bị mất sau khi chạy nhầm lệnh nguy hiểm `git reset --hard` mà không cần viết lại mã nguồn thủ công.

---

## 2. Bối Cảnh Sự Cố & Phân Tích Kỹ Thuật

### 2.1. Tình huống
Trong quá trình làm việc trên repository cục bộ, lập trình viên vô tình thực thi:
```bash
git reset --hard HEAD~1
```
Hậu quả:
1. Con trỏ nhánh hiện tại và con trỏ `HEAD` bị lùi lại 1 commit.
2. Cả **Staging Area (Index)** và **Working Directory** đều bị ghi đè về trạng thái của commit trước đó.
3. Commit quan trọng nhất chứa file `feature.txt` với nội dung `"Day la tinh nang quan trong."` bị biến mất khỏi nhánh, không còn xuất hiện khi chạy `git log`.

### 2.2. Bản chất bên trong của Git
- Khi chạy `git reset --hard HEAD~1`, Git **không hề xóa vĩnh viễn** đối tượng commit ngay lập tức.
- Commit đó chỉ bị mất liên kết (unreachable / orphaned commit) khỏi nhánh hiện tại.
- Mọi sự thay đổi của con trỏ `HEAD` (bao gồm `commit`, `checkout`, `reset`, `merge`, `rebase`,...) đều được Git âm thầm ghi lại trong file nhật ký tham chiếu cục bộ: `.git/logs/HEAD`.
- Do đó, ta hoàn toàn có thể sử dụng `git reflog` để tìm lại mã hash SHA-1 của commit bị mất và kéo nó trở về.

---

## 3. Các Bước Thực Hiện Chi Tiết

### Bước 1: Khởi tạo kho chứa và tạo commit mô phỏng

1. Khởi tạo repository Git mới:
   ```bash
   git init
   ```

2. Tạo commit ban đầu (gốc):
   ```bash
   echo "Khoi tao du an" > app.txt
   git add app.txt
   git commit -m "Khoi tao du an"
   ```
   *Kết quả mẫu:*
   ```text
   [master (root-commit) dd4b6ed] Khoi tao du an
    1 file changed, 1 insertion(+)
    create mode 100644 app.txt
   ```

3. Thêm tính năng quan trọng và commit thay đổi:
   ```bash
   echo "Day la tinh nang quan trong." > feature.txt
   git add .
   git commit -m "Them tinh nang quan trong"
   ```
   *Kết quả mẫu:*
   ```text
   [master 84ce97c] Them tinh nang quan trong
    1 file changed, 1 insertion(+)
    create mode 100644 feature.txt
   ```

4. Kiểm tra lịch sử commit bình thường:
   ```bash
   git log --oneline
   ```
   *Đầu ra hiển thị:*
   ```text
   84ce97c Them tinh nang quan trong
   dd4b6ed Khoi tao du an
   ```

---

### Bước 2: Giả lập sự cố mất commit bằng `git reset --hard`

Chạy lệnh lùi commit và xóa sạch working tree:
```bash
git reset --hard HEAD~1
```
*Đầu ra thông báo:*
```text
HEAD is now at dd4b6ed Khoi tao du an
```

**Kiểm tra xác nhận sự cố:**
- Kiểm tra lại bằng `git log --oneline`:
  ```text
  dd4b6ed Khoi tao du an
  ```
  *(Commit `84ce97c - Them tinh nang quan trong` đã biến mất hoàn toàn khỏi log)*
- Kiểm tra file trong thư mục làm việc:
  File `feature.txt` đã bị xóa sạch khỏi disk.

---

### Bước 3: Sử dụng `git reflog` để truy vết commit đã mất

Thực thi lệnh xem nhật ký chuyển dịch của `HEAD`:
```bash
git reflog
```

*Kết quả trả về từ terminal:*
```text
dd4b6ed HEAD@{0}: reset: moving to HEAD~1
84ce97c HEAD@{1}: commit: Them tinh nang quan trong
dd4b6ed HEAD@{2}: commit (initial): Khoi tao du an
```

**Giải thích kết quả Reflog:**
- **Cột 1 (`84ce97c`):** Mã hash 7 ký tự rút gọn của commit.
- **Cột 2 (`HEAD@{1}`):** Định danh tham chiếu con trỏ theo vị trí tương đối theo thời gian (`HEAD@{0}` là thao tác gần nhất, `HEAD@{1}` là thao tác trước đó 1 bước).
- **Cột 3 (`commit`):** Hành động đã tác động lên con trỏ.
- **Cột 4 (`Them tinh nang quan trong`):** Thông điệp commit ban đầu.

-> **Xác định mục tiêu phục hồi:** Mã hash của commit bị mất là **`84ce97c`** (tại tham chiếu `HEAD@{1}`).

---

### Bước 4: Khôi phục commit về lại nhánh làm việc

Có 2 phương pháp khôi phục phổ biến:

#### Cách 1: Phục hồi trực tiếp trên nhánh hiện tại bằng `git reset --hard` (Khuyên dùng cho bài này)
Đưa cả con trỏ nhánh và working directory trở lại đúng commit đã mất:
```bash
git reset --hard 84ce97c
# Hoặc có thể dùng cú pháp: git reset --hard HEAD@{1}
```

*Đầu ra thông báo:*
```text
HEAD is now at 84ce97c Them tinh nang quan trong
```

#### Cách 2: Khôi phục an toàn bằng cách tạo nhánh mới (`git checkout` / `git switch`)
Trong trường hợp đang làm việc trong môi trường thực tế và muốn bảo toàn nhánh hiện tại để kiểm tra trước:
```bash
# Tạo nhánh mới có tên recovery-feature từ commit 84ce97c
git checkout -b recovery-feature 84ce97c
# Hoặc với Git phiên bản mới:
git switch -c recovery-feature 84ce97c
```

---

### Bước 5: Kiểm tra và xác thực kết quả (Verification)

1. **Kiểm tra lại `git log --oneline`:**
   ```bash
   git log --oneline
   ```
   *Kết quả thực tế:*
   ```text
   84ce97c Them tinh nang quan trong
   dd4b6ed Khoi tao du an
   ```
   *(Commit đã xuất hiện trở lại ở vị trí đỉnh nhánh)*

2. **Kiểm tra nội dung file mã nguồn đã được khôi phục:**
   ```bash
   cat feature.txt
   ```
   *Nội dung hiển thị:*
   ```text
   Day la tinh nang quan trong.
   ```

3. **Kiểm tra lại toàn bộ nhật ký `git reflog`:**
   ```bash
   git reflog
   ```
   *Kết quả ghi nhận toàn bộ tiến trình:*
   ```text
   84ce97c HEAD@{0}: reset: moving to 84ce97c
   dd4b6ed HEAD@{1}: reset: moving to HEAD~1
   84ce97c HEAD@{2}: commit: Them tinh nang quan trong
   dd4b6ed HEAD@{3}: commit (initial): Khoi tao du an
   ```

---

## 4. Tổng Kết & Kiến Thức Bổ Sung Quan Trọng

### 4.1. Bảng so sánh giữa `git log` và `git reflog`

| Tiêu chí | `git log` | `git reflog` |
| :--- | :--- | :--- |
| **Phạm vi ghi nhận** | Lịch sử commit tuyến tính của nhánh hiện tại | Lịch sử dịch chuyển con trỏ `HEAD` trên máy cục bộ |
| **Commit bị mồ côi (Dangling)** | Không hiển thị | Vẫn hiển thị đầy đủ |
| **Phạm vi lưu trữ** | Toàn bộ repository (đồng bộ khi `push`/`pull`) | Chỉ tồn tại cục bộ trên máy lập trình viên (Local only) |
| **Mục đích chính** | Theo dõi tiến độ, lịch sử phát triển dự án | "Hộp đen cứu hộ" (Safety net) khi gặp sự cố lệnh |

### 4.2. Thời gian tồn tại của Reflog (Garbage Collection)
Dữ liệu trong Reflog không tồn tại mãi mãi mà sẽ được bộ dọn rác tự động của Git (`git gc`) thu hồi sau một khoảng thời gian:
- **Reachable commits:** Mặc định tồn tại trong **90 ngày** (`gc.reflogExpire = 90.days`).
- **Unreachable / Dangling commits (như commit vừa khôi phục):** Mặc định tồn tại trong **30 ngày** (`gc.reflogExpireUnreachable = 30.days`).

> **Khuyến cáo:** Khi lỡ chạy nhầm các lệnh làm mất commit (`reset --hard`, `rebase`, `commit --amend`, xóa nhánh nhầm), hãy bình tĩnh chạy `git reflog` ngay lập tức để lấy lại mã hash trước khi Git tiến hành dọn rác tự động.
