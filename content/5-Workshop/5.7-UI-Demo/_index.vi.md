---
title: "5.7. Demo Giao diện Ứng dụng Thực tế"
date: 2026-07-06
weight: 7
---

Chương này trình bày chi tiết giao diện thực tế của hệ thống **NovaTech Laptop E-Commerce** và cách các thành phần giao diện tương tác trực tiếp với hạ tầng **AWS Hybrid Cloud** (Amazon RDS, S3, CloudFront, Cognito, Bedrock) và máy chủ **DigitalOcean VPS**.

---

#### 1. Giao diện phía Khách hàng (Storefront Next.js Website)

##### A. Giao diện Trang chủ (Homepage)
Trang chủ là điểm tiếp xúc đầu tiên của khách hàng với hệ thống, được thiết kế tối ưu hóa tốc độ tải trang nhờ việc phân tách hoàn toàn Frontend tĩnh (Next.js) và Backend API.

*   **Tích hợp hạ tầng:** 
    *   Toàn bộ mã nguồn tĩnh (HTML, CSS, JS, Icon) của Next.js được lưu trữ bảo mật trên **Amazon S3 (SPA Private Origin)** và được phân phối ra toàn cầu thông qua mạng lưới Edge Location của **Amazon CloudFront CDN**.
    *   Các hình ảnh banner, logo được lưu tại S3 bucket và được cache tại CloudFront để giảm thiểu độ trễ tải trang xuống mức dưới 100ms.
*   **Luồng hoạt động:** Người dùng truy cập tên miền → Yêu cầu được phân giải qua **Route 53** → Đi qua **AWS WAF** (ngăn chặn các cuộc tấn công tự động) → CloudFront trả về giao diện tĩnh ngay tại Edge Location gần nhất.

![Giao diện Trang chủ](/images/5-Workshop/demo/homepage.jpg)

---

##### B. Trang Đăng ký & Đăng nhập (Cognito Authentication)
Hệ thống loại bỏ hoàn toàn việc lưu trữ mật khẩu trực tiếp trong cơ sở dữ liệu để đảm bảo an toàn tuyệt đối cho danh tính khách hàng.

*   **Tích hợp hạ tầng:** 
    *   Sử dụng **Amazon Cognito User Pool** làm nhà cung cấp định danh (Identity Provider - IdP).
    *   Frontend Next.js sử dụng thư viện **AWS Amplify SDK Auth** để giao tiếp trực tiếp với endpoint Cognito.
*   **Luồng hoạt động:** 
    1.  Khách hàng nhập Email và Mật khẩu → Gọi API của Cognito User Pool.
    2.  Cognito thực hiện kiểm tra chính sách bảo mật, gửi mã kích hoạt tài khoản (OTP) về Email người dùng qua **Amazon SES**.
    3.  Khi đăng nhập thành công, Cognito cấp một mã **JSON Web Token (JWT)** bao gồm *Access Token*, *Id Token* và *Refresh Token*.
    4.  Mã JWT này được lưu trữ ở LocalStorage/Cookie phía Client và được đính kèm vào tiêu đề (`Authorization: Bearer <Token>`) trong mọi request gửi về Spring Boot Backend trên DigitalOcean để xác thực quyền truy cập.

![Giao diện Đăng ký](/images/5-Workshop/demo/register.jpg)
![Giao diện Đăng nhập](/images/5-Workshop/demo/login.jpg)

---

##### C. Trang Thông tin cá nhân (User Profile)
Cho phép khách hàng xem và chỉnh sửa thông tin hồ sơ cá nhân và quản lý lịch sử đơn hàng đã đặt.

*   **Tích hợp hạ tầng:** 
    *   Thông tin hồ sơ và lịch sử mua hàng được lưu trữ trong cơ sở dữ liệu **Amazon RDS PostgreSQL** đặt trong vùng Private Subnet của AWS VPC.
*   **Luồng hoạt động:** 
    *   Trình duyệt gửi yêu cầu lấy profile → Nginx (DigitalOcean) chuyển tiếp đến Spring Boot Backend → Backend giải mã và validate token JWT nhận từ Cognito → Gửi truy vấn SQL qua đường truyền mã hóa **AWS Site-to-Site VPN** vào **RDS PostgreSQL** → Trả kết quả về cho Client.

![Thông tin cá nhân](/images/5-Workshop/demo/user_profile.jpg)

---

##### D. Trang Chi tiết sản phẩm (Product Detail)
Trang hiển thị cấu hình chi tiết của laptop (CPU, RAM, VGA, ổ cứng, dung lượng pin), giá bán và nút đặt mua hàng.

*   **Tích hợp hạ tầng:** 
    *   Thông tin văn bản (specs) của laptop được lưu trữ ở **RDS PostgreSQL**.
    *   Hình ảnh độ phân giải cao của sản phẩm được lưu tại **Amazon S3** (bucket `novatech-product-images`). 
    *   Để bảo mật hình ảnh sản phẩm không bị truy cập trái phép hoặc tải lậu hàng loạt, S3 Bucket được chặn hoàn toàn truy cập công cộng (Block Public Access). Truy cập ảnh chỉ được phép đi qua **Amazon CloudFront** sử dụng cấu hình **Origin Access Control (OAC)**.
*   **Luồng hoạt động:** Khi người dùng click vào sản phẩm → Frontend lấy dữ liệu thông số từ Backend → Trả về URL của ảnh có cấu trúc: `https://d1amspckjkbrez.cloudfront.net/products/filename.jpg`. CloudFront sẽ kiểm tra chữ ký OAC và lấy ảnh từ S3 trả về cho người dùng.

![Chi tiết sản phẩm](/images/5-Workshop/demo/product_detail.jpg)

---

##### E. Trang Tạo đơn hàng & Thanh toán (Checkout & PayOS QR Code)
Luồng thanh toán trực tuyến tự động không qua trung gian giúp doanh nghiệp tối ưu dòng tiền.

*   **Tích hợp hạ tầng:** 
    *   Tích hợp thư viện SDK của cổng thanh toán **PayOS** (hệ thống VietQR của Việt Nam).
*   **Luồng hoạt động:** 
    1.  Người dùng chọn sản phẩm vào giỏ hàng và nhấn **Đặt hàng & Thanh toán**.
    2.  Spring Boot API tạo một bản ghi đơn hàng mới trong **RDS PostgreSQL** ở trạng thái `PENDING_PAYMENT`.
    3.  Backend gọi API của PayOS để tạo một liên kết thanh toán duy nhất chứa mã đơn hàng và số tiền → Sinh ra mã QR Code động (VietQR).
    4.  Người dùng quét mã QR bằng ứng dụng ngân hàng để chuyển khoản trực tuyến.

![Tạo đơn và thanh toán](/images/5-Workshop/demo/checkout_payment.jpg)

---

##### F. Trang Thanh toán thành công (Order Success)
Trang hiển thị trạng thái hoàn thành giao dịch ngay lập tức mà không cần khách hàng phải gửi ảnh chụp màn hình chuyển khoản cho hỗ trợ viên.

*   **Tích hợp hạ tầng:** 
    *   Sử dụng cơ chế **Webhook** của PayOS kết nối tới API Endpoint của Spring Boot.
*   **Luồng hoạt động:** 
    1.  Ngay khi hệ thống ngân hàng nhận được tiền → PayOS gửi một tín hiệu HTTP POST (Webhook) kèm chữ ký bảo mật (Signature) về máy chủ DigitalOcean VPS.
    2.  Spring Boot Backend xác thực chữ ký bảo mật từ Webhook → Cập nhật trạng thái đơn hàng sang `PAID` trong **RDS PostgreSQL**.
    3.  Frontend Next.js (sử dụng WebSocket hoặc cơ chế polling) nhận trạng thái mới → Chuyển sang màn hình thông báo giao dịch thành công và mã hóa hóa đơn gửi về Email khách hàng qua **Amazon SES**.

![Thanh toán thành công](/images/5-Workshop/demo/order_success.jpg)

---

#### 2. Giao diện phía Quản trị viên (Admin Dashboard - Next.js Admin Panel)

##### A. Bảng điều khiển Quản trị (Admin Dashboard Overview)
Trang tổng quan giúp quản trị viên nắm bắt nhanh số liệu kinh doanh của cửa hàng laptop NovaTech.

*   **Tích hợp hạ tầng:** 
    *   Hệ thống sử dụng các câu lệnh SQL tối ưu hóa kết hợp hàm aggregate (`SUM`, `COUNT`) truy vấn trực tiếp xuống **RDS PostgreSQL** để tính toán tổng doanh số, tổng đơn hàng và số người dùng đăng ký mới.
    *   Nhằm tăng hiệu năng và tránh việc quá tải cho RDS Database chính khi Admin tải lại dashboard liên tục, các dữ liệu báo cáo này được cache tạm thời trong **Redis App Cache** nằm trên máy chủ DigitalOcean VPS với thời gian hết hạn (TTL) là 15 phút.

![Bảng điều khiển Admin](/images/5-Workshop/demo/admin_dashboard.jpg)

---

##### B. Quản lý Sản phẩm (Product Management)
Nơi quản trị viên thêm mới, cập nhật giá bán, cấu hình chi tiết hoặc xóa thông tin sản phẩm laptop.

*   **Tích hợp hạ tầng:** 
    *   Sử dụng **AWS SDK Java S3 Client** tích hợp trong Spring Boot để tải ảnh sản phẩm trực tiếp từ Admin Dashboard lên Amazon S3 Bucket.
*   **Luồng hoạt động:** 
    1.  Admin điền form thông số laptop và chọn tệp ảnh tải lên → Gửi lên Spring Boot API.
    2.  Spring Boot Backend tạo yêu cầu `PutObjectRequest` gửi tệp ảnh lên S3 bucket `novatech-product-images` → Nhận về khóa định danh ảnh (Object Key).
    3.  Backend lưu Object Key và các thuộc tính laptop vào bảng `products` của **RDS PostgreSQL**.

![Quản lý sản phẩm](/images/5-Workshop/demo/admin_products.jpg)

---

##### C. Quản lý Đơn hàng (Order Management)
Quản trị viên có thể theo dõi và lọc trạng thái tất cả các đơn hàng trong hệ thống (Đang xử lý, Đã thanh toán, Đã hủy, Đang giao hàng).

*   **Tích hợp hạ tầng:** 
    *   Bảng dữ liệu `orders` và `order_items` kết nối chặt chẽ trong **RDS PostgreSQL** để bảo toàn tính nhất quán và ràng buộc khóa ngoại (Foreign Key Integrity).

![Quản lý đơn hàng](/images/5-Workshop/demo/admin_orders.jpg)

---

##### D. Quản lý Khách hàng (User Management)
Hiển thị danh sách khách hàng đã đăng ký sử dụng dịch vụ trên hệ thống.

*   **Tích hợp hạ tầng:** 
    *   Mặc dù dữ liệu xác thực nằm an toàn trên **Amazon Cognito User Pool**, một số thông tin hiển thị cơ bản (như Tên hiển thị, Số điện thoại, Địa chỉ giao hàng) được lưu đồng bộ về bảng `users` trong **RDS PostgreSQL** để phục vụ việc liên kết đơn hàng.

![Quản lý khách hàng](/images/5-Workshop/demo/admin_users.jpg)

---

##### E. Quản lý Mã giảm giá (Voucher Management)
Nơi Admin tạo các mã giảm giá theo phần trăm hoặc số tiền cố định để kích cầu mua sắm.

*   **Tích hợp hạ tầng:** 
    *   Quản lý trong bảng `vouchers` thuộc **RDS PostgreSQL**.
    *   Spring Boot Backend xử lý logic kiểm tra thời gian hiệu lực và số lượng voucher khả dụng khi người dùng áp dụng mã ở màn hình thanh toán.

![Quản lý mã giảm giá](/images/5-Workshop/demo/admin_vouchers.jpg)

---

##### F. Báo cáo & Thống kê Doanh thu (Reports)
Cung cấp các biểu đồ dạng đường, dạng cột thể hiện chi tiết tốc độ tăng trưởng doanh thu theo ngày, tuần, tháng.

*   **Tích hợp hạ tầng:** 
    *   Dữ liệu được tổng hợp từ bảng `payments` và `orders` trong **RDS PostgreSQL** và trả về định dạng JSON cho thư viện Chart.js ở Frontend hiển thị.

![Báo cáo thống kê](/images/5-Workshop/demo/admin_reports.jpg)
