# 🌏 Hệ thống Đặt Tour Du lịch
**Mã số sinh viên:** DTC245200232 | **Họ tên:** Thèn Thành Đạt  
**Môn học:** Triển khai và Quản trị Hệ thống

---

## 📋 Mô tả dự án

Hệ thống đặt tour du lịch được containerize hoàn toàn bằng **Docker Compose**, bao gồm:
- Ứng dụng web **Node.js/Express** với phân quyền 2 role (Admin / Customer)
- Cơ sở dữ liệu **PostgreSQL 16** được bảo vệ trong mạng nội bộ
- **Nginx** làm reverse proxy với HTTPS và đầy đủ Security Headers
- Stack giám sát **Prometheus + Grafana + cAdvisor** theo thời gian thực
- Hệ thống log tập trung **Loki + Promtail** với truy vấn LogQL
- Quản trị database qua **pgAdmin 4**
- Áp dụng đầy đủ các biện pháp **hardening** bảo mật container

---

## 🛠 Công nghệ sử dụng (Tech Stack)

| Thành phần | Công nghệ |
|---|---|
| Ứng dụng | Node.js 20 + Express |
| Cơ sở dữ liệu | PostgreSQL 16-alpine |
| Quản trị CSDL | pgAdmin 4 |
| Web server / Proxy | Nginx (nginx-unprivileged) + HTTPS |
| Container hóa | Docker + Docker Compose |
| Giám sát metrics | Prometheus + Grafana + cAdvisor |
| Exporter | nginx-prometheus-exporter, postgres-exporter |
| Log tập trung | Loki 3.0.0 + Promtail 3.0.0 |

---

## 🚀 Hướng dẫn triển khai

### Yêu cầu hệ thống
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows/macOS) hoặc Docker Engine (Linux)
- Git
- OpenSSL (thường có sẵn trên Linux/macOS; Windows cần cài thêm)

### Bước 1 — Clone repository

```bash
git clone https://github.com/dtc245200232/tour-booking.git
cd tour-booking
```

### Bước 2 — Cấu hình biến môi trường

```bash
cp .env.example .env
```

Mở file `.env` và **thay đổi các giá trị mật khẩu** trước khi chạy:

```env
POSTGRES_USER=admin_db
POSTGRES_PASSWORD=<mật_khẩu_mạnh>      # Đổi lại
POSTGRES_DB=tourdb
APP_DB_PASSWORD=<mật_khẩu_mạnh>        # Đổi lại
PGADMIN_EMAIL=admin@tour.com
PGADMIN_PASSWORD=<mật_khẩu_mạnh>       # Đổi lại
GRAFANA_PASSWORD=<mật_khẩu_mạnh>       # Đổi lại
SESSION_SECRET=<chuỗi_ngẫu_nhiên_dài>  # Đổi lại
ADMIN_EMAIL=admin@tour.com
ADMIN_PASSWORD=<mật_khẩu_mạnh>         # Đổi lại
```

> ⚠️ **Lưu ý bảo mật:** File `.env` đã được thêm vào `.gitignore` và sẽ **không bao giờ** được commit lên GitHub.

### Bước 3 — Tạo chứng chỉ SSL tự ký (HTTPS)

```bash
# Tạo thư mục chứa chứng chỉ
mkdir -p nginx/certs

# Tạo chứng chỉ SSL (có hiệu lực 365 ngày)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/certs/server.key \
  -out nginx/certs/server.crt \
  -subj "//CN=localhost"
```

> 💡 **Trên Windows** (PowerShell): thay `//CN=localhost` bằng `/CN=localhost`

### Bước 4 — Khởi động toàn bộ hệ thống

```bash
docker compose up -d --build
```

Hệ thống sẽ tự động khởi động **11 container** theo đúng thứ tự phụ thuộc.

### Bước 5 — Kiểm tra trạng thái

```bash
docker compose ps
```

Tất cả các container phải ở trạng thái `Up` (db có thêm `(healthy)`).

### Dừng hệ thống

```bash
# Dừng, giữ nguyên dữ liệu
docker compose down

# Dừng và XÓA TOÀN BỘ dữ liệu (reset từ đầu)
docker compose down -v
```

---

## 🌐 Địa chỉ truy cập các dịch vụ

| Dịch vụ | URL | Ghi chú |
|---|---|---|
| **Website** | https://localhost:8443 | Trang chủ ứng dụng (HTTPS) |
| **pgAdmin** | http://localhost:5050 | Quản trị PostgreSQL |
| **Grafana** | http://localhost:3001 | Dashboard giám sát hệ thống |
| **Prometheus** | http://localhost:9090 | Metrics & Targets |

> 💡 Khi truy cập `https://localhost:8443` lần đầu, trình duyệt sẽ cảnh báo chứng chỉ tự ký. Chọn **Advanced → Proceed to localhost** để tiếp tục.

---

## 🔑 Tài khoản mặc định

| Dịch vụ | Tài khoản | Mật khẩu |
|---|---|---|
| Website (Admin) | `admin@tour.com` | Xem trong file `.env` — `ADMIN_PASSWORD` |
| Website (Khách hàng) | Đăng ký mới tại `/register` | — |
| pgAdmin | `admin@tour.com` | Xem trong file `.env` — `PGADMIN_PASSWORD` |
| Grafana | `admin` | Xem trong file `.env` — `GRAFANA_PASSWORD` |
| Prometheus | Không cần đăng nhập | — |

---

## 🏗 Cấu trúc dự án

```
tour-booking/
├── app/                        # Ứng dụng Node.js
│   ├── Dockerfile              # Build image non-root
│   ├── server.js               # Entry point Express
│   ├── package.json
│   ├── public/                 # Static files (CSS, JS)
│   └── views/                  # Template EJS
│       ├── dashboard.ejs
│       ├── login.ejs, register.ejs
│       ├── tours.ejs, tour.ejs
│       ├── bookings.ejs, my-bookings.ejs
│       ├── customers.ejs
│       ├── settings.ejs
│       └── partials/           # Header, footer
├── db/
│   └── init/
│       ├── 01-schema.sql       # Tạo bảng & dữ liệu mẫu
│       └── 02-app-user.sh      # Tạo user DB phân quyền
├── nginx/
│   ├── conf.d/default.conf     # Reverse proxy + Security Headers
│   └── certs/                  # SSL certs (không commit)
├── prometheus/
│   └── prometheus.yml          # Cấu hình scrape jobs
├── grafana/
│   └── provisioning/
│       └── datasources/ds.yml  # Auto-provision Prometheus & Loki
├── loki/
│   └── loki-config.yml         # Cấu hình Loki
├── promtail/
│   └── promtail-config.yml     # Cấu hình thu thập log Docker
├── docker-compose.yml          # Định nghĩa toàn bộ 11 services
├── .env.example                # Mẫu biến môi trường
├── .gitignore                  # Loại trừ .env, nginx/certs/
└── README.md
```

---

## 📊 Kiến trúc hệ thống

```
[Người dùng / Trình duyệt]
        │ HTTPS :8443 / HTTP :8080 (redirect)
        ▼
┌─────────────────────┐
│  Nginx (Reverse     │  ← Security Headers, TLS 1.2/1.3
│  Proxy + HTTPS)     │  ← Chặn /metrics từ ngoài
└────────┬────────────┘
         │ proxy_pass :3000
         ▼
┌─────────────────────┐     ┌─────────────────────┐
│  Node.js App        │────▶│  PostgreSQL          │
│  (tour_app)         │     │  (tour_db) [db_net]  │
│  non-root, readonly │     │  internal: true      │
└─────────────────────┘     └──────────┬──────────┘
                                        │
                              ┌─────────▼─────────┐
                              │  pgAdmin           │
                              │  (tour_pgadmin)    │
                              └───────────────────┘

═══════════════ TẦNG GIÁM SÁT [monitor_net] ═══════════════
┌─────────────┐   scrape    ┌──────────────────────────────┐
│ Prometheus  │ ◄─────────── │ cAdvisor (container metrics) │
│ (mỗi 10s)  │ ◄─────────── │ nginx-exporter               │
└──────┬──────┘             │ postgres-exporter             │
       │                    │ app /metrics                  │
       ▼                    └──────────────────────────────┘
┌─────────────┐
│   Grafana   │ ◄── Loki (log storage)
│  Dashboard  │          ▲
└─────────────┘          │ push
                   ┌─────┴──────┐
                   │  Promtail  │ ◄── Docker socket (tất cả log)
                   └────────────┘
```

---

## 🔒 Các biện pháp bảo mật (Hardening)

| Biện pháp | Chi tiết |
|---|---|
| **Non-root container** | App chạy bằng `user: node`; Nginx dùng `nginx-unprivileged` |
| **Read-only filesystem** | `read_only: true` + `tmpfs: [/tmp]` cho App |
| **Drop capabilities** | `cap_drop: [ALL]` cho App và Nginx |
| **No privilege escalation** | `security_opt: ["no-new-privileges:true"]` |
| **Network isolation** | `db_net: internal: true` — DB không có đường ra Internet |
| **Least privilege DB** | User `tour_app` chỉ có quyền DML, không có DDL |
| **Security Headers** | X-Frame-Options, CSP, HSTS, X-XSS-Protection, ... |
| **Secret management** | Mật khẩu qua `.env`, không hardcode, không commit |
| **Endpoint protection** | `/metrics` bị Nginx chặn từ ngoài (`return 404`) |

---

## 📈 Giám sát hệ thống

### Prometheus Targets (http://localhost:9090/targets)
| Job | Nguồn | Metrics |
|---|---|---|
| `app` | `app:3000/metrics` | HTTP requests, response time |
| `cadvisor` | `cadvisor:8080` | CPU, RAM, Network container |
| `nginx` | `nginx-exporter:9113` | Connections, requests |
| `postgres` | `postgres-exporter:9187` | Connections, transactions, queries |

### Câu truy vấn PromQL mẫu
```promql
# Số kết nối Nginx đang hoạt động
nginx_connections_active

# Số kết nối vào database
pg_stat_database_numbackends{datname="tourdb"}

# Tổng HTTP request theo route
http_requests_total

# % CPU Docker đang dùng
rate(container_cpu_usage_seconds_total{id="/docker"}[1m]) * 100
```

---

## 📋 Quản lý log tập trung (Loki + Promtail)

Truy cập **Grafana → Explore → chọn datasource Loki** để truy vấn log.

### Câu truy vấn LogQL mẫu
```logql
# Toàn bộ log ứng dụng
{container="tour_app"}

# Theo dõi sự kiện đăng nhập
{container="tour_app"} |= "LOGIN"

# Phát hiện lỗi HTTP 4xx/5xx từ Nginx
{container="tour_nginx"} |~ " (4|5)[0-9]{2} "

# Tốc độ sinh log theo thời gian
rate({container="tour_app"}[1m])
```

---

## ⚠️ Xử lý sự cố thường gặp

| Lỗi | Nguyên nhân | Cách xử lý |
|---|---|---|
| Không đăng nhập được tài khoản Admin | App khởi động trước DB xong | `docker compose restart app` |
| Trình duyệt chặn HTTPS (ERR_CERT_*) | Chứng chỉ tự ký | Tại màn hình lỗi gõ `thisisunsafe` (Chrome) |
| pgAdmin báo "Email not valid" | Đuôi email `.local` bị chặn | Dùng email có đuôi `.com` như `admin@tour.com` |
| Container không start | Chưa có file `.env` | Chạy `cp .env.example .env` |

---

## 📜 Lịch sử phát triển (Commit History)

| Commit | Nội dung |
|---|---|
| `373e410` | Initial commit — cấu trúc dự án ban đầu |
| `8142b51` | feat: web booking tour + PostgreSQL + pgAdmin + Nginx (HTTPS, security headers) |
| `9e381bb` | feat: monitoring với Prometheus, Grafana, cAdvisor, nginx và postgres exporters |
| `7234e4f` | feat: centralized logging với Loki + Promtail (LogQL) và system hardening |
| `ead7054` | Update system configuration, Docker setup, and RBAC |
| `6fedd51` | docs: update README với tất cả tài khoản dịch vụ và fix pgAdmin email |
