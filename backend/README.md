# SLIFE Backend

## Chạy local với MySQL `slife_db`

```bash
cd backend
./mvnw spring-boot:run -Dspring-boot.run.jvmArguments="-DSPRING_DATASOURCE_URL=jdbc:mysql://localhost:3306/slife_db -DSPRING_DATASOURCE_USERNAME=root -DSPRING_DATASOURCE_PASSWORD=123456"
# Windows PowerShell (khuyến nghị - tránh lỗi parse `-Dspring-boot.run.jvmArguments`)
$env:SPRING_DATASOURCE_URL="jdbc:mysql://localhost:3306/slife_db"
$env:SPRING_DATASOURCE_USERNAME="root"
$env:SPRING_DATASOURCE_PASSWORD="123456"
.\mvnw.cmd spring-boot:run
```

Backend chạy ở `http://localhost:8080`.

## Kiểm tra API admin user list

```bash
curl.exe http://localhost:8080/api/admin/users
# hoặc PowerShell
Invoke-WebRequest http://localhost:8080/api/admin/users
```

## Chạy test backend

```bash
./mvnw test
# Windows PowerShell
.\mvnw.cmd test
```

> Nếu gặp lỗi tải dependency từ Maven Central (403/network), cần kiểm tra proxy/repository mirror trong môi trường của bạn.


## Lỗi thường gặp trên PowerShell

Nếu bạn thấy Maven cố tải artifact lạ như:
`run/jvmArguments=-DSPRING_DATASOURCE_URL=jdbc/...`
thì PowerShell đã parse sai tham số `-Dspring-boot.run.jvmArguments`.

Cách ổn định nhất là **không truyền jvmArguments inline** trên PowerShell, mà set biến môi trường trước rồi chạy `mvnw.cmd` như hướng dẫn ở trên.
