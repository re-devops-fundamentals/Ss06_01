# Báo Cáo Bài Tập 1: Phân Quyền Thư Mục Và Quyền Sở Hữu (chmod, chown) Cho Ứng Dụng Web

- **Môn học**: DevOps Fundamentals (IT-209)
- **Session**: 06 - Linux Administration, Permissions & Process Management
- **Bài tập**: Bài 1 - Phân quyền thư mục và quyền sở hữu cho ứng dụng Web
- **Đường dẫn nộp bài trên GitHub**: `homework/session_06/ex1/`
- **Học viên**: HoQuocKhai
- **Email**: khaiho6040@gmail.com

---

## 1. Mục tiêu bài thực hành
1. **Thiết lập quyền truy cập tệp và thư mục bằng `chmod`**: Nắm vững cơ chế tính toán giá trị phân quyền theo hệ cơ số 8 (Octal Notation) và phân bổ quyền `rwx` cho Owner, Group, Others.
2. **Quản lý quyền sở hữu User & Group bằng `chown`**: Biết cách thay đổi người sở hữu và nhóm sở hữu đệ quy (`-R`) cho toàn bộ cây thư mục dự án.
3. **Áp dụng nguyên tắc bảo mật cho ứng dụng Web đa người dùng**: Đảm bảo máy chủ web (nhóm `www-data`) có quyền đọc mã nguồn `public`, quyền ghi log vào `logs`, đồng thời ngăn chặn tuyệt đối người dùng khác (`others`) can thiệp vào mã nguồn và dữ liệu nhật ký.

---

## 2. Cơ sở lý thuyết & Phân tích quyền Octal

### 2.1. Bảng đối chiếu quyền truy cập Linux (Octal Permission)
| Ký hiệu | Giá trị nhị phân | Giá trị Octal | Ý nghĩa với Tệp tin (File) | Ý nghĩa với Thư mục (Directory) |
| :---: | :---: | :---: | :--- | :--- |
| **`r`** | `100` | **4** | Cho phép xem nội dung tệp. | Cho phép liệt kê danh sách tệp (`ls`). |
| **`w`** | `010` | **2** | Cho phép chỉnh sửa/ghi tệp. | Cho phép tạo mới, xóa, đổi tên tệp bên trong. |
| **`x`** | `001` | **1** | Cho phép thực thi chương trình/script. | Cho phép truy cập vào thư mục (`cd`) và mở file con. |

### 2.2. Phân tích quyền cấu hình cho ứng dụng `/var/www/my-app`
- **Thư mục `/var/www/my-app/public` (`750` = `rwxr-x---`)**:
  - **Owner (`7` = 4+2+1 = `rwx`)**: User quản trị (`devops`) có toàn quyền tạo, chỉnh sửa, xóa mã nguồn.
  - **Group (`5` = 4+0+1 = `r-x`)**: Nhóm dịch vụ web (`www-data`) có quyền đọc (`r`) và truy cập (`x`) để phục vụ nội dung ra ngoài cho người dùng, nhưng **không có quyền ghi/sửa** (`w`), ngăn chặn nguy cơ deface/sửa file nếu ứng dụng bị lộ lỗ hổng.
  - **Others (`0` = `---`)**: Các tài khoản khác trong hệ điều hành hoàn toàn không có bất kỳ quyền nào.
- **Thư mục `/var/www/my-app/logs` (`770` = `rwxrwx---`)**:
  - **Owner (`7` = `rwx`)**: User quản trị có toàn quyền đọc và dọn dẹp log.
  - **Group (`7` = `rwx`)**: Nhóm web server (`www-data`) có toàn quyền đọc và **ghi nhật ký truy cập/lỗi** (`access.log`, `error.log`).
  - **Others (`0` = `---`)**: Bị cô lập hoàn toàn, không thể đọc trộm log hệ thống.

---

## 3. Quá trình thực hiện chi tiết

### Bước 1: Tạo cấu trúc thư mục ứng dụng
Sử dụng cờ `-p` để tự động tạo cây thư mục cha `/var/www/my-app/` cùng các thư mục con:
```bash
sudo mkdir -p /var/www/my-app/public
sudo mkdir -p /var/www/my-app/logs
```

---

### Bước 2: Thiết lập quyền truy cập hạn chế (chmod)
Thiết lập quyền `750` cho thư mục chứa mã nguồn công khai và `770` cho thư mục lưu trữ nhật ký:
```bash
sudo chmod 750 /var/www/my-app/public
sudo chmod 770 /var/www/my-app/logs
```

---

### Bước 3: Phân chia quyền sở hữu User và Group (chown)
Gán quyền sở hữu người dùng cho `$USER` (`devops`) và nhóm quản lý dịch vụ web `www-data` đệ quy trên toàn bộ thư mục dự án:
```bash
sudo chown -R $USER:www-data /var/www/my-app
```

---

## 4. Kết quả kiểm tra thực tế trên hệ thống

Thực thi lệnh kiểm tra chi tiết danh sách tệp và quyền hạn trong thư mục `/var/www/my-app`:
```bash
devops@1161186107523368:~$ ls -la /var/www/my-app
```

**Log đầu ra thực tế từ máy chủ:**
```text
total 16
drwxr-xr-x 4 devops www-data 4096 Oct  7 11:43 .
drwxr-xr-x 4 root   root     4096 Oct  7 11:43 ..
drwxrwx--- 2 devops www-data 4096 Oct  7 11:43 logs
drwxr-x--- 2 devops www-data 4096 Oct  7 11:43 public
```

---

## 5. Đánh giá và Kết luận
- **Quyền hạn chính xác**:
  - `public` mang quyền `drwxr-x---` (tương ứng `750`).
  - `logs` mang quyền `drwxrwx---` (tương ứng `770`).
- **Quyền sở hữu chuẩn xác**: Cả hai thư mục đều thuộc sở hữu của User `devops` và Group `www-data`.
- Cấu hình đáp ứng trọn vẹn nguyên tắc bảo mật tối thiểu (*Principle of Least Privilege*) cho ứng dụng Web trên Linux.