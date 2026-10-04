# Bài Tập Thực Hành DevOps - Session 5

- **Học viên:** Nguyễn Mạnh Hùng
- **Repository:** [NguyenManhHung_DevOps_ss5](https://github.com/HigaWateru/NguyenManhHung_DevOps_ss5)

---

## Danh Sách Bài Tập

### 1. [Bài Tập 1: Khôi Phục Commit Đã Mất Bằng Git Reflog](./ss5_1/README.md)
- **Thư mục:** [`ss5_1/`](./ss5_1/)
- **Mục tiêu:** Hiểu cơ chế Reflog, truy vết lịch sử commit mồ côi (dangling / unreachable commit) và phục hồi an toàn sau khi chạy nhầm lệnh nguy hiểm `git reset --hard`.
- **Nội dung:**
  - Báo cáo chi tiết: [`ss5_1/README.md`](./ss5_1/README.md)
  - File dữ liệu: [`app.txt`](./ss5_1/app.txt), [`feature.txt`](./ss5_1/feature.txt)

---

### 2. [Bài Tập 4: Mô Phỏng Quy Trình Hotfix & Gitflow Thực Tế](./ss5_4/README.md)
- **Thư mục:** [`ss5_4/`](./ss5_4/)
- **Mục tiêu:** Nắm vững mô hình Gitflow chuẩn, xử lý sự cố Production an toàn qua nhánh `hotfix/v1.0.1` xuất phát từ tag `v1.0.0` trên `main`, và đồng bộ hóa mã nguồn ngược lại nhánh `develop` (Back-merge).
- **Nội dung:**
  - Báo cáo chi tiết: [`ss5_4/README.md`](./ss5_4/README.md)
  - Mã nguồn: [`server.js`](./ss5_4/server.js), [`cart.js`](./ss5_4/cart.js), [`payment.js`](./ss5_4/payment.js)
