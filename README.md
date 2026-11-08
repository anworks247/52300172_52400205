# Web quản lý Task

## Chức năng và dữ liệu
Ứng dụng quản lý task có đăng ký/đăng nhập JWT, CRUD task, lọc theo trạng thái và dashboard thống kê. Mỗi user chỉ xem task của mình. Schema PostgreSQL nằm ở schema.sql.

## API
Các API task cần Bearer token, trừ đăng ký/đăng nhập và health check.

| Method | Endpoint | Mô tả |
|---|---|---|
| GET | /api/health | Kiểm tra API và kết nối PostgreSQL |
| POST | /api/auth/register | Tạo tài khoản |
| POST | /api/auth/login | Đăng nhập, trả token |
| GET | /api/tasks?status= | Danh sách task |
| GET | /api/tasks/:id | Chi tiết task |
| POST | /api/tasks | Tạo task |
| PUT | /api/tasks/:id | Cập nhật task |
| DELETE | /api/tasks/:id | Xóa task |
| GET | /api/stats | Thống kê dashboard |

## Yêu cầu và chạy local
Cài Node.js LTS tương thích môi trường deploy và PostgreSQL. Tạo database taskdb, chạy schema.sql, copy .env.example thành .env rồi điền credentials riêng:

    npm ci
    npm start

Mặc định API và giao diện Node tĩnh chạy tại http://localhost:3000. Endpoint health trả HTTP 200 khi database hoạt động và 503 khi database chưa kết nối.

Tùy chọn giao diện PHP local: chạy API bằng npm start, sau đó chạy PHP built-in server với document root frontend và đặt API_BASE trỏ tới API. Production VPS trong TV2 package hiện phục vụ giao diện public/index.html qua Node/Nginx.

