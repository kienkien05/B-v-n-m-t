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
| **11** | **Activity: Đăng ký tài khoản (Register)** | **Activity** | **§11** |
| **12** | **Activity: Đặt sân (Create Booking)** | **Activity** | **§12** |
| **13** | **Activity: Hủy đơn (Cancel Booking)** | **Activity** | **§13** |
| **14** | **Activity: Check-in / Check-out** | **Activity** | **§14** |
| **15** | **Activity: VIP Auto-Booking 30 ngày** | **Activity** | **§15** |
| **16** | **Activity: Scheduler Cron Jobs** | **Activity** | **§16** |
| **17** | **Activity: Quản lý sân (CRUD Court)** | **Activity** | **§17** |
| **18** | **Activity: Quản lý mã giảm giá** | **Activity** | **§18** |
| **19** | **Activity: Xác thực OTP** | **Activity** | **§19** |
| **20** | **Activity: Loyalty Rewards** | **Activity** | **§20** |

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

## §11. Activity Diagram: Đăng ký tài khoản (Register Flow)

### Mô tả

Luồng đăng ký tài khoản mới với OTP verification. User điền form → nhận OTP → xác thực → hoàn tất đăng ký.

### Quy ước ký hiệu

| Ký hiệu | Ý nghĩa |
|----------|---------|
| ● | Nút bắt đầu (Initial Node) |
| ◎ | Nút kết thúc (Final Node) |
| [ĐK] | Điều kiện rẽ nhánh (Guard) |
| ◇ | Điểm quyết định (Decision Node) |
| █ | Thanh Fork/Join (song song) |

### Swimlanes

- **User**: Người dùng trên trình duyệt
- **Frontend**: React SPA
- **Backend**: Express API Server
- **Database**: PostgreSQL

### Các bước thực hiện

```
● Bắt đầu
│
├──[Swimlane: User]──
│   │
│   ├─(1) Vào trang Login
│   │
│   ├─(2) Chuyển sang tab "Đăng ký" (Register)
│   │
│   ├─(3) Nhập form đăng ký:
│   │      - Họ tên (full_name)
│   │      - Email
│   │      - Số điện thoại (tùy chọn)  
│   │      - Mật khẩu (password)
│   │      - Xác nhận mật khẩu (confirm_password)
│   │
│   ├─(4) Click nút "Đăng ký"
│   │
│   └──[Chuyển sang Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(5) Validate form client-side
│   │      - Tất cả trường bắt buộc đã điền?
│   │      - Email đúng định dạng?
│   │      - password === confirm_password?
│   │
│   ├─◇ (5a) [Validation thất bại]
│   │   └─→ Hiển thị lỗi, quay lại (3)
│   │
│   ├─◇ (5b) [Validation OK]
│   │   └─→ POST /api/auth/register
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(6) Nhận request { email, password, confirm_password, full_name, phone_number }
│   │
│   ├─◇ (6a) [Thiếu email/password/full_name] → 400 "Vui lòng nhập đầy đủ thông tin"
│   │
│   ├─◇ (6b) [password !== confirm_password] → 400 "Mật khẩu xác nhận không khớp"
│   │
│   ├─(7) SELECT * FROM users WHERE email = $1
│   │
│   ├─◇ (7a) [Email đã tồn tại] → 400 "Email đã được sử dụng"
│   │
│   ├─◇ (7b) [phone_number có giá trị]
│   │   └─→ SELECT * FROM users WHERE soDienThoai = $1
│   │       └─◇ [SĐT đã tồn tại] → 400 "Số điện thoại đã được sử dụng"
│   │
│   ├─(8) Tạo OTP 6 số ngẫu nhiên (generateOTP())
│   │
│   ├─(9) Lưu vào otpStore Map:
│   │      key: "register:{email}"
│   │      value: { otp, password, full_name, phone_number, expires: now + 10phút }
│   │
│   ├─(10) console.log OTP (chỉ dev mode)
│   │
│   ├─(11) Response: { message: "Mã OTP đã được gửi (kiểm tra console)" }
│   │
│   └──[Trả về Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(12) Hiển thị form nhập OTP (6 chữ số)
│   │
│   ├─(13) Hiển thị nút "Gửi lại OTP" + đồng hồ đếm ngược 10 phút
│   │
│   └──[User nhập OTP]──
│
├──[Swimlane: User]──
│   │
│   ├─(14) Lấy OTP từ console server
│   │
│   ├─(15) Nhập OTP vào form xác thực
│   │
│   ├─(16) Click nút "Xác thực" (Verify)
│   │
│   └──[Chuyển sang Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(17) POST /api/auth/verify-register
│   │       Body: { email, otp, password, full_name }
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(18) otpStore.get("register:{email}")
│   │
│   ├─◇ (18a) [OTP không tồn tại hoặc không khớp hoặc hết hạn]
│   │   └─→ 400 "Mã OTP không chính xác hoặc đã hết hạn"
│   │
│   ├─◇ (18b) [OTP hợp lệ]
│   │   │
│   │   ├─(19) bcrypt.hash(password, 10) → hashedPw
│   │   │
│   │   ├─(20) INSERT INTO users (hoTen, email, matKhau, soDienThoai)
│   │   │       VALUES ($1, $2, $3, $4) RETURNING id, hoTen, email, ...
│   │   │
│   │   ├─(21) Tạo mã chào mừng WELCOME{userId}:
│   │   │       INSERT INTO discounts (code, noiDung, loaiGiamGia='percentage', mucGiamGia=20,
│   │   │         ngayBatDau=NOW(), ngayKetThuc=NOW()+30days, soLuongBanDau=1, nguoiDungId=userId)
│   │   │
│   │   ├─(22) jwt.sign({ id: user.id, email, role: user.vaiTro }, JWT_SECRET, { expiresIn: '7d' })
│   │   │
│   │   ├─(23) otpStore.delete("register:{email}")
│   │   │
│   │   └─(24) Response 200: { data: { token, user: { id, email, full_name, ... } } }
│   │
│   └──[Trả về Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(25) Lưu token + user vào authStore (Zustand persist → localStorage)
│   │
│   ├─(26) Hiển thị toast "Đăng ký thành công!"
│   │
│   ├─(27) Chuyển hướng về trang Home (/)
│   │
│   └──→ ◎ Kết thúc (Đăng ký thành công)
│
└──[Trường hợp lỗi]──
    │
    ├─→ 400/500: Hiển thị toast lỗi, quay lại form đăng ký
    │
    └─→ ◎ Kết thúc (Đăng ký thất bại)
```

---

## §12. Activity Diagram: Đặt sân (Create Booking Flow)

### Mô tả

Luồng người dùng đặt sân Pickleball: chọn sân → chọn ngày → chọn khung giờ → thêm dịch vụ → áp mã giảm giá → xác nhận đặt → tạo QR.

### Swimlanes

- **Customer**: Người dùng đã đăng nhập
- **Frontend**: React SPA  
- **Backend**: Express API (có transaction)
- **Database**: PostgreSQL

### Các bước thực hiện

```
● Bắt đầu (Customer đã đăng nhập)
│
├──[Swimlane: Customer]──
│   │
│   ├─(1) Vào trang danh sách sân (/courts)
│   │
│   ├─(2) Chọn 1 sân → vào trang chi tiết (/courts/:id)
│   │
│   ├─(3) Chọn ngày chơi (date picker)
│   │
│   ├─(4) Hệ thống hiển thị các khung giờ khả dụng
│   │
│   ├─(5) Chọn 1 hoặc nhiều khung giờ (checkbox)
│   │     Mỗi khung giờ hiển thị: giờ bắt đầu-kết thúc + giá
│   │
│   ├─(6) [Tùy chọn] Chọn dịch vụ đi kèm:
│   │     - Chọn loại dịch vụ (vợt, bóng, nước uống...)
│   │     - Nhập số lượng
│   │
│   ├─(7) [Tùy chọn] Nhập mã giảm giá (nếu có)
│   │
│   ├─(8) Click nút "Đặt sân"
│   │
│   └──[Chuyển sang Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(9) Validate:
│   │     - Đã chọn sân? Đã chọn ngày? Đã chọn ít nhất 1 khung giờ?
│   │
│   ├─◇ (9a) [Thiếu thông tin] → Hiển thị lỗi, quay lại form
│   │
│   ├─◇ (9b) [Có mã giảm giá]
│   │   └─→ POST /api/discounts/validate
│   │       { code, totalAmount: tổng tiền hiện tại, courtId }
│   │       │
│   │       ├─◇ [Mã không hợp lệ] → Hiển thị lỗi, xóa mã
│   │       │
│   │       └─◇ [Mã hợp lệ] → Hiển thị discountAmount, cập nhật tổng tiền
│   │
│   ├─◇ (9c) [Validation OK]
│   │   └─→ POST /api/bookings
│   │       Body: { sanId, ngayChoi, khungGioIds, dichVu, maGiamGia, phuongThuc, isAutoBooking?, repeatServices? }
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(10) pool.connect() → BEGIN TRANSACTION
│   │
│   ├─(11) Validate input:
│   │       - sanId và ngayChoi và khungGioIds không rỗng?
│   │       - Nếu isAutoBooking: user có isVIP = true?
│   │
│   ├─◇ (11a) [Thiếu input] → ROLLBACK → 400
│   ├─◇ (11b) [Không phải VIP mà bật auto] → ROLLBACK → 403
│   │
│   ├─(12) Kiểm tra trạng thái sân:
│   │       SELECT trangThai FROM courts WHERE id = sanId
│   │
│   ├─◇ (12a) [Sân không tồn tại] → ROLLBACK → 404
│   ├─◇ (12b) [Sân đang Bảo trì] → ROLLBACK → 400 "Sân đang bảo trì"
│   ├─◇ (12c) [Sân không khả dụng (Ẩn...)] → ROLLBACK → 400
│   │
│   ├─(13) Kiểm tra xung đột (có FOR UPDATE để khóa row):
│   │       SELECT FROM bookings b JOIN timeslots t
│   │       WHERE b.sanId = sanId AND b.ngayChoi = ANY(dates)
│   │       AND b.khungGioId = ANY(slotIds)
│   │       AND b.trangThai NOT IN ('Đã hủy')
│   │       FOR UPDATE  ← ngăn race condition
│   │
│   ├─◇ (13a) [Có xung đột] → ROLLBACK → 409 "Khung giờ X ngày Y đã có người đặt"
│   │
│   ├─(14) Kiểm tra thời gian:
│   │       - ngayChoi < hôm nay? → ROLLBACK → 400 "Không thể đặt ngày quá khứ"
│   │       - Nếu ngayChoi = hôm nay: kiểm tra từng khung giờ
│   │         IF now > gioBatDau + 15 phút → ROLLBACK → 400 "Quá thời gian cho phép"
│   │
│   ├─(15) Lấy thông tin khung giờ + tính giá sân:
│   │       SELECT * FROM timeslots WHERE sanId = sanId AND id = ANY(slotIds)
│   │       courtPricePerSession = SUM(mucGia)
│   │
│   ├─(16) [Nếu có dịch vụ] Lấy thông tin dịch vụ + kiểm tra tồn kho:
│   │       SELECT * FROM services WHERE id = ANY(serviceIds)
│   │       - Kiểm tra trạng thái 'Còn hàng'
│   │       - Kiểm tra soLuongTon >= requiredQty
│   │       IF không đủ → ROLLBACK → 400 "Dịch vụ X không đủ tồn kho"
│   │
│   ├─(17) Tính tổng tiền:
│   │       subTotal = courtPrice + servicesPrice
│   │
│   ├─(18) [Nếu có mã giảm giá] Áp dụng:
│   │       SELECT * FROM discounts WHERE code = maGiamGia AND trangThai='Active'
│   │       - Kiểm tra ngày hiệu lực
│   │       - Kiểm tra số lượng còn
│   │       - Tính discountAmount (percentage: subTotal * mucGiamGia/100, fixed: min(mucGiamGia, subTotal))
│   │       - UPDATE discounts SET soLuongDaDung = soLuongDaDung + 1
│   │       IF mã không hợp lệ → ROLLBACK → 400 "Mã giảm giá không hợp lệ hoặc đã hết hạn"
│   │
│   ├─(19) totalPrice = subTotal - discountAmount
│   │
│   ├─(20) [Nếu isAutoBooking] Tạo auto_booking_series:
│   │       INSERT INTO auto_booking_series (nguoiDungId, sanId, khungGioIds, startDate, endDate, ...)
│   │
│   ├─(21) Tạo danh sách booking (mỗi slot/ngày = 1 bản ghi):
│   │       FOR EACH (date, slotId) IN plannedBookings:
│   │         bookingStatus = (phuongThuc === 'cash') ? 'Đã đặt' : 'Đã thanh toán'
│   │         INSERT INTO bookings (nguoiDungId, sanId, khungGioId, ngayChoi, tongTien, tienDaCoc, giaGoc, tienGiam, trangThai, isAutoBooking, autoBookingSeriesId, maGiamGia)
│   │         INSERT INTO payments (donDatId, soTien, loaiThanhToan, trangThai)
│   │         [Nếu slot đầu + có dịch vụ] INSERT INTO booking_services
│   │
│   ├─(22) Trừ kho dịch vụ:
│   │       UPDATE services SET soLuongTon = soLuongTon - soLuong WHERE id = dichVuId
│   │
│   ├─(23) INSERT notification (booking_confirmed / vip_auto_success)
│   │
│   ├─(24) [Loyalty Rewards] Kiểm tra mốc 3 đơn:
│   │       IF tổng đơn của user (không tính đã hủy) đạt bội số của 3:
│   │         INSERT INTO discounts (code='LTY10-XXXX', mucGiamGia=10, nguoiDungId=userId)
│   │         INSERT notification (promotion)
│   │
│   ├─(25) COMMIT TRANSACTION
│   │
│   └─(26) Response 201: { data: { bookingIds, totalPrice, ... } }
│
├──[Swimlane: Frontend]──
│   │
│   ├─(27) Hiển thị toast "Đặt sân thành công!"
│   │
│   ├─(28) Hiển thị modal xác nhận với:
│   │       - Danh sách đơn đã tạo
│   │       - Tổng tiền
│   │       - Mã QR check-in (gọi GET /api/bookings/:id/qr)
│   │
│   └──→ ◎ Kết thúc (Đặt sân thành công)
│
└──[Trường hợp lỗi]──
    │
    ├─→ ROLLBACK transaction → Response lỗi → Toast lỗi
    │
    └─→ ◎ Kết thúc (Đặt sân thất bại)
```

---

## §13. Activity Diagram: Hủy đơn (Cancel Booking Flow)

### Mô tả

Luồng người dùng hủy đơn đặt sân. Có 3 loại hủy: (A) User tự hủy (trước 3h), (B) Admin hủy, (C) Hệ thống tự động hủy (no-show / quá hạn).

### Swimlanes

- **Actor**: Customer / Admin / Scheduler
- **Backend**: Express API
- **Database**: PostgreSQL

### A. User tự hủy đơn

```
● Bắt đầu (Customer đã đăng nhập)
│
├──[Swimlane: Customer]──
│   │
│   ├─(A1) Vào trang "Đơn của tôi" (/my-bookings)
│   │
│   ├─(A2) Tìm đơn muốn hủy → Click "Hủy đơn"
│   │
│   ├─(A3) Hiển thị modal xác nhận hủy:
│   │       - Cảnh báo: "Nếu đơn đã thanh toán/cọc, khoản tiền không được hoàn lại"
│   │       - Nếu còn dưới 3h đến giờ chơi: "Đã quá thời gian cho phép hủy"
│   │
│   ├─(A4) Click "Xác nhận hủy"
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(A5) POST /api/bookings/:id/cancel
│   │       (middleware: authenticate → req.user.id)
│   │
│   ├─(A6) SELECT b.*, t.gioBatDau FROM bookings b JOIN timeslots t
│   │       WHERE b.id = :id AND b.nguoiDungId = req.user.id
│   │
│   ├─◇ (A6a) [Không tìm thấy đơn] → 404
│   │
│   ├─◇ (A6b) [trangThai KHÔNG trong ('Đã thanh toán', 'Đã đặt', 'Đã cọc')]
│   │   └─→ 400 "Chỉ có thể hủy đơn ở trạng thái Đã thanh toán, Đã cọc hoặc Đã đặt"
│   │
│   ├─(A7) Tính thời gian còn lại:
│   │       hoursLeft = (ngayChoi + gioBatDau - now) / 3600000
│   │
│   ├─◇ (A7a) [hoursLeft < 3] → 400 "Đã quá thời gian cho phép hủy sân (Yêu cầu hủy trước 3 tiếng)"
│   │
│   ├─◇ (A7b) [hoursLeft >= 3] → Cho phép hủy:
│   │   │
│   │   ├─(A8) UPDATE bookings SET trangThai = 'Đã hủy', ghiChu = 'Khách tự hủy...', updated_at = NOW()
│   │   │
│   │   ├─(A9) Hoàn lại stock dịch vụ:
│   │   │       SELECT dichVuId, soLuong FROM booking_services WHERE donDatId = bookingId
│   │   │       FOR EACH: UPDATE services SET soLuongTon = soLuongTon + soLuong
│   │   │
│   │   ├─(A10) Hoàn lại lượt dùng mã giảm giá:
│   │   │        UPDATE discounts SET soLuongDaDung = GREATEST(soLuongDaDung - 1, 0)
│   │   │        WHERE code = booking.maGiamGia AND soLuongDaDung > 0
│   │   │
│   │   ├─(A11) INSERT INTO notifications (nguoiDungId, tieuDe='Hủy đặt sân', noiDung, loai='booking_cancelled')
│   │   │
│   │   └─(A12) Response: { message: "Hủy đặt sân thành công..." }
│   │
│   └──→ ◎ Kết thúc (Hủy thành công)
│
└──[Trường hợp lỗi]──
    └─→ 400/500 → Toast lỗi → ◎ Kết thúc (Hủy thất bại)
```

### B. Admin hủy đơn (No-show hoặc hủy thủ công)

```
● Bắt đầu (Admin đã đăng nhập)
│
├──[Swimlane: Admin]──
│   │
│   ├─(B1) Vào trang quản lý đơn (/admin/bookings)
│   │
│   ├─(B2) Tìm đơn → Click "Đánh dấu vắng mặt" (No-show)
│   │     HOẶC "Hủy đơn" (Admin cancel)
│   │
│   ├─(B3) Xác nhận trong modal
│   │
│   └──[Backend xử lý]──
│       │
│       ├─(B4) Kiểm tra quyền admin
│       │
│       ├─(B5) Kiểm tra trạng thái đơn (chỉ hủy Đã TT/Đã cọc/Đã đặt)
│       │
│       ├─(B6) cancelBookingWithReason(pool, booking, 'ADMIN_NOSHOW')
│       │       - UPDATE bookings SET trangThai='Đã hủy', ghiChu='Khách không đến...'
│       │       - Hoàn stock dịch vụ (như A9)
│       │       - Hoàn discount usage (như A10)
│       │       - INSERT notification (loại='noshow')
│       │
│       └─→ ◎ Kết thúc
```

### C. Hệ thống tự động hủy (Scheduler)

```
● Bắt đầu (Trigger bởi cron job)
│
├──[Swimlane: Scheduler]──
│   │
│   ├─(C1) Cron job handleBookingStatus() chạy mỗi phút
│   │
│   ├─(C2) Tìm booking no-show:
│   │       SELECT * FROM bookings WHERE trangThai IN ('Đã thanh toán','Đã cọc','Đã đặt')
│   │       AND ngayChoi <= hôm nay AND (ngayChoi < hôm nay OR gioBatDau <= now - 15phút)
│   │
│   ├─◇ (C2a) [Có booking] → FOR EACH:
│   │   ├─ cancelBookingWithReason(pool, booking, 'AUTO_NOSHOW')
│   │   └─ Ghi chú: "Hệ thống tự động hủy do quá 15 phút..."
│   │
│   ├─(C3) Cron job autoCancelPastBookings() chạy lúc 00:05
│   │
│   ├─(C4) Tìm booking auto quá hạn:
│   │       SELECT * FROM bookings WHERE isAutoBooking = TRUE
│   │       AND ngayChoi < hôm nay AND trangThai NOT IN ('Đã hủy','Hoàn thành','Đang sử dụng')
│   │
│   ├─◇ (C4a) [Có booking] → FOR EACH:
│   │   └─ cancelBookingWithReason(pool, booking, 'AUTO_BOOKING_EXPIRED')
│   │
│   └─→ ◎ Kết thúc
```

---

## §14. Activity Diagram: Check-in / Check-out (Admin Operation)

### Mô tả

Admin quản lý trạng thái đơn tại quầy: quét QR check-in → khách vào chơi → check-out khi xong.

### Swimlanes

- **Admin**: Nhân viên quầy
- **Frontend**: Admin React SPA
- **Backend**: API Server
- **Database**: PostgreSQL
- **Scheduler**: node-cron (dự phòng)

### Các bước thực hiện

```
● Bắt đầu
│
├──[Swimlane: Admin]──
│   │
│   ├─(1) [2 cách check-in]
│   │
│   ├──(1a) Quét mã QR từ điện thoại khách:
│   │        - Mở trang /admin/scanner
│   │        - Dùng camera quét QR → lấy booking ID
│   │
│   ├──(1b) Tìm thủ công:
│   │        - Vào /admin/bookings
│   │        - Lọc theo ngày hôm nay
│   │        - Tìm theo tên khách/số điện thoại
│   │
│   ├─(2) Xác nhận thông tin đơn:
│   │       - Tên khách, Sân, Khung giờ, Trạng thái
│   │
│   ├─(3) Click "Check-in"
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(4) POST /api/bookings/:id/checkin
│   │       Kiểm tra: authenticate + role = Admin
│   │
│   ├─(5) SELECT * FROM bookings WHERE id = :id
│   │
│   ├─◇ (5a) [Không tìm thấy] → 404
│   │
│   ├─◇ (5b) [trangThai NOT IN ('Đã thanh toán','Đã đặt','Đã cọc')]
│   │   └─→ 400 "Chỉ check-in đơn ở trạng thái hợp lệ"
│   │
│   ├─◇ (5c) [trangThai hợp lệ]
│   │   │
│   │   ├─(6) UPDATE bookings SET trangThai = 'Đang sử dụng', updated_at = NOW()
│   │   │
│   │   ├─(7) INSERT INTO notifications (nguoiDungId, tieuDe='Check-in thành công',
│   │   │         noiDung='Đơn #X đã được check-in. Chúc bạn chơi vui vẻ!',
│   │   │         loaiThongBao='auto_checkin', maDonDat=bookingId)
│   │   │
│   │   └─(8) Response: { message: "Check-in thành công" }
│   │
│   └──[Trả về Frontend]──
│
├──[Swimlane: Frontend]──
│   │
│   ├─(9) Toast "Check-in thành công"
│   │
│   ├─(10) Cập nhật UI: trạng thái → "Đang sử dụng" (màu xanh)
│   │
│   └──[Đợi khách chơi xong]──
│
│         ─── Sau khi hết giờ chơi ───
│
├──[Swimlane: Admin]──
│   │
│   ├─(11) Click "Check-out"
│   │
│   └──[Backend xử lý]──
│       │
│       ├─(12) POST /api/bookings/:id/checkout
│       │        Kiểm tra: authenticate + role = Admin
│       │
│       ├─(13) SELECT * FROM bookings WHERE id = :id
│       │
│       ├─◇ (13a) [trangThai != 'Đang sử dụng'] → 400 "Chỉ check-out đơn đang sử dụng"
│       │
│       ├─◇ (13b) [trangThai = 'Đang sử dụng']
│       │   │
│       │   ├─(14) UPDATE bookings SET trangThai = 'Hoàn thành', updated_at = NOW()
│       │   │
│       │   ├─(15) INSERT notification (loại='auto_checkout', nội dung='Đơn #X đã hoàn thành. Cảm ơn!')
│       │   │
│       │   └─(16) Response: { message: "Check-out thành công" }
│       │
│       └──→ ◎ Kết thúc
│
├──[Swimlane: Scheduler]── (dự phòng cho check-out)
│   │
│   ├─(D1) Cron job mỗi phút: handleBookingStatus()
│   │
│   ├─(D2) SELECT * FROM bookings WHERE trangThai = 'Đang sử dụng'
│   │       AND (ngayChoi < hôm nay OR (ngayChoi = hôm nay AND gioKetThuc <= now))
│   │
│   ├─◇ (D2a) [Có booking hết giờ] → FOR EACH:
│   │   ├─ UPDATE bookings SET trangThai = 'Hoàn thành'
│   │   └─ INSERT notification (loại='auto_checkout')
│   │
│   └─→ ◎ Kết thúc (Auto check-out)
│
└──◎ Kết thúc
```

---

## §15. Activity Diagram: VIP Auto-Booking 30 ngày

### Mô tả

Khách VIP đặt sân 1 lần → hệ thống tự động tạo booking cho tất cả các ngày cùng thứ trong 30 ngày tiếp theo.

### Swimlanes

- **VIP User**: Khách hàng VIP
- **Frontend**: React SPA
- **Backend**: Express API
- **Database**: PostgreSQL
- **Scheduler**: node-cron (legacy)

### Các bước thực hiện

```
● Bắt đầu (User đã được Admin bật VIP)
│
├──[Swimlane: VIP User]──
│   │
│   ├─(1) Vào trang đặt sân như bình thường
│   │
│   ├─(2) Chọn sân, ngày bắt đầu, khung giờ, dịch vụ
│   │
│   ├─(3) Bật toggle "Tự động đặt 30 ngày" (isAutoBooking = true)
│   │
│   ├─(4) Tùy chọn: "Áp dụng dịch vụ cho tất cả buổi" (repeatServices)
│   │
│   ├─(5) Click "Đặt sân"
│   │
│   └──[Chuyển sang Backend]──
│
├──[Swimlane: Backend]──
│   │
│   ├─(6) BEGIN TRANSACTION
│   │
│   ├─(7) Kiểm tra user.isVIP === true
│   │     IF false → ROLLBACK → 403 "Chỉ tài khoản VIP mới được bật tự động đặt sân"
│   │
│   ├─(8) Tính danh sách ngày trong 30 ngày (cùng thứ với startDate):
│   │       bookingDates = buildWeeklyDates(startDate) → mảng 4-5 ngày
│   │       (từ startDate, mỗi 7 ngày, trong 30 ngày)
│   │
│   ├─(9) Kiểm tra xung đột CHO TOÀN BỘ 30 NGÀY:
│   │       SELECT ... FROM bookings b JOIN timeslots t
│   │       WHERE b.sanId = sanId AND b.ngayChoi = ANY(bookingDates)
│   │       AND b.khungGioId = ANY(slotIds)
│   │       AND b.trangThai NOT IN ('Đã hủy')
│   │       FOR UPDATE
│   │
│   ├─◇ (9a) [Có xung đột] → ROLLBACK → 409
│   │       "Khung giờ X ngày Y đã có người đặt. Vui lòng chọn giờ khác"
│   │
│   ├─◇ (9b) [Không xung đột] → Tiếp tục
│   │
│   ├─(10) Tính giá: courtPricePerSession + servicesPricePerSession
│   │
│   ├─(11) Áp dụng mã giảm giá (nếu có)
│   │
│   ├─(12) INSERT INTO auto_booking_series:
│   │        (nguoiDungId, sanId, khungGioIds, startDate, endDate,
│   │         repeatServices, servicePolicy, totalAmount, trangThai='Active')
│   │        → Lấy autoBookingSeriesId
│   │
│   ├─(13) FOR EACH date IN bookingDates:
│   │       FOR EACH slotId IN slotIds:
│   │         INSERT INTO bookings (
│   │           nguoiDungId, sanId, khungGioId, ngayChoi,
│   │           tongTien, tienDaCoc (tổng * 10%),
│   │           giaGoc, tienGiam,
│   │           trangThai = 'Đã cọc',
│   │           isAutoBooking = TRUE,
│   │           autoBookingSeriesId = seriesId
│   │         )
│   │         INSERT INTO payments (donDatId, soTien, loaiThanhToan='VIP Auto 30 ngày')
│   │         [Date đầu + có dịch vụ] → INSERT booking_services
│   │         [Date sau + repeatServices = false] → KHÔNG thêm dịch vụ
│   │
│   ├─(14) Trừ kho dịch vụ
│   │
│   ├─(15) INSERT notification (loại='vip_auto_success'):
│   │        "Đã khóa X buổi trong 30 ngày cho lịch VIP của bạn.
│   │         Tổng thanh toán: XXXđ."
│   │
│   ├─(16) COMMIT TRANSACTION
│   │
│   └─(17) Response 201: { bookingIds, totalPrice, autoBookingSeriesId, bookingDates }
│
├──[Swimlane: Frontend]──
│   │
│   ├─(18) Hiển thị toast thành công
│   │
│   ├─(19) Hiển thị danh sách các ngày đã được đặt tự động
│   │
│   └──→ ◎ Kết thúc (VIP Auto-Booking thành công)
│
│   ═══════════ LEGACY SCHEDULER (cho VIP cũ) ═══════════
│
├──[Swimlane: Scheduler]──
│   │
│   ├─(L1) Cron job: 00:01 thứ 2 hàng tuần
│   │       processVipAutoBooking(force=false)
│   │
│   ├─(L2) SELECT DISTINCT VIP users có isAutoBooking=TRUE
│   │       nhưng chưa có autoBookingSeriesId (legacy)
│   │
│   ├─(L3) FOR EACH VIP:
│   │       - Lấy ngày đặt cuối cùng → tính ngày tiếp theo (+7 ngày)
│   │       - Kiểm tra xung đột
│   │
│   │       ◇ [Slot trống] → INSERT booking mới (trangThai='Đã cọc')
│   │       │                 → INSERT notification (vip_auto_success)
│   │       │
│   │       ◇ [Slot đã có người đặt] → UPDATE bookings SET isAutoBooking=FALSE
│   │                                   → INSERT notification (vip_auto_conflict)
│   │
│   └─→ ◎ Kết thúc
│
│   ═══════════ DAILY CLEANUP ═══════════
│
├──[Swimlane: Scheduler]──
│   │
│   ├─(C1) Cron job: 00:05 mỗi ngày
│   │       autoCancelPastBookings()
│   │
│   ├─(C2) SELECT * FROM bookings WHERE isAutoBooking=TRUE
│   │       AND ngayChoi < hôm nay
│   │       AND trangThai NOT IN ('Đã hủy','Hoàn thành','Đang sử dụng')
│   │
│   ├─(C3) FOR EACH: cancelBookingWithReason(pool, booking, 'AUTO_BOOKING_EXPIRED')
│   │       - Hoàn stock dịch vụ
│   │       - Hoàn discount usage
│   │
│   └─→ ◎ Kết thúc
│
└──◎ Kết thúc
```

---

## §16. Activity Diagram: Scheduler Cron Jobs (Tổng hợp 3 job)

### Mô tả

3 cron job chạy tự động trong cùng process Express.js để quản lý trạng thái booking.

### Danh sách Cron Jobs

| # | Cron | Tần suất | Hàm | Chức năng |
|---|------|----------|-----|-----------|
| J1 | `* * * * *` | Mỗi phút | `handleBookingStatus()` | Auto check-out + hủy no-show + hủy quá hạn thanh toán |
| J2 | `5 0 * * *` | 00:05 mỗi ngày | `autoCancelPastBookings()` | Hủy auto-booking VIP quá ngày chưa thanh toán |
| J3 | `1 0 * * 1` | 00:01 thứ 2 | `processVipAutoBooking()` | Tạo booking mới cho VIP legacy mỗi tuần |

### J1: handleBookingStatus (mỗi phút)

```
● Trigger: Cron ["* * * * *"]
│
├─(1) Lấy now = new Date(), today = formatDateLocal(now), currentTime = "HH:MM"
│
├─═══ Pha 1: Auto Check-out ═══
│   │
│   ├─(2) SELECT b.* FROM bookings b JOIN timeslots t ON b.khungGioId = t.id
│   │      WHERE b.trangThai = 'Đang sử dụng'
│   │      AND (b.ngayChoi < today OR (b.ngayChoi = today AND t.gioKetThuc <= currentTime))
│   │
│   ├─◇ (2a) [Có booking] → FOR EACH:
│   │   ├─ UPDATE bookings SET trangThai = 'Hoàn thành', updated_at = NOW()
│   │   └─ INSERT notification (loại='auto_checkout')
│   │
│   └─◇ (2b) [Không có] → qua Pha 2
│
├─═══ Pha 2: Hủy quá hạn thanh toán ═══
│   │
│   ├─(3) SELECT b.* FROM bookings b JOIN payments p ON p.donDatId = b.id
│   │      WHERE b.trangThai = 'Đã cọc'
│   │      AND p.trangThai IN ('Chờ thanh toán', 'Chờ xác nhận')
│   │      AND p.ngayGiaoDich <= NOW() - INTERVAL '15 minutes'
│   │
│   ├─◇ (3a) [Có booking] → FOR EACH:
│   │   └─ cancelBookingWithReason(client, booking, 'PAYMENT_TIMEOUT')
│   │       - Hoàn stock dịch vụ
│   │       - Hoàn discount usage
│   │
│   └─◇ (3b) [Không có] → qua Pha 3
│
├─═══ Pha 3: Hủy No-Show ═══
│   │
│   ├─(4) noShowCutoff = now - 15 phút
│   │
│   ├─(5) SELECT b.* FROM bookings b JOIN timeslots t ON b.khungGioId = t.id
│   │      WHERE b.trangThai IN ('Đã thanh toán', 'Đã đặt', 'Đã cọc')
│   │      AND (b.ngayChoi < noShowCutoffDate
│   │           OR (b.ngayChoi = noShowCutoffDate AND t.gioBatDau <= noShowCutoffTime))
│   │
│   ├─◇ (5a) [Có booking] → FOR EACH:
│   │   └─ cancelBookingWithReason(client, booking, 'AUTO_NOSHOW')
│   │       - UPDATE ghiChu = "Hệ thống tự động hủy do quá 15 phút..."
│   │       - Hoàn stock dịch vụ
│   │       - Hoàn discount usage
│   │
│   └─◇ (5b) [Không có] → Kết thúc
│
└─→ ◎ Kết thúc J1
```

### J2: autoCancelPastBookings (00:05 mỗi ngày)

```
● Trigger: Cron ["5 0 * * *"]
│
├─(1) Lấy today = formatDateLocal(new Date())
│
├─(2) SELECT * FROM bookings
│      WHERE isAutoBooking = TRUE
│      AND ngayChoi < today
│      AND trangThai NOT IN ('Đã hủy', 'Hoàn thành', 'Đang sử dụng')
│      ORDER BY ngayChoi ASC
│
├─◇ (2a) [Có booking] → FOR EACH:
│   └─ cancelBookingWithReason(client, booking, 'AUTO_BOOKING_EXPIRED')
│       - UPDATE ghiChu = "Hệ thống tự động hủy do quá ngày chơi..."
│       - Hoàn stock dịch vụ
│       - Hoàn discount usage
│
├─◇ (2b) [Không có] → Kết thúc
│
└─→ ◎ Kết thúc J2
```

### J3: processVipAutoBooking (00:01 thứ 2)

```
● Trigger: Cron ["1 0 * * 1"] hoặc force=true (admin trigger)
│
├─(1) IF today.getDay() !== 1 AND force=false → return (chỉ chạy thứ 2)
│
├─(2) SELECT DISTINCT ON (b.nguoiDungId, b.sanId, b.khungGioId)
│      b.nguoiDungId, b.sanId, b.khungGioId, b.ngayChoi as lastBookingDate
│      FROM bookings b JOIN users u ON b.nguoiDungId = u.id
│      WHERE u.isVIP = TRUE AND b.isAutoBooking = TRUE
│      AND b.autoBookingSeriesId IS NULL  ← chỉ xử lý legacy (chưa có series)
│      AND b.trangThai NOT IN ('Đã hủy')
│
├─◇ (2a) [Có VIP] → FOR EACH VIP:
│   │
│   ├─(3) Tính nextDate = lastBookingDate + 7 ngày
│   │      Nhảy qua các ngày đã qua (nextDate <= today → nextDate += 7)
│   │
│   ├─(4) Kiểm tra đã có booking cho ngày đó chưa
│   │      IF có → continue (bỏ qua)
│   │
│   ├─(5) Kiểm tra xung đột với người khác
│   │
│   ├─◇ (5a) [Có xung đột]
│   │   ├─ UPDATE bookings SET isAutoBooking = FALSE (tắt auto cho VIP này)
│   │   └─ INSERT notification (loại='vip_auto_conflict')
│   │       "Khung giờ tự động cho ngày X đã có người đặt trước.
│   │        Tính năng tự động đặt đã bị tắt."
│   │
│   ├─◇ (5b) [Không xung đột]
│   │   ├─ SELECT mucGia FROM timeslots WHERE id = slotId
│   │   ├─ INSERT INTO bookings (nguoiDungId, sanId, khungGioId, ngayChoi,
│   │   │     tongTien=slotPrice, tienDaCoc=slotPrice*10%, trangThai='Đã cọc',
│   │   │     isAutoBooking=TRUE)
│   │   └─ INSERT notification (loại='vip_auto_success')
│   │       "Đã tự động đặt lịch cho ngày X. Vui lòng thanh toán trước ngày chơi."
│   │
│   └─→ Next VIP
│
└─→ ◎ Kết thúc J3
```

---

## §17. Activity Diagram: Quản lý sân (CRUD Court — Admin)

### Mô tả

Admin thực hiện các thao tác CRUD trên sân Pickleball: Tạo mới, Sửa, Xóa mềm (ẩn), Quản lý ảnh, Quản lý khung giờ.

### Các bước thực hiện

```
● Bắt đầu (Admin đã đăng nhập)
│
├──[A. TẠO SÂN MỚI]──
│   │
│   ├─(A1) Vào /admin/courts → Click "Thêm sân"
│   │
│   ├─(A2) Nhập form:
│   │       - Tên sân (bắt buộc, unique)
│   │       - Mô tả
│   │       - Ảnh đại diện
│   │       - Trạng thái (mặc định "Sẵn sàng")
│   │
│   ├─(A3) Click "Tạo"
│   │
│   ├──[Backend]──
│   │   ├─ POST /api/courts { tenSan, moTa, hinhAnh, trangThai }
│   │   ├─◇ tenSan rỗng? → 400
│   │   ├─◇ SELECT id FROM courts WHERE tenSan = trimmedName → có? → 400 "Tên sân đã tồn tại"
│   │   ├─ INSERT INTO courts → RETURNING *
│   │   └─ Response 201: { data: newCourt }
│   │
│   └─→ Toast "Tạo sân thành công" → Refresh danh sách
│
├──[B. SỬA SÂN]──
│   │
│   ├─(B1) Click "Sửa" trên sân trong danh sách
│   │
│   ├─(B2) Sửa thông tin (chỉ field được thay đổi mới gửi lên)
│   │
│   ├──[Backend]──
│   │   ├─ PUT /api/courts/:id { tenSan?, moTa?, hinhAnh?, trangThai? }
│   │   ├─ Nếu đổi tên: kiểm tra trùng với sân khác
│   │   ├─ UPDATE courts SET ... COALESCE($1, field) ... RETURNING *
│   │   └─ Response: { data: updatedCourt }
│   │
│   └─→ Toast "Cập nhật thành công"
│
├──[C. XÓA MỀM SÂN (Ẩn)]──
│   │
│   ├─(C1) Click "Xóa" trên sân → Modal xác nhận
│   │
│   ├─(C2) Click "Xác nhận xóa"
│   │
│   ├──[Backend]──
│   │   ├─ DELETE /api/courts/:id
│   │   ├─ SELECT DISTINCT nguoiDungId FROM bookings
│   │   │   WHERE sanId = :id AND ngayChoi >= CURRENT_DATE
│   │   │   AND trangThai NOT IN ('Đã hủy')
│   │   ├─ UPDATE courts SET trangThai = 'Ẩn', updated_at = NOW()
│   │   ├─ FOR EACH affected user:
│   │   │   └─ INSERT notification: "Sân bạn đã đặt lịch đã tạm ngừng hoạt động. LH admin."
│   │   └─ Response: { message: "Đã ẩn sân thành công" }
│   │
│   └─→ Toast "Đã ẩn sân thành công" → Refresh
│
├──[D. QUẢN LÝ ẢNH SÂN]──
│   │
│   ├─(D1) Trong form sửa sân → Tab "Ảnh sân"
│   │
│   ├─(D2) Upload ảnh mới:
│   │       POST /api/upload/court-images { sanId, files[] }
│   │       (requireAdmin + authenticate)
│   │       - Ảnh đầu tiên → isMain = TRUE
│   │       - Các ảnh sau → isMain = FALSE
│   │
│   ├─(D3) Đặt ảnh chính:
│   │       PUT /api/courts/:courtId/images/:imageId/main
│   │       - UPDATE tất cả ảnh của sân → isMain = FALSE
│   │       - UPDATE ảnh được chọn → isMain = TRUE
│   │
│   ├─(D4) Xóa ảnh:
│   │       DELETE /api/courts/:courtId/images/:imageId
│   │
│   └──→ ◎ Kết thúc
│
├──[E. QUẢN LÝ KHUNG GIỜ]──
│   │
│   ├─(E1) Tạo khung giờ mới:
│   │       POST /api/courts/:id/timeslots { gioBatDau, gioKetThuc, mucGia }
│   │       ◇ Validate: gioKetThuc > gioBatDau? mucGia > 0?
│   │       ◇ Kiểm tra trùng: gioBatDau < old.gioKetThuc AND gioKetThuc > old.gioBatDau?
│   │       → INSERT INTO timeslots
│   │
│   ├─(E2) Sửa khung giờ:
│   │       PUT /api/courts/:courtId/timeslots/:id { gioBatDau?, gioKetThuc?, mucGia? }
│   │
│   ├─(E3) Xóa khung giờ:
│   │       DELETE /api/courts/:courtId/timeslots/:id
│   │       ◇ Kiểm tra booking tương lai:
│   │         SELECT id FROM bookings WHERE khungGioId = :id
│   │         AND ngayChoi >= CURRENT_DATE AND trangThai NOT IN ('Đã hủy')
│   │         → Nếu có → 400 "Không thể xóa khung giờ đang có đơn đặt trong tương lai"
│   │       → DELETE FROM timeslots WHERE id = :id AND sanId = :courtId
│   │
│   └──→ ◎ Kết thúc
│
└──◎ Kết thúc
```

---

## §18. Activity Diagram: Quản lý mã giảm giá (Discount Management)

### Mô tả

Admin tạo/sửa/xóa mã giảm giá. User xem mã khả dụng, validate mã khi đặt sân, và "săn" mã về kho cá nhân.

### A. Admin tạo mã giảm giá

```
● Bắt đầu (Admin đã đăng nhập)
│
├──[Swimlane: Admin]──
│   │
│   ├─(1) Vào /admin/discounts → Click "Tạo mã"
│   │
│   ├─(2) Nhập form:
│   │       - Mã code (bắt buộc, unique) — vd: "SUMMER2026"
│   │       - Nội dung hiển thị
│   │       - Mô tả chi tiết
│   │       - Loại giảm: percentage (%) hoặc fixed (VNĐ)
│   │       - Mức giảm (vd: 20% hoặc 50000đ)
│   │       - Giảm tối đa (nếu là %)
│   │       - Ngày bắt đầu / Ngày kết thúc
│   │       - Số lượng phát hành (0 = không giới hạn)
│   │       - Giới hạn sử dụng/user (mặc định 1)
│   │       - Điều kiện (JSON):
│   │         + min_order_value: giá trị đơn tối thiểu
│   │         + applicable_court_ids: chỉ áp dụng cho sân cụ thể
│   │         + target_audience: all / new_user / vip
│   │
│   ├─(3) Click "Tạo"
│   │
│   └──[Backend]──
│       │
│       ├─ POST /api/discounts { code, mucGiamGia, ... }
│       │
│       ├─◇ Validate:
│       │   - code và mucGiamGia bắt buộc?
│       │   - mucGiamGia >= 0?
│       │   - Nếu percentage: mucGiamGia <= 100?
│       │   - soLuongBanDau >= 0?
│       │
│       ├─ INSERT INTO discounts (...)
│       │
│       ├─◇ [Lỗi unique code 23505] → 409 "Mã giảm giá đã tồn tại"
│       │
│       └─ Response 201: { data: newDiscount }
│
└──→ ◎ Kết thúc
```

### B. User dùng mã giảm giá khi đặt sân

```
● Bắt đầu (User đang đặt sân)
│
├──(B1) Trong form đặt sân → Nhập mã giảm giá
│
├──(B2) POST /api/discounts/validate { code, totalAmount, courtId }
│       │
│       ├─(B3) SELECT * FROM discounts WHERE code = :code
│       │       AND (nguoiDungId IS NULL OR nguoiDungId = userId)
│       │       ORDER BY nguoiDungId DESC LIMIT 1
│       │
│       ├─◇ [Không tìm thấy] → 400 "Mã giảm giá không tồn tại"
│       ├─◇ [trangThai != 'active'] → 400 "Mã đang bị vô hiệu hóa"
│       ├─◇ [nguoiDungId != NULL AND nguoiDungId != userId] → 400 "Không thuộc sở hữu của bạn"
│       ├─◇ [ngayBatDau > now] → 400 "Chưa đến ngày hiệu lực"
│       ├─◇ [ngayKetThuc < now] → 400 "Đã hết hạn"
│       ├─◇ [soLuongDaDung >= soLuongBanDau > 0] → 400 "Đã hết số lượng"
│       ├─◇ [user đã dùng >= usage_limit_per_user] → 400 "Bạn đã sử dụng mã này rồi"
│       ├─◇ [totalAmount < min_order_value] → 400 "Đơn tối thiểu phải từ Xđ"
│       ├─◇ [courtId không trong applicable_court_ids] → 400 "Không áp dụng cho sân này"
│       ├─◇ [target_audience='new_user' AND user đã có đơn] → 400 "Chỉ dành cho khách mới"
│       ├─◇ [target_audience='vip' AND user không VIP] → 400 "Chỉ dành cho VIP"
│       │
│       ├─(B4) Tính discountAmount:
│       │       IF percentage: Math.round(totalAmount * mucGiamGia / 100)
│       │         IF giamToiDa > 0: min(discountAmount, giamToiDa)
│       │       IF fixed: min(mucGiamGia, totalAmount)
│       │
│       └─(B5) Response: { data: { ...discount, discountAmount, conditions } }
│
├──(B6) Frontend cập nhật hiển thị:
│        - Giá gốc: XXXđ (gạch ngang)
│        - Giảm: -YYYđ
│        - Tổng: ZZZđ
│
└──→ Tiếp tục luồng đặt sân (§12)
```

### C. User "săn" mã về kho cá nhân (Claim)

```
● Bắt đầu (User thấy mã công khai)
│
├──(C1) Vào trang Mã giảm giá (/my-vouchers)
│
├──(C2) Thấy danh sách mã chung đang khả dụng
│       (GET /api/discounts/my → lọc theo điều kiện)
│
├──(C3) Click "Săn mã" (Claim)
│
├──(C4) POST /api/discounts/validate { code, totalAmount, courtId, isClaiming: true }
│       │
│       ├─ Kiểm tra tất cả validate (như B3)
│       ├─ Kiểm tra chưa sở hữu mã này (check notifications)
│       ├─ Tạo bản sao mã riêng cho user:
│       │   INSERT INTO discounts (code, noiDung, ..., soLuongBanDau=1, nguoiDungId=userId, is_hidden=FALSE)
│       ├─ INSERT notification (promotion): "Săn mã thành công! Mã X đã vào kho của bạn."
│       └─ Response: { data: { ...discount, discountAmount } }
│
├──(C5) Toast "Săn mã thành công!"
│
└──→ ◎ Kết thúc
```

### D. Admin xóa mã giảm giá

```
● Admin click "Xóa" trên 1 mã discount
│
├──(D1) DELETE /api/discounts/:id
│       │
│       ├─ SELECT code FROM discounts WHERE id = :id
│       ├─ SELECT id FROM bookings WHERE maGiamGia = code
│       │   AND trangThai != 'Đã hủy' LIMIT 1
│       │
│       ├─◇ [Đang có đơn sử dụng] → 400
│       │   "Không thể xóa mã giảm giá đang được sử dụng bởi đơn đặt chưa hủy."
│       │
│       └─◇ [Không có đơn nào dùng] → DELETE FROM discounts WHERE id = :id
│
└──→ ◎ Kết thúc
```

---

## §19. Activity Diagram: Xác thực OTP (OTP Verification Flow)

### Mô tả

Luồng gửi và xác thực OTP cho 2 mục đích: (A) Đăng ký tài khoản, (B) Reset mật khẩu. OTP 6 số, hiệu lực 10 phút, lưu trong memory Map.

### A. OTP cho Đăng ký

```
● Bắt đầu (User đã submit form đăng ký)
│
├──[Swimlane: Backend]──
│   │
│   ├─(1) Nhận POST /api/auth/register
│   │
│   ├─(2) Validate form → tạo OTP
│   │
│   ├─(3) Lưu otpStore:
│   │       key = "register:{email}"
│   │       value = { otp, password, full_name, phone_number, expires: Date.now() + 10*60*1000 }
│   │
│   ├─(4) [NODE_ENV !== 'production'] console.log OTP
│   │
│   └─(5) Response: "Mã OTP đã được gửi"
│
├──[Swimlane: User]──
│   │
│   ├─(6) Đọc OTP từ console server (dev)
│   │     HOẶC từ email (production — chưa implement)
│   │
│   ├─(7) Nhập OTP 6 số vào form xác thực
│   │
│   ├─(8) Click "Verify"
│   │
│   └──[Backend]──
│       │
│       ├─(9) POST /api/auth/verify-register { email, otp, password, full_name }
│       │
│       ├─(10) stored = otpStore.get("register:{email}")
│       │
│       ├─◇ (10a) [!stored OR stored.otp !== otp OR Date.now() > stored.expires]
│       │   └─→ 400 "Mã OTP không chính xác hoặc đã hết hạn"
│       │
│       ├─◇ (10b) [OTP hợp lệ]
│       │   ├─ Hash password → INSERT user → Tạo JWT → Tạo mã chào mừng
│       │   ├─ otpStore.delete("register:{email}")
│       │   └─ Response 200: { token, user }
│       │
│       └──→ ◎ Kết thúc (Đăng ký thành công)
│
├──[Swimlane: User]── (nếu OTP hết hạn hoặc sai)
│   │
│   ├─(11) Click "Gửi lại OTP"
│   │
│   ├─(12) POST /api/auth/resend-otp { email, type: 'register' }
│   │       │
│   │       ├─ Kiểm tra email chưa đăng ký
│   │       ├─ stored = otpStore.get("register:{email}")
│   │       ├─◇ [!stored] → 400 "Không tìm thấy yêu cầu đăng ký. Vui lòng đăng ký lại."
│   │       ├─ Tạo OTP mới → gán lại expires = now + 10 phút
│   │       └─ Response: "Mã OTP mới đã được gửi"
│   │
│   └──→ Quay lại (7)
│
└──◎ Kết thúc
```

### B. OTP cho Reset Mật khẩu

```
● Bắt đầu (User quên mật khẩu)
│
├──[Swimlane: User]──
│   │
│   ├─(B1) Vào /login → Click "Quên mật khẩu"
│   │
│   ├─(B2) Nhập email đã đăng ký
│   │
│   ├─(B3) Click "Gửi mã OTP"
│   │
│   └──[Backend]──
│       │
│       ├─ POST /api/auth/forgot-password { email }
│       ├─ SELECT id FROM users WHERE email = :email
│       ├─◇ [Không tồn tại] → 404 "Email chưa được đăng ký"
│       ├─ Tạo OTP → otpStore.set("reset:{email}", { otp, expires: now + 10phút })
│       └─ Response: "Mã OTP đã được gửi"
│
├──[Swimlane: User]──
│   │
│   ├─(B4) Nhập OTP + mật khẩu mới
│   │
│   ├─(B5) Click "Đặt lại mật khẩu"
│   │
│   └──[Backend]──
│       │
│       ├─ POST /api/auth/reset-password { email, otp, new_password }
│       ├─ stored = otpStore.get("reset:{email}")
│       ├─◇ [OTP sai/hết hạn] → 400
│       ├─ bcrypt.hash(new_password, 10)
│       ├─ UPDATE users SET matKhau = hashedPw WHERE email = email
│       ├─ otpStore.delete("reset:{email}")
│       └─ Response: "Đổi mật khẩu thành công"
│
└──→ ◎ Kết thúc (User có thể đăng nhập với MK mới)
```

---

## §20. Activity Diagram: Loyalty Rewards (Tặng mã tri ân mỗi 3 đơn)

### Mô tả

Mỗi khi user đặt đủ bội số của 3 đơn (3, 6, 9, 12...), hệ thống tự động tặng 1 mã giảm giá 10% tri ân.

### Các bước thực hiện

```
● Bắt đầu (User vừa tạo đơn đặt sân thành công)
│
├──[Swimlane: Backend]── (trong transaction của POST /api/bookings)
│   │
│   ├─(1) Đếm tổng số đơn trước khi tạo đơn mới (trừ đã hủy):
│   │       SELECT COUNT(*) as count FROM bookings
│   │       WHERE nguoiDungId = userId
│   │       AND trangThai != 'Đã hủy'
│   │       AND id NOT IN (vừa tạo)
│   │       → prevTotal
│   │
│   ├─(2) newTotal = prevTotal + số đơn vừa tạo (bookingIds.length)
│   │
│   ├─(3) Tính số mốc 3:
│   │       prevRewards = Math.floor(prevTotal / 3)
│   │       newRewards = Math.floor(newTotal / 3)
│   │
│   ├─◇ (3a) [newRewards > prevRewards] → User vừa đạt mốc mới!
│   │   │
│   │   ├─(4) FOR EACH i FROM 0 TO (newRewards - prevRewards - 1):
│   │   │
│   │   │   ├─(5) Tạo mã giảm giá ngẫu nhiên:
│   │   │   │       randomStr = Math.random().toString(36).substring(2, 6).toUpperCase()
│   │   │   │       rewardCode = "LTY10-{randomStr}"
│   │   │   │
│   │   │   ├─(6) INSERT INTO discounts (
│   │   │   │       code = rewardCode,
│   │   │   │       noiDung = 'Quà tặng đặt sân',
│   │   │   │       moTa = 'Mã giảm giá 10% tri ân mỗi 3 đơn đặt sân',
│   │   │   │       loaiGiamGia = 'percentage',
│   │   │   │       mucGiamGia = 10,
│   │   │   │       ngayBatDau = NOW(),
│   │   │   │       ngayKetThuc = '2026-12-31',
│   │   │   │       soLuongBanDau = 1,
│   │   │   │       nguoiDungId = userId,
│   │   │   │       trangThai = 'Active'
│   │   │   │     )
│   │   │   │
│   │   │   └─(7) INSERT INTO notifications (
│   │   │         nguoiDungId = userId,
│   │   │         tieuDe = 'Quà tặng tri ân!',
│   │   │         noiDung = 'Bạn vừa đạt mốc {newRewards*3} đơn đặt sân!
│   │   │                    Hệ thống tặng bạn mã giảm giá 10% tri ân: {rewardCode}',
│   │   │         loaiThongBao = 'promotion'
│   │   │       )
│   │   │
│   │   └─→ Next reward
│   │
│   ├─◇ (3b) [newRewards <= prevRewards] → Không làm gì
│   │
│   └─→ Tiếp tục COMMIT transaction (trong §12)
│
├──[Swimlane: Frontend]──
│   │
│   ├─(8) [Nếu có loyalty reward] Hiển thị toast đặc biệt:
│   │       "Chúc mừng! Bạn đã đạt mốc X đơn và nhận được mã giảm giá 10%!"
│   │
│   └──→ ◎ Kết thúc
│
└──◎ Kết thúc
```

---

## Phụ lục: Tổng kết Activity Diagrams

| # | Activity Diagram | Số node chính | Swimlanes |
|---|-----------------|---------------|-----------|
| §11 | Đăng ký tài khoản | 27 steps, 8 decisions | User, Frontend, Backend, Database |
| §12 | Đặt sân | 28 steps, 10 decisions | Customer, Frontend, Backend, Database |
| §13 | Hủy đơn | 3 luồng (A/B/C), 15 decisions | Customer, Admin, Backend, Scheduler |
| §14 | Check-in/Check-out | 16 steps + scheduler | Admin, Frontend, Backend, Scheduler |
| §15 | VIP Auto-Booking | 19 steps + legacy + cleanup | VIP, Frontend, Backend, Scheduler |
| §16 | Scheduler Cron Jobs | 3 jobs (J1:3 pha, J2, J3) | Scheduler, Database |
| §17 | Quản lý sân (CRUD) | 5 phần (A-E) | Admin, Backend, Database |
| §18 | Quản lý mã giảm giá | 4 phần (A-D) | Admin, User, Backend |
| §19 | Xác thực OTP | 2 luồng (Register + Reset) | User, Backend |
| §20 | Loyalty Rewards | 8 steps, 1 decision | Backend, Frontend |

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
