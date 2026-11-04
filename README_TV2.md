# Triển khai ứng dụng web (Deployment of Web Applications)

## Hướng dẫn demo và cài đặt dự án

README này là tài liệu kỹ thuật dành cho người chấm bài. Tài liệu nêu các yêu cầu hệ thống và cung cấp các lệnh để cài đặt phụ thuộc, chạy bản demo và kiểm thử các chức năng chính. Nhóm phải thực hiện các hướng dẫn này trên một máy tính sạch trước khi nộp bài và hoàn thành bản ghi xác minh ở Mục 9. Không bài kiểm thử nào được đánh dấu là đạt nếu chưa có kết quả quan sát thực tế.

## 1. Yêu cầu hệ thống

**Môi trường cục bộ:** Windows 10/11 64-bit (các lệnh bên dưới dùng PowerShell; có thể dùng shell tương đương trên macOS/Linux), Git, Node.js LTS tương thích với trường engines trong package.json, npm, trình duyệt phiên bản hiện hành và kết nối Internet. Ghi lại phiên bản bằng các lệnh node --version, npm --version và git --version. Không yêu cầu Flutter. Cần PostgreSQL để chạy cục bộ; người chấm có thể dùng bản demo đã triển khai thay thế.

**Yêu cầu dịch vụ triển khai:** quyền truy cập vào kho mã nguồn của môn học, Render Web Service và Neon PostgreSQL. Có thể dùng kho riêng tư (private) khi Render có quyền truy cập. Không yêu cầu VPS hay tên miền riêng cho bản demo onrender.com hiện tại.

## 2. Clone mã nguồn và cài đặt phụ thuộc (⚠️Dùng để tái tạo mô hình nếu Render lỗi, chạy trên localhost)

Mở PowerShell và thay URL kho mã nguồn cùng tên thư mục bằng các giá trị được cung cấp trong hồ sơ nộp bài:

    node --version
    npm --version
    git --version
    Set-Location $HOME\Documents
    git clone <REPOSITORY_URL>
    Set-Location .\<REPOSITORY_FOLDER>
    npm ci

Lệnh npm ci cài đặt đúng các phiên bản phụ thuộc được ghi trong package-lock.json. Chạy lệnh từ thư mục gốc của kho mã nguồn. Nếu cài đặt thất bại, hãy ghi lại đầy đủ thông báo lỗi cùng phiên bản Node/npm; không tự ý thay đổi phiên bản phụ thuộc.

## 3. Cấu hình môi trường cục bộ

Nếu kho mã nguồn có tệp .env.example:

    Copy-Item .env.example .env
    notepad .env

Đặt các giá trị mà ứng dụng yêu cầu, chẳng hạn DATABASE_URL, JWT_SECRET, CORS_ORIGINS, NODE_ENV, HOST và PORT. Đối chiếu tên chính xác với .env.example. Dùng cơ sở dữ liệu kiểm thử đã được phê duyệt; không công bố thông tin xác thực thật trong tài liệu này.

Xác nhận Git bỏ qua tệp .env mà không in ra các giá trị bí mật:

    git check-ignore -v .env
    git status --short --ignored -- .env

## 4. Chạy và kiểm thử ứng dụng cục bộ

Từ thư mục gốc của dự án trong PowerShell:

    $env:NODE_ENV = 'development'
    $env:HOST = '127.0.0.1'
    $env:PORT = '3000'
    npm start

Mở http://localhost:3000 hoặc URL/cổng được in ra trong log khởi động. Dừng máy chủ bằng Ctrl+C. Chạy npm run để xem các script có sẵn; chỉ dùng npm test nếu package.json có định nghĩa script test.

| ID | Trường hợp kiểm thử | Kết quả mong đợi | Kết quả thực tế |
|---|---|---|---|
| L-01 | Mở trang chủ | Trang tải xong, không có lỗi chặn | ✅ |
| L-02 | Gửi yêu cầu GET /api/health | HTTP 200 và trạng thái cơ sở dữ liệu khớp với hợp đồng API | ✅ |
| L-03 | Đăng ký và đăng nhập bằng tài khoản thử nghiệm | Thông tin hợp lệ được xác thực; thông tin sai bị từ chối an toàn | ✅ |
| L-04 | Tạo, xem, cập nhật và xóa một tác vụ (task) | Dữ liệu hiển thị phản ánh đúng từng thao tác | ✅ |
| L-05 | Dùng tài khoản thử nghiệm thứ hai | Không thể đọc hoặc thay đổi dữ liệu của tài khoản thứ nhất | ✅ |
| L-06 | Khởi động lại ứng dụng và đăng nhập lại | Dữ liệu PostgreSQL vẫn còn | ✅ |


## 5. Cấu hình Neon PostgreSQL

1. Mở dự án Neon của nhóm và chọn nhánh (branch) cùng cơ sở dữ liệu cho môi trường dự kiến.
2. Ghi lại vùng (region) và gói dịch vụ quan sát được vào ngày xác minh.
3. Áp dụng schema hoặc migration đã được phê duyệt và xác nhận các bảng, khóa, ràng buộc và chỉ mục như mong đợi.
4. Lấy chuỗi kết nối (URI) TLS phù hợp; dùng endpoint có pooling nếu ứng dụng được cấu hình để dùng pooling.
5. Thêm URI vào Render dưới dạng biến môi trường bí mật DATABASE_URL.
6. Kiểm tra kết nối thông qua endpoint health và một luồng nghiệp vụ của ứng dụng.

## 6. Cấu hình Render Web Service

1. Kết nối Render với kho GitHub và chọn nhánh dự kiến.
2. Tạo một Node.js Web Service.
3. Đặt npm ci làm lệnh cài đặt/build và npm start làm lệnh khởi chạy nếu phù hợp với package.json đã nộp.
4. Cấu hình các biến môi trường mà ứng dụng sử dụng, chẳng hạn NODE_ENV=production, HOST=0.0.0.0, DATABASE_URL, JWT_SECRET và CORS_ORIGINS. Render cấp một cổng cho dịch vụ; ứng dụng phải lắng nghe trên cổng đó.
5. Đặt /api/health làm đường dẫn kiểm tra sức khỏe (health check) nếu phản hồi phù hợp.
6. Triển khai và xem lại log build và log chạy.
7. Ghi lại URL, nhánh, mã SHA của commit, thời điểm triển khai và gói instance.

Render quản lý tên miền onrender.com

## 7. Xác minh triển khai, HTTPS, khởi động lại và log

    $base = 'https://https://giuakiweb-quanlytask.onrender.com'
    Invoke-WebRequest "$base/api/health" -Method Get


- Log: mở trang Logs của dịch vụ trên dashboard Render; ghi lại dấu thời gian.
- Khởi động lại: khởi động lại dịch vụ từ dashboard; ghi lại thời điểm thực hiện, thời gian để về trạng thái hoạt động bình thường (healthy) và tình trạng dữ liệu.
- Tính bền vững dữ liệu: tạo một bản ghi thử nghiệm, khởi động lại web service và xác nhận bản ghi vẫn còn trong PostgreSQL.
- Rollback: nếu một bản phát hành thất bại, ghi lại SHA bị lỗi, SHA ổn định, thao tác rollback và kết quả xác minh sau đó.

## 8. Bằng chứng và quan sát hiệu năng

| Tên tệp đề xuất | Bằng chứng |
|---|---|
| 01-main-page.png |Tổng quát về trang web và Domain |
| 02-runtime-logs.png | Log khởi động đã che thông tin bí mật |
| 03-login-page.png | Trang đăng nhập |
| 04-register.png | Trang đăng ký |
| 05-failed-register.png | Đăng ký trùng email sẽ không được tạo tài khoản |
| 06-health.png | URL health, trạng thái và phản hồi |
| 07-https.png | Trang web secure và chạy tốt |
| 08-neon-schema.png | Schema trong giao diện Neon |
| 09-restart-persistence.mp4 | video minh họa khi reload web vẫn giữ log trước |
