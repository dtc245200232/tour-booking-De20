# Hệ thống Booking Tour - DTC245200232 - Thèn Thành Đạt

Dự án triển khai và quản trị hệ thống ứng dụng Web Đặt Tour Du Lịch tự động hóa bằng Docker Compose, tích hợp Reverse Proxy Nginx (HTTPS), hệ thống giám sát thời gian thực Prometheus + Grafana và quản lý nhật ký tập trung Loki + Promtail.

---

## 🛠 Tech Stack
- **Web App:** Node.js 20, Express, EJS, PostgreSQL Driver (`pg`), Prom-Client
- **Database:** PostgreSQL 16 (Alpine), pgAdmin 4
- **Web Server / Reverse Proxy:** Nginx (Unprivileged, TLS 1.2/1.3, Security Headers)
- **Monitoring & Metrics:** Prometheus, Grafana, cAdvisor, Nginx-Exporter, Postgres-Exporter
- **Centralized Logging:** Grafana Loki, Promtail (LogQL)
- **Security Hardening:** Non-root user, Read-only rootfs, Drop all Linux capabilities, Network isolation (`db_net internal`)

---

## 🚀 Hướng dẫn Cài đặt & Khởi chạy Hệ thống

### Bước 1: Clone Repository từ GitHub
Mở terminal trên máy tính của bạn và thực hiện clone dự án:
```bash
git clone https://github.com/dtc245200232/tour-booking.git
cd tour-booking
```

### Bước 2: Cấu hình biến môi trường
Tạo file `.env` từ file mẫu `.env.example`:
```bash
cp .env.example .env
```
> *(Tùy chọn: Bạn có thể mở file `.env` để kiểm tra hoặc điều chỉnh mật khẩu Database, pgAdmin, Grafana hoặc tài khoản Admin nếu muốn).*

### Bước 3: Khởi tạo chứng chỉ SSL cho Nginx (HTTPS)
Tạo thư mục chứng chỉ và sinh chứng chỉ tự ký (Self-signed Certificate):
```bash
mkdir -p nginx/certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx/certs/server.key -out nginx/certs/server.crt -subj "//CN=localhost"
```

### Bước 4: Khởi chạy toàn bộ hệ thống bằng Docker Compose
Chạy lệnh sau để build image và khởi động ngầm toàn bộ 11 container:
```bash
docker compose up -d --build
```

### Bước 5: Kiểm tra trạng thái hoạt động
Đảm bảo tất cả các container đều ở trạng thái `Up` (hoặc `healthy`):
```bash
docker compose ps
```

---

## 🌐 Danh sách Dịch vụ & Tài khoản Truy cập

| Dịch vụ | URL | Thông tin Đăng nhập |
|---|---|---|
| **Website (HTTPS)** | [https://localhost:8443](https://localhost:8443) | **Admin:** `admin@tour.com` / `change_me`<br>**Khách hàng:** `khachhang@gmail.com` / `12345678` *(hoặc tự Đăng ký)* |
| **Website (HTTP)** | [http://localhost:8080](http://localhost:8080) | Tự động chuyển hướng (301 Redirect) sang HTTPS |
| **pgAdmin 4** | [http://localhost:5050](http://localhost:5050) | **Email:** `admin@tour.com`<br>**Password:** `Pg_Str0ng_2026` |
| **Grafana** | [http://localhost:3001](http://localhost:3001) | **Username:** `admin`<br>**Password:** `Grafana_Str0ng_2026` |
| **Prometheus** | [http://localhost:9090](http://localhost:9090) | Không yêu cầu đăng nhập (Kiểm tra menu *Status > Targets*) |

---

## 🛑 Dừng hệ thống

- **Dừng hệ thống (Giữ nguyên dữ liệu Database & Logs):**
  ```bash
  docker compose down
  ```
- **Dừng hệ thống và XÓA TOÀN BỘ dữ liệu (Reset từ đầu):**
  ```bash
  docker compose down -v
  ```

---

## 🔧 Một số lỗi thường gặp & Cách xử lý

1. **Cảnh báo bảo mật SSL trên trình duyệt (Your connection is not private):**
   - Do đây là chứng chỉ SSL tự ký local, tại màn hình cảnh báo bạn bấm vào nút **Advanced** ➔ Chọn **Proceed to localhost (unsafe)**. (Trên Chrome có thể click vào màn hình và gõ phím `thisisunsafe`).
2. **Không đăng nhập được tài khoản Admin trên Website:**
   - Sử dụng lệnh `docker compose restart app` để server nạp lại tài khoản Admin từ file `.env` vào Database.
3. **pgAdmin báo lỗi Email/Username is not valid:**
   - Đảm bảo sử dụng email `admin@tour.com` với mật khẩu `Pg_Str0ng_2026` đã cấu hình trong `.env`.
