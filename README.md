# Đặc tả Diagram Hệ thống Đặt sân Pickleball

> **Mục đích**: Tài liệu này mô tả chi tiết toàn bộ sơ đồ (diagram) của hệ thống, dùng làm bản vẽ kỹ thuật để vẽ lại bằng công cụ vẽ (draw.io, Visio, Figma, PlantUML...).
>
> **Công nghệ**: Frontend React 18 + TypeScript, Backend Node.js Express, Database PostgreSQL, Deploy Docker.

---

## Danh sách Diagram

| # | Tên sơ đồ | Loại | Trang |
|---|-----------|------|-------|
| 1 | Kiến trúc tổng thể hệ thống | System Architecture | §1 |
| 2 | Sơ đồ quan hệ dữ liệu (ERD) | Entity-Relationship | §2 |
| 3 | Sơ đồ Use Case tổng quát | Use Case | §3 |
| 4 | Sơ đồ luồng đặt sân (User Booking) | Sequence | §4 |
| 5 | Sơ đồ luồng check-in / check-out | Sequence | §5 |
| 6 | Sơ đồ máy trạng thái Booking | State Machine | §6 |
| 7 | Sơ đồ thành phần Frontend | Component Tree | §7 |
| 8 | Sơ đồ triển khai (Deployment) | Deployment | §8 |
| 9 | Sơ đồ xác thực & phân quyền | Sequence | §9 |
| 10 | Sơ đồ luồng thanh toán | Sequence | §10 |
| 11 | Sơ đồ VIP Auto-Booking | Activity | §11 |
| 12 | Sơ đồ Scheduler & Cron Jobs | Activity | §12 |

---

## §1. Kiến trúc tổng thể hệ thống (System Architecture Diagram)

### Mô tả

Sơ đồ mô tả kiến trúc 3 tầng (3-tier architecture) của hệ thống Pickleball Booking.

### Các thành phần chính

| Tầng | Thành phần | Chi tiết |
|------|-----------|----------|
| **Client** | Web Browser | Người dùng truy cập qua trình duyệt, giao thức HTTPS |
| **Frontend** | React SPA (Vite) | Single Page Application, build ra static files |
| **Backend** | Express.js Server | REST API server, port 3000 |
| **Database** | PostgreSQL | Lưu trữ toàn bộ dữ liệu quan hệ |
| **File Storage** | Local /public/uploads | Lưu ảnh sân, avatar (có thể thay bằng S3) |
| **Scheduler** | node-cron jobs | Chạy tác vụ tự động định kỳ trong cùng process |

### Bố cục sơ đồ

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT (Browser)                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │
│  │   Người dùng  │    │   Chủ sân    │    │   Admin      │       │
│  │  (Customer)  │    │  (Customer+) │    │   (Admin)    │       │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘       │
│         │                   │                   │                │
│         └───────────────────┼───────────────────┘                │
│                             │ HTTPS                              │
└─────────────────────────────┼────────────────────────────────────┘
                              │
┌─────────────────────────────┼────────────────────────────────────┐
│                    FRONTEND (React + Vite)                        │
│                             │                                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Nginx / Express Static (phục vụ file build)              │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  React SPA                                                │   │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌───────────┐ │   │
│  │  │ Auth     │ │ User     │ │ Admin      │ │ Error     │ │   │
│  │  │ Pages    │ │ Pages    │ │ Pages      │ │ Pages     │ │   │
│  │  │ /login   │ │ /        │ │ /admin/*   │ │ /forbidden│ │   │
│  │  │ /forgot  │ │ /courts  │ │ dashboard  │ │            │ │   │
│  │  │ /reset   │ │ /profile │ │ courts     │ │            │ │   │
│  │  └──────────┘ │ /bookings│ │ timeslots  │ └───────────┘ │   │
│  │               │ /vouchers│ │ bookings   │               │   │
│  │               └──────────┘ │ services   │               │   │
│  │                            │ users      │               │   │
│  │                            │ reports    │               │   │
│  │                            │ discounts  │               │   │
│  │                            │ scanner    │               │   │
│  │                            │ schedule   │               │   │
│  │                            └────────────┘               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  State: Zustand (authStore, themeStore)                           │
│  Server State: TanStack React Query                               │
│  Routing: React Router v7 (lazy loading, code splitting)          │
│  HTTP Client: Axios (with interceptors)                           │
│  UI: Tailwind CSS + Radix UI + Lucide Icons + Recharts            │
└─────────────────────────────┬────────────────────────────────────┘
                              │ REST API (JSON) /api/*
                              │ Authentication: JWT Bearer Token
┌─────────────────────────────┼────────────────────────────────────┐
│                    BACKEND (Express.js)                            │
│                             │                                      │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  Middleware Pipeline                                       │    │
│  │  cors() → express.json() → serveStatic() → Routes → 404  │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Route Handlers                                            │   │
│  │  ┌─────────┐ ┌─────────┐ ┌──────────┐ ┌──────────┐      │   │
│  │  │ /api/auth│ │/api/users│ │/api/courts│ │/api/bookings│   │   │
│  │  │ Đăng ký │ │ QL User │ │ QL Sân   │ │ Đặt sân   │      │   │
│  │  │ Đăng nhập│ │(admin)  │ │+Timeslot │ │Check-in   │      │   │
│  │  │ Quên MK │ │          │ │          │ │Hủy        │      │   │
│  │  └─────────┘ └─────────┘ └──────────┘ └──────────┘      │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │   │
│  │  │/api/reviews│ │/api/admin│ │/api/upload│ │ Router [*] │  │   │
│  │  │ Đánh giá │ │Dashboard │ │Upload file│ │ SPA Fallback│  │   │
│  │  │          │ │Reports   │ │           │ │            │  │   │
│  │  │          │ │Services  │ │           │ │            │  │   │
│  │  │          │ │Discounts │ │           │ │            │  │   │
│  │  │          │ │Noti      │ │           │ │            │  │   │
│  │  │          │ │Schedule  │ │           │ │            │  │   │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Middleware: authenticate (JWT verify)                    │    │
│  │             requireAdmin (role check)                     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │  Utils           │  │  Scheduler       │                     │
│  │  bookingCancel   │  │  node-cron jobs  │                     │
│  │  formatDate      │  │  - handleStatus  │                     │
│  │                  │  │  - autoCancel    │                     │
│  │                  │  │  - vipAutoBook   │                     │
│  └──────────────────┘  └──────────────────┘                     │
└─────────────────────────────┬────────────────────────────────────┘
                              │ pg (node-postgres)
                              │ SQL / Connection Pool
┌─────────────────────────────┼────────────────────────────────────┐
│                    DATABASE (PostgreSQL)                           │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Tables:                                                  │    │
│  │  users | courts | timeslots | services | bookings        │    │
│  │  booking_services | payments | reviews | notifications   │    │
│  │  court_images | discounts | auto_booking_series          │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  File Storage: /public/uploads/                           │    │
│  │  ├── avatars/    (ảnh đại diện người dùng)               │    │
│  │  ├── courts/     (ảnh sân pickleball)                    │    │
│  │  └── general/    (ảnh khác)                               │    │
│  └─────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────┐
│                   EXTERNAL SERVICES (tùy chọn)                    │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │  SePay (Payment)  │  │  Email Service   │                     │
│  │  Webhook xử lý    │  │  Gửi OTP /       │                     │
│  │  thanh toán       │  │  thông báo       │                     │
│  └──────────────────┘  └──────────────────┘                     │
└──────────────────────────────────────────────────────────────────┘
```

---

## §2. Sơ đồ quan hệ dữ liệu - ERD (Entity-Relationship Diagram)

### Danh sách bảng và quan hệ

#### Bảng: `users` (Người dùng)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| hoTen | VARCHAR(100) | NOT NULL | Họ tên |
| email | VARCHAR(150) | UNIQUE, NOT NULL | Email đăng nhập |
| matKhau | VARCHAR(255) | NOT NULL | Mật khẩu hash (bcrypt) |
| soDienThoai | VARCHAR(15) | UNIQUE | Số điện thoại |
| vaiTro | VARCHAR(50) | DEFAULT 'Customer' | Vai trò: Customer / Admin |
| isVIP | BOOLEAN | DEFAULT FALSE | Khách VIP |
| gioiTinh | VARCHAR(10) | | Giới tính |
| diaChi | VARCHAR(255) | | Địa chỉ |
| avatar_url | VARCHAR(500) | | URL ảnh đại diện |
| trangThai | VARCHAR(50) | DEFAULT 'Active' | Trạng thái: Active / Locked |
| created_at | TIMESTAMP | DEFAULT NOW() | Ngày tạo |
| updated_at | TIMESTAMP | DEFAULT NOW() | Ngày cập nhật |

#### Bảng: `courts` (Sân Pickleball)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| tenSan | VARCHAR(100) | NOT NULL | Tên sân |
| moTa | TEXT | | Mô tả |
| hinhAnh | VARCHAR(500) | | Ảnh đại diện |
| trangThai | VARCHAR(50) | DEFAULT 'Sẵn sàng' | Sẵn sàng / Bảo trì / Ngừng |

#### Bảng: `timeslots` (Khung giờ)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| sanId | INTEGER | FK → courts(id) ON DELETE CASCADE | Sân |
| gioBatDau | TIME | NOT NULL | Giờ bắt đầu |
| gioKetThuc | TIME | NOT NULL | Giờ kết thúc |
| mucGia | DECIMAL(15,2) | NOT NULL | Mức giá khung giờ |
| trangThai | VARCHAR(50) | DEFAULT 'Active' | Active / Inactive |

#### Bảng: `services` (Dịch vụ)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| tenDichVu | VARCHAR(100) | NOT NULL | Tên dịch vụ (vợt, bóng, nước uống...) |
| donGia | DECIMAL(15,2) | NOT NULL | Đơn giá |
| loaiDichVu | VARCHAR(50) | | Loại: equipment / drink |
| soLuongTon | INTEGER | DEFAULT 0 | Số lượng tồn kho |
| trangThai | VARCHAR(50) | DEFAULT 'Còn hàng' | Trạng thái |

#### Bảng: `bookings` (Đơn đặt sân)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| nguoiDungId | INTEGER | FK → users(id) | Người đặt |
| sanId | INTEGER | FK → courts(id) | Sân đã chọn |
| khungGioId | INTEGER | FK → timeslots(id) | Khung giờ đã chọn |
| ngayChoi | DATE | NOT NULL | Ngày chơi |
| tongTien | DECIMAL(15,2) | NOT NULL | Tổng tiền |
| tienDaCoc | DECIMAL(15,2) | DEFAULT 0 | Tiền đã cọc |
| giaGoc | DECIMAL(15,2) | | Giá gốc trước giảm |
| tienGiam | DECIMAL(15,2) | DEFAULT 0 | Tiền giảm giá |
| maGiamGia | VARCHAR(50) | | Mã giảm giá đã dùng |
| trangThai | VARCHAR(50) | DEFAULT 'Đã cọc' | Trạng thái đơn |
| isAutoBooking | BOOLEAN | DEFAULT FALSE | Là auto-booking VIP? |
| autoBookingSeriesId | INTEGER | FK → auto_booking_series(id) | Series auto-booking |
| ghiChu | TEXT | | Ghi chú |

#### Bảng: `auto_booking_series` (Series đặt tự động VIP)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| nguoiDungId | INTEGER | FK → users(id) | Người đặt |
| sanId | INTEGER | FK → courts(id) | Sân |
| khungGioIds | JSONB | NOT NULL | Mảng ID khung giờ |
| startDate | DATE | NOT NULL | Ngày bắt đầu |
| endDate | DATE | NOT NULL | Ngày kết thúc |
| repeatServices | BOOLEAN | DEFAULT FALSE | Lặp dịch vụ? |
| servicePolicy | VARCHAR(50) | | first_only / every_slot |
| totalAmount | DECIMAL(15,2) | | Tổng tiền cả series |
| trangThai | VARCHAR(50) | DEFAULT 'Active' | Active / Cancelled |

#### Bảng: `booking_services` (Dịch vụ trong đơn)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| donDatId | INTEGER | FK → bookings(id) ON DELETE CASCADE | Đơn đặt |
| dichVuId | INTEGER | FK → services(id) | Dịch vụ |
| soLuong | INTEGER | DEFAULT 1 | Số lượng |
| tongTien | DECIMAL(15,2) | NOT NULL | Thành tiền |

#### Bảng: `payments` (Thanh toán)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| donDatId | INTEGER | FK → bookings(id) ON DELETE CASCADE | Đơn đặt |
| soTien | DECIMAL(15,2) | NOT NULL | Số tiền thanh toán |
| loaiThanhToan | VARCHAR(50) | NOT NULL | Hình thức: Tiền mặt / Chuyển khoản |
| ngayGiaoDich | TIMESTAMP | DEFAULT NOW() | Ngày giao dịch |
| trangThai | VARCHAR(50) | DEFAULT 'Thành công' | Trạng thái |

#### Bảng: `reviews` (Đánh giá)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| donDatId | INTEGER | FK → bookings(id) (nullable) | Đơn đặt (có thể null - đánh giá cấp sân) |
| nguoiDungId | INTEGER | FK → users(id) | Người đánh giá |
| sanId | INTEGER | FK → courts(id) | Sân được đánh giá |
| diemSao | INTEGER | CHECK 1-5 | Điểm sao (1-5) |
| binhLuan | TEXT | | Bình luận |
| ngayTao | TIMESTAMP | DEFAULT NOW() | Ngày đánh giá |

#### Bảng: `notifications` (Thông báo)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| nguoiDungId | INTEGER | FK → users(id) | Người nhận |
| tieuDe | VARCHAR(255) | NOT NULL | Tiêu đề |
| noiDung | TEXT | | Nội dung |
| loaiThongBao | VARCHAR(50) | DEFAULT 'system' | Loại: system / booking / payment |
| daDoc | BOOLEAN | DEFAULT FALSE | Đã đọc? |
| maDonDat | INTEGER | FK → bookings(id) | Đơn liên quan |
| thoiGianTao | TIMESTAMP | DEFAULT NOW() | Thời gian tạo |

#### Bảng: `court_images` (Ảnh sân)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| sanId | INTEGER | FK → courts(id) ON DELETE CASCADE | Sân |
| duongDanAnh | VARCHAR(500) | NOT NULL | Đường dẫn ảnh |
| isMain | BOOLEAN | DEFAULT FALSE | Là ảnh chính? |
| created_at | TIMESTAMP | DEFAULT NOW() | Ngày tạo |

#### Bảng: `discounts` (Mã giảm giá)

| Cột | Kiểu | Ràng buộc | Mô tả |
|-----|------|-----------|-------|
| id | SERIAL | PK | Khóa chính |
| code | VARCHAR(50) | UNIQUE, NOT NULL | Mã code |
| noiDung | VARCHAR(255) | | Tên hiển thị |
| moTa | TEXT | | Mô tả chi tiết |
| loaiGiamGia | VARCHAR(50) | DEFAULT 'percentage' | percentage / fixed |
| mucGiamGia | DECIMAL(15,2) | NOT NULL | Mức giảm (% hoặc VND) |
| ngayBatDau | TIMESTAMP | | Ngày bắt đầu hiệu lực |
| ngayKetThuc | TIMESTAMP | | Ngày hết hạn |
| soLuongBanDau | INTEGER | DEFAULT 0 | Tổng lượt |
| soLuongDaDung | INTEGER | DEFAULT 0 | Đã dùng |
| usage_limit_per_user | INTEGER | DEFAULT 1 | Giới hạn/user |
| giamToiDa | DECIMAL(15,2) | | Giảm tối đa |
| conditions | JSONB | DEFAULT '{}' | Điều kiện: min_order, court_ids, target_audience |
| nguoiDungId | INTEGER | FK → users(id) | Chủ sở hữu voucher |
| is_hidden | BOOLEAN | DEFAULT FALSE | Ẩn khỏi danh sách công khai |
| trangThai | VARCHAR(50) | DEFAULT 'Active' | Active / Inactive |

### Quan hệ giữa các bảng

```
users (1) ────< (N) bookings
users (1) ────< (N) reviews
users (1) ────< (N) notifications
users (1) ────< (N) discounts
users (1) ────< (N) auto_booking_series

courts (1) ────< (N) timeslots
courts (1) ────< (N) bookings
courts (1) ────< (N) court_images
courts (1) ────< (N) reviews
courts (1) ────< (N) auto_booking_series

timeslots (1) ────< (N) bookings

bookings (1) ────< (N) booking_services
bookings (1) ────< (N) payments
bookings (1) ────< (N) reviews
bookings (1) ────< (N) notifications

services (1) ────< (N) booking_services

auto_booking_series (1) ────< (N) bookings
```

---

## §3. Sơ đồ Use Case (Use Case Diagram)

### Actors

| Actor | Mô tả |
|-------|-------|
| **Khách (Chưa đăng nhập)** | Người dùng chưa có tài khoản, có thể xem sân, đăng ký |
| **Customer (Đã đăng nhập)** | Người dùng đã đăng nhập, có thể đặt sân, quản lý đơn |
| **VIP Customer** | Customer có cờ isVIP = true, được auto-booking 30 ngày |
| **Admin** | Quản trị viên hệ thống, có toàn quyền quản lý |

### Danh sách Use Case

#### Actor: Khách (Guest)
- UC-01: Đăng ký tài khoản
- UC-02: Đăng nhập (có OTP)
- UC-03: Quên mật khẩu / Reset password
- UC-04: Xem danh sách sân
- UC-05: Xem chi tiết sân + đánh giá

#### Actor: Customer
- UC-06: Đặt sân (chọn sân, ngày, khung giờ, dịch vụ, mã giảm giá)
- UC-07: Xem lịch sử đặt sân (my bookings)
- UC-08: Xem chi tiết đơn đặt
- UC-09: Hủy đơn (trước 3h giờ chơi)
- UC-10: Xem mã QR (để check-in)
- UC-11: Viết đánh giá sân
- UC-12: Quản lý profile (cập nhật thông tin, avatar)
- UC-13: Xem voucher / mã giảm giá của tôi
- UC-14: Nhận thông báo

#### Actor: VIP Customer (kế thừa Customer)
- UC-15: Kích hoạt auto-booking 30 ngày
- UC-16: Xem series auto-booking

#### Actor: Admin
- UC-17: Xem dashboard thống kê
- UC-18: Quản lý sân (CRUD)
- UC-19: Quản lý khung giờ (CRUD)
- UC-20: Quản lý đơn đặt sân (xem tất cả, check-in, check-out, no-show)
- UC-21: Quản lý dịch vụ (CRUD)
- UC-22: Quản lý người dùng (khóa/mở khóa, nâng VIP)
- UC-23: Quản lý mã giảm giá (CRUD, tạo mã riêng cho user)
- UC-24: Quét mã QR check-in
- UC-25: Xem báo cáo doanh thu + xuất Excel
- UC-26: Xem bảng lịch sân (Schedule Board)
- UC-27: Gửi thông báo
- UC-28: Upload ảnh sân

---

## §4. Sơ đồ luồng đặt sân - User Booking (Sequence Diagram)

### Mô tả

Luồng Customer đặt sân: chọn sân → chọn ngày → chọn khung giờ → thêm dịch vụ → áp mã giảm giá → đặt cọc → tạo đơn thành công.

### Các bên tham gia (Lifelines)

- **User** (Browser)
- **Frontend** (React SPA)
- **API** (Express Server)
- **Database** (PostgreSQL)

### Luồng tuần tự

```
User              Frontend (React)          API (Express)          Database (PG)
 │                     │                        │                      │
 │  Chọn sân           │                        │                      │
 │────────────────────>│                        │                      │
 │                     │ GET /api/courts         │                      │
 │                     │───────────────────────>│                      │
 │                     │                        │ SELECT courts        │
 │                     │                        │─────────────────────>│
 │                     │                        │<─────────────────────│
 │                     │<───────────────────────│                      │
 │  Hiển thị DS sân    │                        │                      │
 │<────────────────────│                        │                      │
 │                     │                        │                      │
 │  Chọn sân, xem      │                        │                      │
 │  chi tiết           │                        │                      │
 │────────────────────>│                        │                      │
 │                     │ GET /api/courts/:id     │                      │
 │                     │───────────────────────>│                      │
 │                     │                        │ SELECT court + imgs  │
 │                     │                        │ SELECT slots by date │
 │                     │                        │─────────────────────>│
 │                     │                        │<─────────────────────│
 │                     │<───────────────────────│                      │
 │  Hiển thị chi tiết  │                        │                      │
 │  sân + khung giờ    │                        │                      │
 │<────────────────────│                        │                      │
 │                     │                        │                      │
 │  Chọn khung giờ,    │                        │                      │
 │  thêm dịch vụ,      │                        │                      │
 │  nhập mã giảm giá   │                        │                      │
 │                     │                        │                      │
 │  Bấm "Đặt sân"      │                        │                      │
 │────────────────────>│                        │                      │
 │                     │ POST /api/discounts/    │                      │
 │                     │      validate           │                      │
 │                     │───────────────────────>│                      │
 │                     │                        │ Validate mã giảm giá │
 │                     │                        │ (trạng thái, hạn,    │
 │                     │                        │  số lượng, điều kiện)│
 │                     │                        │─────────────────────>│
 │                     │                        │<─────────────────────│
 │                     │<───────────────────────│                      │
 │  Hiển thị giá       │  { valid, discountAmt,│                      │
 │  sau giảm           │    finalAmount }       │                      │
 │<────────────────────│                        │                      │
 │                     │                        │                      │
 │  Xác nhận đặt       │                        │                      │
 │────────────────────>│                        │                      │
 │                     │ POST /api/bookings      │                      │
 │                     │───────────────────────>│                      │
 │                     │                        │ BEGIN TRANSACTION    │
 │                     │                        │─────────────────────>│
 │                     │                        │ 1. Kiểm tra xung đột │
 │                     │                        │    (slot đã đặt chưa)│
 │                     │                        │ 2. INSERT booking    │
 │                     │                        │ 3. INSERT booking_   │
 │                     │                        │    services          │
 │                     │                        │ 4. INSERT payment    │
 │                     │                        │    (tiền cọc)        │
 │                     │                        │ 5. UPDATE discount   │
 │                     │                        │    usage count       │
 │                     │                        │ 6. INSERT noti       │
 │                     │                        │ COMMIT               │
 │                     │                        │─────────────────────>│
 │                     │<───────────────────────│                      │
 │  Hiển thị thành     │  { booking, qrCode }  │                      │
 │  công + mã QR       │                        │                      │
 │<────────────────────│                        │                      │
```

---

## §5. Sơ đồ luồng Check-in / Check-out (Sequence Diagram)

### Mô tả

Admin check-in khách (quét QR hoặc thủ công) → đổi trạng thái đơn → check-out khi hết giờ.

### Luồng tuần tự

```
Admin             Frontend (Admin)         API                    DB
 │                     │                     │                      │
 │  Quét mã QR         │                     │                      │
 │  hoặc tìm đơn       │                     │                      │
 │────────────────────>│                     │                      │
 │                     │ GET /api/bookings/:id│                     │
 │                     │────────────────────>│                      │
 │                     │                     │ SELECT booking       │
 │                     │                     │─────────────────────>│
 │                     │                     │<─────────────────────│
 │                     │<────────────────────│                      │
 │  Hiển thị thông tin │                     │                      │
 │  đơn đặt            │                     │                      │
 │<────────────────────│                     │                      │
 │                     │                     │                      │
 │  Bấm "Check-in"     │                     │                      │
 │────────────────────>│                     │                      │
 │                     │ POST /api/bookings/  │                      │
 │                     │      :id/checkin     │                      │
 │                     │────────────────────>│                      │
 │                     │                     │ UPDATE bookings      │
 │                     │                     │ SET trangThai =      │
 │                     │                     │   'Đang sử dụng'     │
 │                     │                     │─────────────────────>│
 │                     │                     │ INSERT notification  │
 │                     │                     │─────────────────────>│
 │                     │<────────────────────│                      │
 │  Check-in thành     │                     │                      │
 │  công               │                     │                      │
 │<────────────────────│                     │                      │
 │                     │                     │                      │
 │         ─── Sau khi hết giờ chơi ───      │                      │
 │                     │                     │                      │
 │  Bấm "Check-out"    │                     │                      │
 │────────────────────>│                     │                      │
 │                     │ POST /api/bookings/  │                      │
 │                     │      :id/checkout    │                      │
 │                     │────────────────────>│                      │
 │                     │                     │ UPDATE bookings      │
 │                     │                     │ SET trangThai =      │
 │                     │                     │   'Hoàn thành'       │
 │                     │                     │─────────────────────>│
 │                     │                     │ INSERT notification  │
 │                     │                     │─────────────────────>│
 │                     │<────────────────────│                      │
 │  Check-out thành    │                     │                      │
 │  công               │                     │                      │
 │<────────────────────│                     │                      │
```

---

## §6. Sơ đồ máy trạng thái Booking (State Machine Diagram)

### Mô tả

Vòng đời của một đơn đặt sân từ lúc tạo đến khi kết thúc.

### Các trạng thái (States)

| Trạng thái | Mô tả |
|-----------|-------|
| **Đã cọc** (default) | Đơn mới tạo, đã đặt cọc 10% |
| **Đã thanh toán** | Khách đã thanh toán toàn bộ (hoặc phần còn lại) |
| **Đã đặt** | Đơn VIP auto-booking chưa thanh toán |
| **Đang sử dụng** | Khách đã check-in, đang chơi |
| **Hoàn thành** | Đã check-out, đơn hoàn tất |
| **Đã hủy** | Đơn bị hủy (user cancel / admin no-show / auto cancel) |

### Sự kiện chuyển trạng thái (Transitions)

```
                        ┌──────────────┐
                        │              │
                        │   Đã cọc     │────── User hủy (>3h trước) ──────┐
                        │  (khởi tạo)  │                                    │
                        │              │────── Admin hủy ──────────────────┤
                        └──────┬───────┘                                    │
                               │                                            │
                               │ Thanh toán đầy đủ                          │
                               ▼                                            │
                        ┌──────────────┐                                    │
                        │              │                                    │
                        │ Đã thanh toán│────── User hủy (>3h trước) ──────┤
                        │              │                                    │
                        │              │────── Admin hủy ──────────────────┤
                        └──────┬───────┘                                    │
                               │                                            │
                               │ Admin Check-in                             │
                               ▼                                            │
                        ┌──────────────┐                                    │
                        │              │                                    │
                        │ Đang sử dụng │────── Auto check-out ──────────┐  │
                        │              │       (hết giờ chơi)            │  │
                        └──────┬───────┘                                 │  │
                               │                                         │  │
                               │ Admin Check-out                         │  │
                               ▼                                         │  │
                        ┌──────────────┐                                 │  │
                        │              │                                 │  │
                        │  Hoàn thành  │                                 │  │
                        │              │                                 │  │
                        └──────────────┘                                 │  │
                                                                         │  │
                               ┌──────────────┐                          │  │
                               │              │                          │  │
                               │   Đã đặt     │── Admin hủy ────────────┤  │
                               │ (VIP auto)   │── Hết hạn auto ─────────┤  │
                               └──────────────┘                          │  │
                                                                         │  │
                        ┌────────────────────────────────────────────────┘  │
                        │          ┌────────────────────────────────────────┘
                        │          │
                        ▼          ▼
                 ┌──────────────────────┐
                 │                      │
                 │      Đã hủy          │
                 │  (terminal state)    │
                 │                      │
                 │  Lý do hủy:          │
                 │  - user_cancel       │
                 │  - admin_cancel      │
                 │  - noshow (auto)     │
                 │  - auto_past_booking │
                 │  - vip_conflict      │
                 └──────────────────────┘
```

### Chú thích transition

| Transition | Trigger | Actor | Điều kiện |
|-----------|---------|-------|-----------|
| Đã cọc → Đã thanh toán | Thanh toán phần còn lại | Customer | Đơn ở trạng thái Đã cọc |
| Đã cọc → Đã hủy | Hủy đơn | Customer / Admin | Customer: trước 3h giờ chơi |
| Đã cọc → Đang sử dụng | Check-in | Admin | Đúng ngày + đúng khung giờ |
| Đã thanh toán → Đã hủy | Hủy đơn | Customer / Admin | Customer: trước 3h giờ chơi |
| Đã thanh toán → Đang sử dụng | Check-in | Admin | Đúng ngày + đúng khung giờ |
| Đã thanh toán → Đã hủy (noshow) | Auto no-show | Scheduler | Quá 15 phút sau giờ bắt đầu |
| Đang sử dụng → Hoàn thành | Check-out | Admin / Scheduler | Scheduler: hiện tại > giờ kết thúc |
| Đã đặt (VIP) → Đã hủy | Hủy | Admin / Scheduler | Chưa thanh toán |
| Đã cọc → Đã hủy (noshow) | Auto no-show | Scheduler | Quá 15 phút sau giờ bắt đầu |

---

## §7. Sơ đồ thành phần Frontend (Component Tree Diagram)

### Cấu trúc thư mục và quan hệ component

```
App.tsx (Root)
├── <Suspense fallback={<PageLoader />}>
│   └── <Routes>
│       │
│       ├── [Auth Routes] (không cần layout)
│       │   ├── /login → <LoginPage />
│       │   ├── /forgot-password → <ForgotPasswordPage />
│       │   ├── /reset-password → <ResetPasswordPage />
│       │   └── /verify-otp → <OTPVerifyPage />
│       │
│       ├── [User Routes] → <UserLayout /> (Header + Footer + Outlet)
│       │   ├── / → <HomePage />
│       │   ├── /courts → <CourtListPage />
│       │   ├── /courts/:id → <CourtDetailPage />
│       │   ├── /payment/sepay-return → <PaymentReturnPage />
│       │   │
│       │   └── [Protected Routes] (require auth)
│       │       ├── /profile → <ProtectedRoute><ProfilePage /></ProtectedRoute>
│       │       ├── /my-bookings → <ProtectedRoute><MyBookingsPage /></ProtectedRoute>
│       │       ├── /booking/:id → <ProtectedRoute><BookingDetailPage /></ProtectedRoute>
│       │       └── /my-vouchers → <ProtectedRoute><VouchersPage /></ProtectedRoute>
│       │
│       ├── [Admin Routes] → <ProtectedRoute requireAdmin><AdminLayout /></ProtectedRoute>
│       │   ├── /admin → <DashboardPage />
│       │   ├── /admin/courts → <CourtsManagePage />
│       │   ├── /admin/timeslots → <TimeSlotsManagePage />
│       │   ├── /admin/bookings → <BookingsManagePage />
│       │   ├── /admin/services → <ServicesManagePage />
│       │   ├── /admin/users → <UsersManagePage />
│       │   ├── /admin/reports → <ReportsPage />
│       │   ├── /admin/discounts → <DiscountsManagePage />
│       │   ├── /admin/scanner → <QRScannerPage />
│       │   └── /admin/schedule-board → <ScheduleBoardPage />
│       │
│       ├── /forbidden → <ForbiddenPage />
│       └── * → <Navigate to="/" />
```

### Cây phụ thuộc State & Services

```
App.tsx
├── Stores (Zustand)
│   ├── authStore.ts
│   │   ├── state: { user, token, isAuthenticated }
│   │   └── actions: { login, logout, setUser, setToken }
│   └── themeStore.ts
│       └── state: { theme: 'light' | 'dark' }
│
├── Services (Axios-based)
│   └── services/index.ts
│       ├── authService        → /api/auth/*
│       ├── courtService       → /api/courts/*
│       ├── timeSlotService    → /api/courts/*/timeslots
│       ├── bookingService     → /api/bookings/*
│       ├── serviceService     → /api/admin/services
│       ├── reviewService      → /api/reviews/*
│       ├── discountService    → /api/admin/discounts
│       ├── adminService       → /api/dashboard, /api/reports, /api/schedule-board
│       ├── notificationService→ /api/notifications
│       └── uploadService      → /api/upload
│
├── Shared Components
│   ├── components/ui/
│   │   ├── Button.tsx        (có variant: primary, secondary, outline, ghost, destructive)
│   │   ├── Modal.tsx         (dialog có overlay + animation)
│   │   └── Skeleton.tsx      (placeholder loading)
│   ├── components/layout/
│   │   ├── UserLayout.tsx    (Header + Outlet + Footer)
│   │   └── AdminLayout.tsx   (Sidebar + Topbar + Outlet)
│   └── NotificationBell.tsx  (dropdown thông báo + unread count)
│
└── Libraries
    ├── react-router-dom v7   (routing)
    ├── @tanstack/react-query  (server state + caching)
    ├── axios                  (HTTP client)
    ├── zustand                (client state)
    ├── tailwindcss            (utility CSS)
    ├── recharts               (biểu đồ dashboard)
    ├── framer-motion          (animation)
    ├── sonner                 (toast notifications)
    ├── lucide-react           (icons)
    └── html5-qrcode           (QR scanner)
```

---

## §8. Sơ đồ triển khai (Deployment Diagram)

### Mô tả

Hệ thống được triển khai dưới dạng Docker container, chạy trên một máy chủ duy nhất (single server).

### Các node

```
┌─────────────────────────────────────────────────────────────────┐
│                   MÁY CHỦ (VPS / Cloud Server)                   │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Docker Container                           ││
│  │                    (node:18-alpine)                           ││
│  │                                                               ││
│  │  ┌─────────────────────────┐  ┌────────────────────────────┐ ││
│  │  │   Express Server        │  │   node-cron Scheduler      │ ││
│  │  │                         │  │                            │ ││
│  │  │   Port: 3000            │  │   * * * * *  (mỗi phút)   │ ││
│  │  │                         │  │   5 0 * * *  (hàng ngày)  │ ││
│  │  │   Static Files:         │  │   1 0 * * 1  (thứ 2)      │ ││
│  │  │   ├─ /uploads/          │  │                            │ ││
│  │  │   └─ /frontend/dist/    │  └────────────────────────────┘ ││
│  │  └─────────────────────────┘                                  ││
│  │                                                               ││
│  │  ┌──────────────────────────────────────────────────────────┐ ││
│  │  │                   File System                             │ ││
│  │  │  /app/server/public/uploads/                              │ ││
│  │  │  ├── avatars/     (RW - multer upload)                    │ ││
│  │  │  ├── courts/      (RW - multer upload)                    │ ││
│  │  │  └── general/     (RW - multer upload)                    │ ││
│  │  └──────────────────────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │              PostgreSQL Database (port 5432)                  ││
│  │                                                               ││
│  │  Database: pickleball_rework (hoặc từ DATABASE_URL)           ││
│  │  Connection: Pool (pg Pool)                                   ││
│  │                                                               ││
│  │  ┌─────────────────────────────────────────────────────────┐ ││
│  │  │ Tables: users, courts, timeslots, services, bookings,    │ ││
│  │  │ booking_services, payments, reviews, notifications,      │ ││
│  │  │ court_images, discounts, auto_booking_series             │ ││
│  │  └─────────────────────────────────────────────────────────┘ ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘

         │                          │
         │ HTTPS :443               │ TCP :5432
         │                          │
┌────────▼────────┐          ┌──────▼──────────┐
│   Người dùng    │          │  Admin Tools    │
│   (Browser)     │          │  (pgAdmin, psql)│
└─────────────────┘          └─────────────────┘
```

---

## §9. Sơ đồ xác thực & phân quyền (Authentication & Authorization)

### Mô tả luồng JWT authentication

```
User              Frontend                API                        DB
 │                   │                     │                         │
 │  Nhập email+pass  │                     │                         │
 │──────────────────>│                     │                         │
 │                   │ POST /api/auth/login │                         │
 │                   │────────────────────>│                         │
 │                   │                     │ SELECT user by email    │
 │                   │                     │────────────────────────>│
 │                   │                     │<────────────────────────│
 │                   │                     │ bcrypt.compare(pass)    │
 │                   │                     │                         │
 │                   │                     │ [Nếu match]             │
 │                   │                     │ jwt.sign({id,email,role})│
 │                   │                     │ + JWT_SECRET            │
 │                   │                     │ expiry: 7d              │
 │                   │                     │                         │
 │                   │<────────────────────│ { token, user }         │
 │                   │                     │                         │
 │                   │ authStore.setAuth() │                         │
 │                   │ Zustand persist     │                         │
 │                   │                     │                         │
 │  Đăng nhập TC     │                     │                         │
 │<──────────────────│                     │                         │
 │                   │                     │                         │
 │  ─── Các request sau ───               │                         │
 │                   │                     │                         │
 │  Gọi API bất kỳ   │                     │                         │
 │──────────────────>│                     │                         │
 │                   │ Axios Interceptor   │                         │
 │                   │ gắn header:         │                         │
 │                   │ Authorization:      │                         │
 │                   │ Bearer <token>      │                         │
 │                   │ x-user-id: <id>     │                         │
 │                   │────────────────────>│                         │
 │                   │                     │ authenticate middleware │
 │                   │                     │ jwt.verify(token)      │
 │                   │                     │ gán req.user           │
 │                   │                     │                         │
 │                   │                     │ [Nếu route cần admin]  │
 │                   │                     │ requireAdmin middleware │
 │                   │                     │ check req.user.role    │
 │                   │                     │                         │
 │                   │                     │ Xử lý request          │
 │                   │<────────────────────│                         │
 │                   │                     │                         │
 │  ─── Token hết hạn ───                │                         │
 │                   │                     │                         │
 │                   │ API trả về 401      │                         │
 │                   │<────────────────────│                         │
 │                   │ Axios Resp Interceptor                       │
 │                   │ authStore.logout()  │                         │
 │                   │ redirect /login     │                         │
 │                   │ toast "Vui lòng     │                         │
 │                   │  đăng nhập lại"     │                         │
 │<──────────────────│                     │                         │
```

---

## §10. Sơ đồ luồng thanh toán (Payment Flow)

### Mô tả

Hệ thống hỗ trợ 2 hình thức thanh toán:
1. **Tiền mặt** - Admin xác nhận tại quầy
2. **Chuyển khoản qua SePay** - Webhook tự động xác nhận

### Luồng (Sequence)

```
User           Frontend          API              DB           SePay
 │                │               │                │              │
 │  Chọn TT       │               │                │              │
 │────────────────>│               │                │              │
 │                │               │                │              │
 │  [Tiền mặt]    │               │                │              │
 │  ─────────     │               │                │              │
 │                │ POST /api/    │                │              │
 │                │ bookings      │                │              │
 │                │──────────────>│                │              │
 │                │               │ INSERT booking │              │
 │                │               │ (trangThai=    │              │
 │                │               │  'Đã cọc')    │              │
 │                │               │───────────────>│              │
 │                │               │ INSERT payment │              │
 │                │               │ (loaiThanhToan │              │
 │                │               │  = 'Tiền mặt')│              │
 │                │               │───────────────>│              │
 │                │<──────────────│                │              │
 │  Hiển thị QR    │               │                │              │
 │<────────────────│               │                │              │
 │                │               │                │              │
 │  [Chuyển khoản SePay]          │                │              │
 │  ───────────────────           │                │              │
 │                │ POST /api/    │                │              │
 │                │ bookings      │                │              │
 │                │ (paymentMethod│                │              │
 │                │  = 'sepay')   │                │              │
 │                │──────────────>│                │              │
 │                │               │ INSERT booking │              │
 │                │               │ (trangThai=    │              │
 │                │               │  'Đã cọc')    │              │
 │                │               │───────────────>│              │
 │                │               │                │              │
 │                │<──────────────│                │              │
 │  Hiển thị QR +  │               │                │              │
 │  thông tin CK   │               │                │              │
 │<────────────────│               │                │              │
 │                │               │                │              │
 │  Chuyển khoản   │               │                │              │
 │  qua ngân hàng  │               │                │              │
 │──────────────────────────────────────────────────────────────>│
 │                │               │                │              │
 │                │               │  Webhook       │              │
 │                │               │  POST /api/    │              │
 │                │               │  payment/      │              │
 │                │               │  sepay-webhook │              │
 │                │               │<─────────────────────────────│
 │                │               │                │              │
 │                │               │ UPDATE booking │              │
 │                │               │ SET trangThai =│              │
 │                │               │ 'Đã thanh toán'│              │
 │                │               │───────────────>│              │
 │                │               │ INSERT payment │              │
 │                │               │ (loaiThanhToan │              │
 │                │               │  = 'Chuyển     │              │
 │                │               │   khoản')     │              │
 │                │               │───────────────>│              │
 │                │               │ INSERT noti    │              │
 │                │               │───────────────>│              │
 │                │               │                │              │
 │  Nhận thông báo │               │                │              │
 │  thanh toán TC  │               │                │              │
 │<────────────────│<──────────────│                │              │
```

---

## §11. Sơ đồ VIP Auto-Booking (Activity Diagram)

### Mô tả

Khách VIP được đặt sân tự động 30 ngày (từ thứ 2 hiện tại tới 4 tuần sau). Hệ thống tự động tạo series auto_booking_series và tất cả đơn trong series.

### Luồng hoạt động

```
┌──────────────────────────────────────────────────────────────────┐
│                     VIP AUTO-BOOKING FLOW                        │
└──────────────────────────────────────────────────────────────────┘

  [Admin bật VIP cho User]
         │
         ▼
  [User VIP chọn sân + khung giờ + dịch vụ]
         │
         ▼
  ┌─────────────────────────┐
  │ POST /api/bookings       │
  │ (isAutoBooking = true)   │
  │ (startDate, endDate,    │
  │  khungGioIds, services) │
  └────────────┬────────────┘
               │
               ▼
  ┌──────────────────────────────────────┐
  │ BEGIN TRANSACTION                    │
  │                                      │
  │ 1. INSERT auto_booking_series        │
  │    (startDate, endDate,              │
  │     khungGioIds JSONB,               │
  │     repeatServices, servicePolicy)   │
  │                                      │
  │ 2. FOR EACH day FROM startDate       │
  │    TO endDate (30 days):             │
  │    ┌──────────────────────────────┐  │
  │    │ FOR EACH khungGioId:         │  │
  │    │   ┌────────────────────────┐ │  │
  │    │   │ INSERT booking          │ │  │
  │    │   │ - nguoiDungId = VIP    │ │  │
  │    │   │ - sanId = sanId        │ │  │
  │    │   │ - khungGioId = slotId  │ │  │
  │    │   │ - ngayChoi = date      │ │  │
  │    │   │ - tongTien = mucGia    │ │  │
  │    │   │ - tienDaCoc = 10%      │ │  │
  │    │   │ - trangThai = 'Đã cọc' │ │  │
  │    │   │ - isAutoBooking = true │ │  │
  │    │   │ - autoBookingSeriesId  │ │  │
  │    │   └────────────────────────┘ │  │
  │    │   IF repeatServices:         │  │
  │    │     INSERT booking_services  │  │
  │    └──────────────────────────────┘  │
  │                                      │
  │ 3. UPDATE auto_booking_series        │
  │    SET totalAmount                   │
  │                                      │
  │ COMMIT                               │
  └──────────────────────────────────────┘
               │
               ▼
  [Series + 30 ngày booking được tạo]
               │
               ▼
  ┌──────────────────────────────────────┐
  │ Cron Job: Hàng thứ 2 lúc 00:01       │
  │ (processVipAutoBooking - legacy)     │
  │                                      │
  │ - Kiểm tra VIP cũ (chưa có series)   │
  │ - Tạo thêm booking cho tuần mới       │
  │ - Nếu slot đã có người đặt → notify │
  └──────────────────────────────────────┘
               │
               ▼
  ┌──────────────────────────────────────┐
  │ Cron Job: Hàng ngày lúc 00:05        │
  │ (autoCancelPastBookings)             │
  │                                      │
  │ - Hủy booking auto đã quá ngày       │
  │   mà chưa thanh toán                 │
  └──────────────────────────────────────┘
```

---

## §12. Sơ đồ Scheduler & Cron Jobs (Activity Diagram)

### Mô tả

Hệ thống có 3 cron job chạy trong cùng process Express (dùng thư viện `node-cron`).

### Danh sách Cron Jobs

| # | Cron Expression | Tần suất | Hàm | Chức năng |
|---|----------------|----------|-----|-----------|
| 1 | `* * * * *` | Mỗi phút | `handleBookingStatus()` | Tự động check-out + hủy no-show |
| 2 | `5 0 * * *` | 00:05 hàng ngày | `autoCancelPastBookings()` | Hủy auto-booking quá hạn |
| 3 | `1 0 * * 1` | 00:01 thứ 2 | `processVipAutoBooking()` | Tạo booking cho VIP tuần mới (legacy) |

### Luồng Cron Job 1: handleBookingStatus (mỗi phút)

```
  [Mỗi phút]
       │
       ▼
  ┌─────────────────────────────────────┐
  │ handleBookingStatus()               │
  │                                      │
  │ ┌──────────────────────────────────┐ │
  │ │ SELECT * FROM bookings           │ │
  │ │ WHERE trangThai = 'Đang sử dụng' │ │
  │ │ AND giờ hiện tại > giờ kết thúc  │ │
  │ └────────────┬─────────────────────┘ │
  │              │                       │
  │              ▼                       │
  │ ┌──────────────────────────────────┐ │
  │ │ FOR EACH booking:                │ │
  │ │   UPDATE trangThai = 'Hoàn thành'│ │
  │ │   INSERT notification            │ │
  │ └──────────────────────────────────┘ │
  │                                      │
  │ ┌──────────────────────────────────┐ │
  │ │ SELECT * FROM bookings           │ │
  │ │ WHERE trangThai IN               │ │
  │ │   ('Đã thanh toán','Đã cọc',    │ │
  │ │    'Đã đặt')                     │ │
  │ │ AND (now - giờ bắt đầu) > 15 phút│ │
  │ │ AND ngayChoi = hôm nay           │ │
  │ └────────────┬─────────────────────┘ │
  │              │                       │
  │              ▼                       │
  │ ┌──────────────────────────────────┐ │
  │ │ FOR EACH booking:                │ │
  │ │   UPDATE trangThai = 'Đã hủy'    │ │
  │ │   SET ghiChu = 'No-show'         │ │
  │ │   INSERT notification            │ │
  │ │   (loại: noshow)                 │ │
  │ └──────────────────────────────────┘ │
  └─────────────────────────────────────┘
```

### Luồng Cron Job 2: autoCancelPastBookings (00:05 mỗi ngày)

```
  [00:05 mỗi ngày]
       │
       ▼
  ┌─────────────────────────────────────┐
  │ autoCancelPastBookings()            │
  │                                      │
  │ ┌──────────────────────────────────┐ │
  │ │ SELECT * FROM bookings           │ │
  │ │ WHERE isAutoBooking = true       │ │
  │ │ AND ngayChoi < hôm nay           │ │
  │ │ AND trangThai NOT IN             │ │
  │ │   ('Đã hủy', 'Hoàn thành',      │ │
  │ │    'Đang sử dụng')               │ │
  │ └────────────┬─────────────────────┘ │
  │              │                       │
  │              ▼                       │
  │ ┌──────────────────────────────────┐ │
  │ │ FOR EACH booking:                │ │
  │ │   UPDATE trangThai = 'Đã hủy'    │ │
  │ │   INSERT notification            │ │
  │ └──────────────────────────────────┘ │
  └─────────────────────────────────────┘
```

### Luồng Cron Job 3: processVipAutoBooking (00:01 thứ 2)

```
  [00:01 thứ 2 hàng tuần]
       │
       ▼
  ┌─────────────────────────────────────────┐
  │ processVipAutoBooking(force=false)      │
  │ (Legacy: cho VIP chưa có series)        │
  │                                          │
  │ ┌──────────────────────────────────────┐ │
  │ │ SELECT * FROM users                  │ │
  │ │ WHERE isVIP = true                   │ │
  │ │ AND id NOT IN (SELECT nguoiDungId    │ │
  │ │   FROM auto_booking_series           │ │
  │ │   WHERE trangThai = 'Active')        │ │
  │ └────────────┬─────────────────────────┘ │
  │              │                           │
  │              ▼                           │
  │ ┌──────────────────────────────────────┐ │
  │ │ FOR EACH VIP user:                   │ │
  │ │   Lấy danh sách slot VIP đã chọn    │ │
  │ │   FOR EACH ngày (30 ngày tới):       │ │
  │ │     FOR EACH slot:                   │ │
  │ │       Kiểm tra xung đột              │ │
  │ │       IF đã có người đặt:            │ │
  │ │         Gửi thông báo xung đột       │ │
  │ │         Tắt auto-booking             │ │
  │ │       ELSE:                          │ │
  │ │         INSERT booking               │ │
  │ │         (trangThai = 'Đã cọc')       │ │
  │ └──────────────────────────────────────┘ │
  └─────────────────────────────────────────┘
```

---

## Phụ lục: Công nghệ sử dụng

| Tầng | Công nghệ | Phiên bản |
|------|-----------|-----------|
| Frontend Framework | React | 18.3 |
| Build Tool | Vite | 5.4 |
| Ngôn ngữ FE | TypeScript | 5.6 |
| CSS Framework | Tailwind CSS | 3.4 |
| UI Primitives | Radix UI | 1.x |
| Icons | Lucide React | 0.460 |
| Charts | Recharts | 3.6 |
| State Management | Zustand | 5.0 |
| Server State | TanStack React Query | 5.60 |
| HTTP Client | Axios | 1.7 |
| Routing | React Router DOM | 7.0 |
| Animation | Framer Motion | 11.11 |
| Backend Runtime | Node.js | 18 |
| Backend Framework | Express.js | 4.18 |
| Database | PostgreSQL | (qua pg 8.11) |
| Auth | JWT (jsonwebtoken) | 9.0 |
| Password Hash | bcryptjs | 2.4 |
| Scheduler | node-cron | 3.0 |
| File Upload | multer | 1.4 |
| Excel Export | exceljs | 4.4 |
| QR Code | qrcode | 1.5 |
| QR Scanner | html5-qrcode | 2.3 |
| Container | Docker (node:18-alpine) | - |

---

## Phụ lục: API Endpoints

### Auth (`/api/auth`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| POST | /register | No | Đăng ký |
| POST | /verify-register | No | Xác thực OTP |
| POST | /login | No | Đăng nhập |
| POST | /verify-login | No | Xác thực OTP login |
| POST | /forgot-password | No | Gửi OTP reset |
| POST | /reset-password | No | Reset mật khẩu |
| GET | /profile | Yes | Xem profile |
| PUT | /profile | Yes | Cập nhật profile |

### Courts (`/api/courts`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| GET | / | No | Danh sách sân |
| GET | /:id | No | Chi tiết sân |
| POST | / | Admin | Tạo sân |
| PUT | /:id | Admin | Cập nhật sân |
| DELETE | /:id | Admin | Xóa sân |

### Bookings (`/api/bookings`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| POST | / | Yes | Tạo đơn đặt sân |
| GET | /my | Yes | Đơn của tôi |
| GET | /:id | Yes | Chi tiết đơn |
| POST | /:id/cancel | Yes | Hủy đơn |
| GET | /:id/qr | Yes | Mã QR |
| GET | / | Admin | Tất cả đơn |
| POST | /:id/checkin | Admin | Check-in |
| POST | /:id/checkout | Admin | Check-out |
| POST | /:id/noshow | Admin | Đánh dấu vắng mặt |

### Admin (`/api/admin` + `/api`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| GET | /dashboard | Admin | Dashboard |
| GET | /reports | Admin | Báo cáo |
| GET | /reports/export | Admin | Xuất Excel |
| GET | /schedule-board | Admin | Lịch sân |
| GET/POST/PUT/DELETE | /services | Admin | CRUD dịch vụ |
| GET/POST/PUT/DELETE | /discounts | Admin | CRUD mã giảm giá |
| POST | /discounts/validate | Yes | Validate mã |
| GET | /discounts/my | Yes | Mã của tôi |
| GET | /notifications | Yes | Thông báo |
| PATCH | /notifications/:id/read | Yes | Đánh dấu đã đọc |
| PATCH | /notifications/read-all | Yes | Đọc tất cả |
| POST | /trigger-cancel-past | Admin | Trigger hủy |
| POST | /trigger-vip-auto-book | Admin | Trigger VIP |

### Users (`/api/users`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| GET | / | Admin | Danh sách user |
| PUT | /:id | Admin | Cập nhật user |
| PATCH | /:id/toggle-status | Admin | Khóa/Mở khóa |
| PATCH | /:id/toggle-vip | Admin | Bật/Tắt VIP |

### Reviews (`/api/reviews`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| GET | /court/:courtId | No | Đánh giá của sân |
| POST | / | Yes | Tạo đánh giá |

### Upload (`/api/upload`)
| Method | Path | Auth | Mô tả |
|--------|------|------|-------|
| POST | / | Admin | Upload file |
| POST | /court-images | Admin | Upload ảnh sân |
