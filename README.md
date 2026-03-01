# SLIFE - Hướng dẫn chạy local (MySQL + Backend + Frontend)

Tài liệu này giúp bạn chạy nhanh project trên IntelliJ với MySQL local.

## 1) Chuẩn bị

- Java 17+
- Maven 3.9+ (hoặc dùng `./mvnw`)
- Node.js 18+
- MySQL 8+

Thông tin DB local bạn đang dùng:

- Host: `localhost`
- Port: `3306`
- User: `root`
- Password: `123456`
- Database: `slife_db`

## 2) Tạo schema + seed dữ liệu

Bạn có 2 lựa chọn:

### Cách A - Dùng file SQL trong repo (đơn giản, đúng DB `slife_db`)

Chạy lần lượt:

```bash
mysql -u root -p123456 -e "CREATE DATABASE IF NOT EXISTS slife_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -u root -p123456 slife_db < SLife_Database_MySQL/01_create_schema.sql
mysql -u root -p123456 slife_db < SLife_Database_MySQL/02_seed_data.sql
```

### Cách B - Dùng Flyway khi chạy backend

Backend hiện có migration trong `backend/src/main/resources/db/migration`.
Nếu dùng cách này, đảm bảo backend trỏ đúng DB (`slife_db`) trước khi chạy.

## 3) Chạy Backend (Spring Boot)

Từ thư mục `backend`:

```bash
cd backend
./mvnw spring-boot:run -Dspring-boot.run.profiles=local -Dspring-boot.run.jvmArguments="-DSPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/slife_db -DSPRING_DATASOURCE_USERNAME=root -DSPRING_DATASOURCE_PASSWORD=123456"
# Windows PowerShell (khuyến nghị - tránh lỗi parse `-Dspring-boot.run.jvmArguments`)
$env:SPRING_DATASOURCE_URL="jdbc:mysql://localhost:3306/slife_db"
$env:SPRING_DATASOURCE_USERNAME="root"
$env:SPRING_DATASOURCE_PASSWORD="123456"
.\mvnw.cmd spring-boot:run
```

Backend mặc định chạy tại: `http://localhost:8080`

Kiểm tra nhanh endpoint admin user list:

```bash
curl.exe http://localhost:8080/api/admin/users
# hoặc PowerShell
Invoke-WebRequest http://localhost:8080/api/admin/users
```

Nếu DB có dữ liệu, bạn sẽ thấy JSON array user.

## 4) Chạy Frontend (Vite + React)

Từ thư mục `frontend`:

```bash
cd frontend
npm install
npm run dev
```

Frontend chạy tại: `http://localhost:5173`

## 5) Test nhanh chức năng “admin check profile”

1. Mở `http://localhost:5173/admin/users`
2. Trang sẽ gọi API `GET /api/admin/users` và hiển thị bảng user.
3. Bấm **Reload** để tải lại dữ liệu.

> Lưu ý: Route `/admin/users` đang được bọc `ProtectedRoute` theo role `ADMIN`. Khi test local, nếu chưa có flow login hoàn chỉnh, bạn có thể đăng nhập bằng tài khoản admin trong seed data hoặc set token hợp lệ theo flow auth của project.

## 6) Lệnh test/build tham khảo

### Frontend

```bash
cd frontend
npm test -- --run
npm run build
```

### Backend

```bash
cd backend
./mvnw test
# Windows PowerShell
.\mvnw.cmd test
```

Nếu môi trường công ty chặn Maven Central thì backend test có thể fail do không tải được dependency.


## Lỗi thường gặp trên PowerShell

Nếu bạn thấy Maven cố tải artifact lạ như:
`run/jvmArguments=-DSPRING_DATASOURCE_URL=jdbc/...`
thì PowerShell đã parse sai tham số `-Dspring-boot.run.jvmArguments`.

Cách ổn định nhất là **không truyền jvmArguments inline** trên PowerShell, mà set biến môi trường trước rồi chạy `mvnw.cmd` như hướng dẫn ở trên.
