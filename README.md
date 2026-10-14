# HƯỚNG DẪN TRIỂN KHAI VÀ THỰC NGHIỆM LAB DOCKER
## BÀI TẬP LỚN: AN TOÀN MẠNG (INT1482)
**Học viện Công nghệ Bưu chính Viễn thông (PTIT)**  
**Đề tài:** An toàn dịch vụ truyền nhận tệp (FTP vs FTPS/SFTP - CrushFTP 10.6.0)  
**Nhóm thực hiện:** Nhóm 05 - Lớp: INT1482-20261-02  
**Giảng viên hướng dẫn:** TS. Nguyễn Ngọc Điệp  

---

## 1. THÀNH PHẦN TRONG THƯ MỤC
Thư mục này đã được đóng gói sẵn để chạy trực tiếp trên bất kỳ máy tính nào:
- `nhom05_atm_lab.tar`: Tệp ảnh Docker chứa hệ điều hành và cấu hình của cả 2 máy (Ubuntu Server + Kali Linux). *(Tải tại mục [Releases](https://github.com/Snoopyz1/Lab_atm/releases) của repository này)*.
- `docker-compose.yml`: Tệp cấu hình mạng nội bộ `192.168.10.0/24` và ánh xạ cổng.
- `README.md`: Hướng dẫn chi tiết cách chạy và demo.

---

## 2. YÊU CẦU TIÊN QUYẾT
- Máy tính đã cài đặt **Docker Desktop** (trên Windows / macOS) hoặc **Docker Engine** (trên Linux).
- Docker Desktop đang ở trạng thái **Running** (Bật).

---

## 3. CÁCH KHỞI CHẠY LAB (CHỈ CẦN 2 LỆNH)

Mở PowerShell / CMD / Terminal tại thư mục `docker_lab`:

### Bước 1: Nạp image vào Docker (chỉ cần làm 1 lần)
```powershell
docker load -i nhom05_atm_lab.tar
```

### Bước 2: Khởi động 2 máy (Victim và Attacker)
```powershell
docker compose up -d
```

---

## 4. HƯỚNG DẪN KIỂM THỬ VÀ DEMO

### A. Mở giao diện Web máy chủ nạn nhân (CrushFTP 10.6.0)
- Mở trình duyệt trên máy thật, truy cập: `http://localhost:8080`
- Tài khoản quản trị khởi tạo sẵn:
  - **Tài khoản:** `crushadmin`
  - **Mật khẩu:** `admin123`

---

### B. Thực hiện tấn công từ máy Kali Linux
1. Mở terminal chui vào máy Kali Linux:
```powershell
docker exec -it attacker_kali bash
```

2. Kiểm tra thông mạng với máy chủ Ubuntu Server (`192.168.10.20`):
```bash
ping 192.168.10.20 -c 2
```

3. **Demo lỗ hổng CVE-2024-4040 (VFS Escape / Server-Side Template Injection):**
```bash
python3 scan_host.py http://192.168.10.20:8080
```
> **Kết quả kỳ vọng:** In ra `Vulnerable` (khai thác thành công đọc file hệ thống).

4. **Demo lỗ hổng CVE-2025-54309 (Race Condition Authentication Bypass):**
```bash
python3 watchTowr-vs-CrushFTP-CVE-2025-54309.py http://192.168.10.20:8080
```
> **Kết quả kỳ vọng:** Vượt qua xác thực và trích xuất danh sách tài khoản: `crushadmin`, `default`, `TempAccount`.

5. Thoát khỏi terminal Kali:
```bash
exit
```

---

### C. Xem máy chủ Ubuntu Server (Dịch vụ CrushFTP)
Nếu muốn kiểm tra máy chủ nạn nhân:
```powershell
docker exec -it victim_server bash
```
- Thư mục CrushFTP: `/var/opt/CrushFTP10`
- File cờ đánh dấu trong báo cáo: `cat /var/tmp/nhom05-cve4040.txt`
- Thoát ra: `exit`.

---

## 5. DỪNG VÀ GIẢI PHÓNG TÀI NGUYÊN LAB

Sau khi demo xong, tắt hệ thống:
```powershell
docker compose down
```
