# Welcome to VERTEX-R2H Ecosystem
This is the central documentation for the Mini-AWS Cloud Service project.

## Quick Links
- [IAM Gateway](https://github.com/vertex-r2h/v-r2h-iam-gateway) - Golang: Auth, RBAC Policy, Rate Limit & I18n Proxy.
- [Infrastructure](https://github.com/vertex-r2h/v-r2h-infra) - Docker Compose: Shared DB (Postgres), Redis & Nginx Config.
- [Core Engine](https://github.com/vertex-r2h/v-r2h-core-engine) - Rust: Docker/VM Control API & Resource Monitoring.
- [Business Backend](https://github.com/vertex-r2h/v-r2h-business-be) - Node.js (NestJS): Billing, Stripe Integration & User Profile.
- [Console UI](https://github.com/vertex-r2h/v-r2h-console-ui) - Next.js: Admin/User Dashboard & Web Terminal.
- [CLI Tool](https://github.com/vertex-r2h/v-r2h-cli) - Golang: Command-line interface for system interactions.

<br><br><br>
---

# PROJECT BLUEPRINT: VERTEX-R2H (MINI-AWS ECOSYSTEM)

> **Owner:** Giang Bảo Luân  
> **Status:** Planning & Architecture Defined  
> **Version:** 1.0 (Update: 2026-04-07)

---

## I. Tầm nhìn dự án (Project Vision)
Xây dựng một hệ sinh thái Cloud Service thu nhỏ (Mini-AWS) bao gồm các dịch vụ cốt lõi: Compute (EC2), Storage (S3), IAM, và Billing. Dự án tập trung vào việc luyện tập tư duy hệ thống, Microservices, và đa ngôn ngữ lập trình hiệu năng cao.

---

## II. Danh sách 6 Source Code (Repositories) - Polyrepo Strategy

| Repository | Tech Stack | Nhiệm vụ chính (Core Responsibilities) |
| :--- | :--- | :--- |
| **v-r2h-iam-gateway** | **Golang** | Cổng bảo vệ: Check Auth (JWT/Key), Phân quyền Policy (RBAC/ABAC), Rate Limit, Cache I18n. |
| **v-r2h-core-engine** | **Rust** | Động cơ: Điều khiển Docker/VM API, Quản lý File System (S3-like), Monitor tài nguyên (CPU/RAM). |
| **v-r2h-business-be** | **Node.js (NestJS)** | Nghiệp vụ: Tính tiền (Billing), Thanh toán (Stripe), Thông báo (Mail/WSS), User Profile. |
| **v-r2h-console-ui** | **Next.js** | Giao diện: Dashboard User/Admin, Web Terminal (xterm.js), i18n động. |
| **v-r2h-cli** | **Golang** | Công cụ: Dòng lệnh tương tác với hệ thống qua Access Keys. |
| **v-r2h-infra** | **Docker/Config** | Hạ tầng: Docker Compose, Nginx, Shared DB (Postgres), Redis, Logging (PLG Stack). |

---

## III. Quy chuẩn Kỹ thuật (Standard Operating Procedures)

### 1. Mô hình tương tác (Communication)
* **External:** Client -> `iam-gateway` -> Internal Services.
* **Internal Sync:** Sử dụng **gRPC** hoặc **Internal HTTP/JSON**.
* **Internal Async:** Sử dụng **RabbitMQ** hoặc **Redis Pub-Sub** cho Event-driven (ví dụ: VM_CREATED -> START_BILLING).

### 2. API Convention
Tất cả Response phải tuân thủ cấu trúc:
```json
{
  "success": true,
  "data": {}, 
  "error": { "code": "STRING_CODE", "message": "Description" },
  "meta": { "trace_id": "uuid-v4", "timestamp": "ISO-8601" }
}
```

### 3. Mã lỗi & HTTP Status
* **200/201 OK:** Thao tác thành công.
* **400 VALIDATION_ERROR:** Dữ liệu đầu vào không hợp lệ.
* **401 AUTH_INVALID:** Token hết hạn hoặc sai Access Key.
* **403 IAM_DENIED:** Không có quyền truy cập resource (Kiểm tra Policy thất bại).
* **404 NOT_FOUND:** Tài nguyên không tồn tại.
* **429 RATE_LIMIT:** Quá nhiều request từ một IP/User.
* **500 SERVER_ERROR:** Lỗi logic hệ thống hoặc lỗi kết nối database nội bộ.

### 4. Naming & Coding Style (Customized)
* **Variables & Functions:** Thống nhất sử dụng **`snake_case`** cho toàn bộ hệ thống (Ví dụ: `get_user_auth_token`, `is_active_user`).
* **Exporting (Go Specific):** Chỉ viết hoa chữ cái đầu khi cần Export (Ví dụ: `Get_user_config`), logic nội bộ vẫn dùng `snake_case`.
* **JSON Keys (API):** Toàn bộ Request/Response sử dụng **`snake_case`** (Ví dụ: `access_key_id`, `created_at`).
* **Database:** Sử dụng **`snake_case`** cho tên bảng (số nhiều) và tên cột (Ví dụ: `billing_invoices`, `user_id`).
* **File & Folder:**
    - Folder: Sử dụng **`kebab-case`** (Ví dụ: `iam-service`).
    - File: Sử dụng **`snake_case`** (Ví dụ: `auth_handler.go`).
* **Git Branching:** Tuân thủ cấu trúc `type/description` (Sử dụng kebab-case):
    - `feature/<dev_name>/<#id-github>_`: Tính năng mới (Ví dụ: `feature/giang_bao_luan/#1_iam_register`).
    - `fix/<dev_name>/<#id-github>_`: Sửa lỗi (Ví dụ: `fix/giang_bao_luan/#1_db_connection`).
    - `refactor/<dev_name>/<#id-github>_`: Tối ưu cấu trúc code (Ví dụ: `refactor/giang_bao_luan/#1_logging_service`).
    - `docs/<dev_name>/<#id-github>_`: Cập nhật tài liệu (Ví dụ: `docs/giang_bao_luan/#1_update_blueprint`).
    - `chore/`: Cấu hình, thư viện, việc vặt (Ví dụ: `chore/giang_bao_luan/#2_setup_husky`).

---

## IV. Chiến lược Logging & Quan sát (Observability)
* **Format:** Structured Logging (Luôn xuất ra định dạng JSON).
* **Phân loại Log:**
    - **System Log:** Ghi lại các lỗi runtime, panic, lỗi kết nối DB.
    - **Access Log:** Ghi lại IP, Method, Endpoint, Status code.
    - **Audit Log:** Ghi lại hành vi nhạy cảm (Ví dụ: "User Luân đã xóa S3-Bucket-01").
* **Stack lưu trữ:** **PLG** (Promtail + Loki + Grafana).
* **Tracing:** Mỗi request mang theo `X-Trace-Id` để theo dõi luồng đi qua các service.

---

## V. Đa ngôn ngữ (Dynamic I18n)
* **Storage:** Lưu Key-Value bản dịch trong Database (Postgres).
* **Performance (Warm-up Strategy):** - Khi khởi động (Start-up), Backend thực hiện "Warm-up": Quét toàn bộ bản dịch từ Postgres và đồng bộ vào **Redis**.
    - Các node chạy sau sẽ ưu tiên đọc từ Redis để giảm tải cho Database chính.
* **Usage:** Gateway lấy bản dịch từ Redis trả về cho Frontend cực nhanh qua Cache-hit.
* **Frontend:** Fetch bản dịch một lần khi load App và lưu vào Local Storage/State Management.

---

## VI. Lộ trình thực hiện & Tiến độ (Roadmap & Progress)

### Milestone 1: Hạ tầng cơ sở và Kết nối hệ thống (DONE)
- [x] Khởi tạo cấu trúc dự án và quản lý mã nguồn (Polyrepo).
- [x] Thiết lập Gateway với Go, Gin và GORM.
- [x] Triển khai Container hóa hạ tầng (Postgres 15, Redis 7).
- [x] Xây dựng hệ thống quản lý cấu hình đa môi trường (.env.development).
- [x] Thiết lập Global Middleware cho Request Tracing (X-Trace-ID).
- [x] Thiết kế Database Schema ban đầu cho IAM (Bảng users).
- [x] Kết nối thành công Gateway tới Database thông qua Docker.

### Milestone 2: Định danh và Xác thực (IN PROGRESS)
- [ ] Định nghĩa GORM User Model và triển khai Auto Migration.
- [ ] Xây dựng tính năng Register (Mã hóa mật khẩu với Bcrypt).
- [ ] Triển khai JWT Authentication (Login & Token validation).
- [ ] Thiết kế Schema cho Roles và Policies (RBAC/ABAC).

### Milestone 3: Giao diện và Đa ngôn ngữ
- [ ] Dựng Console UI với Next.js.
- [ ] Triển khai I18n Engine (Lưu Postgres, Cache Redis).

### Milestone 4: Core Engine (Rust)
- [ ] Xây dựng Service điều khiển Docker API bằng Rust.

### Milestone 5: Billing & Payments (Node.js)
- [ ] Triển khai tính toán tài nguyên và tích hợp Stripe.

---

## VII. Nhật ký thay đổi (Changelog)
- 2026-04-07: Hoàn thành kết nối Database, xử lý Log màu ANSI, thiết lập chuẩn Trace-ID.
- 2026-04-06: Khởi tạo Project Blueprint và cấu trúc thư mục.
