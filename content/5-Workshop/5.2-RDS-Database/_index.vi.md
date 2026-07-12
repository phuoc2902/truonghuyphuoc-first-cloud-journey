---
title: "5.2. Cấu hình Cơ sở dữ liệu RDS PostgreSQL"
date: 2026-07-06
weight: 2
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán:** Dữ liệu cốt lõi của một hệ thống E-commerce (như thông tin Sản phẩm, Đơn hàng, Lịch sử giao dịch) yêu cầu tính toàn vẹn dữ liệu (Data Integrity) và tính nhất quán (ACID) tuyệt đối. Nếu sử dụng cơ sở dữ liệu cài đặt trực tiếp trên cùng một máy chủ VPS với ứng dụng, khi máy chủ quá tải hoặc gặp sự cố phần cứng, toàn bộ dữ liệu giao dịch có nguy cơ bị mất hoặc hỏng.

**Giải pháp:** Tách rời (Decouple) tầng dữ liệu ra khỏi tầng ứng dụng bằng cách sử dụng dịch vụ quản trị cơ sở dữ liệu quan hệ đám mây **Amazon RDS PostgreSQL**.
- **Tính khả dụng cao (High Availability):** AWS tự động sao lưu (Automated Backups) và quản lý cơ sở hạ tầng.
- **Bảo mật:** Database được đặt trong một Mạng riêng ảo (VPC) và chặn toàn bộ quyền truy cập từ Internet công cộng. Nó chỉ mở cổng kết nối duy nhất (Port 5432) cho địa chỉ IP của máy chủ ứng dụng thông qua **Security Groups**.

Dưới đây là phần **Minh chứng triển khai** cấu hình Amazon RDS PostgreSQL:

#### 2. Khởi tạo DB Instance trên Amazon RDS
1. Truy cập vào AWS Console, tìm kiếm dịch vụ **RDS** và chọn **Create database**.
2. Chọn phương thức cấu hình **Standard create**.
3. Chọn Engine type là **PostgreSQL**.
4. Chọn Templates phù hợp (Chọn **Free Tier** nếu bạn đang thực hành trên tài khoản cá nhân để tránh mất phí).
5. Điền các thông tin:
   - **DB instance identifier**: `novatech-db`
   - **Master username**: `postgres`
   - **Master password**: *Nhập mật khẩu an toàn của bạn*
6. Tại mục **Connectivity**:
   - Chọn VPC thích hợp.
   - Mục **Public access**: Chọn **No** (Chỉ cho phép truy cập nội bộ và qua Security Group chỉ định để tăng tính bảo mật).
7. Nhấn **Create database** và đợi quá trình khởi tạo hoàn tất. Khi hoàn thành, bạn sẽ thấy thông tin Endpoint & Port hiển thị:

![Chi tiết Endpoint RDS](/images/5-Workshop/rds_endpoint_details.png)

---

#### 3. Cấu hình Security Group bảo mật cho Database
Nhằm đảm bảo an toàn, chúng ta chỉ cho phép máy chủ ứng dụng Spring Boot kết nối tới RDS Database.
1. Đi tới phần cấu hình **Security Groups** của cơ sở dữ liệu vừa tạo.
2. Chọn **Inbound rules** (Quy tắc đầu vào) và thêm một quy tắc mới:
   - **Type**: `PostgreSQL` (Port 5432)
   - **Source**: Chọn địa chỉ IP Private của App Server đi qua đường truyền **AWS Site-to-Site VPN** (hoặc chọn IP tĩnh công cộng của máy chủ DigitalOcean nếu kết nối trực tiếp ngoài môi trường mạng nội bộ).

![Cấu hình Security Group cho RDS](/images/5-Workshop/rds_security_groups.png)
![Chi tiết Rule Security Group](/images/5-Workshop/novatech_db_security_group.png)

---

#### 4. Cấu hình mã nguồn Backend kết nối Database
Tại mã nguồn của dự án Spring Boot, thay vì khai báo cứng (hardcode) tài khoản/mật khẩu cơ sở dữ liệu trong file cấu hình, chúng ta sử dụng biến môi trường (Environment Variables) để bảo mật thông tin kết nối.

Dưới đây là đoạn cấu hình trong file [application.yml](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/resources/application.yml):

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
```

**Mô tả tham số môi trường cấu hình tại App Server:**
- `DB_URL`: Có định dạng `jdbc:postgresql://<RDS-Endpoint>:5432/<Database-Name>`
- `DB_USERNAME`: Tài khoản master database.
- `DB_PASSWORD`: Mật khẩu kết nối database.
