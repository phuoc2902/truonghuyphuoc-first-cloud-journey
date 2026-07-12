---
title: "5.1. Sơ đồ kiến trúc & Tổng quan"
date: 2026-07-06
weight: 1
---

#### 1. Bối cảnh dự án (Business Context)

**NovaTech** là một nền tảng thương mại điện tử chuyên biệt bán các sản phẩm công nghệ (Laptop, Phụ kiện). Nỗi đau (pain point) lớn nhất của khách hàng khi mua laptop trực tuyến là họ **không hiểu rõ các thông số kỹ thuật phức tạp** (như CPU dòng U hay H, RAM bus bao nhiêu, v.v.) dẫn đến chần chừ và tỷ lệ thoát trang cao.

**Giải pháp:** Tích hợp một trợ lý ảo AI Chatbot ngay trên giao diện cửa hàng. Thay vì phải Google tìm hiểu, khách hàng chỉ cần hỏi *"Tư vấn laptop lập trình dưới 20 triệu"*, AI sẽ phân tích nhu cầu và đề xuất chính xác sản phẩm có trong kho dữ liệu của cửa hàng.

---

#### 2. Sơ đồ hạ tầng lai (Hybrid Cloud Architecture)

Dưới đây là sơ đồ kiến trúc hệ thống thực tế của dự án, được thiết kế theo mô hình đám mây lai (Hybrid Cloud) kết hợp giữa AWS Cloud và máy chủ DigitalOcean:

![Kiến trúc Hybrid Cloud NovaTech](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.svg)

---

#### 3. Các thành phần chính trong kiến trúc:

##### A. Lớp Biên & Phân phối Nội dung (Edge Front Door - Outside VPC)
- **Amazon Route 53**: Dịch vụ DNS phân giải tên miền hệ thống.
- **AWS WAF (Web Application Firewall)**: Bộ lọc mã độc và bảo vệ API khỏi các cuộc tấn công OWASP Top 10.
- **Amazon CloudFront**: CDN phân phối giao diện web tĩnh (Next.js SPA) và định tuyến các API request.
- **Amazon S3 (Private SPA origin)**: Lưu trữ mã nguồn frontend tĩnh một cách an toàn.
- **AWS Certificate Manager (ACM)**: Quản lý và cấp phát chứng chỉ SSL/TLS tự động.

##### B. Máy chủ Ứng dụng (External Environment - DigitalOcean VPS)
- **Nginx Reverse Proxy**: Nhận yêu cầu từ CloudFront và chuyển tiếp nội dung đến API.
- **Spring Boot API (Java)**: Khối xử lý nghiệp vụ backend chính của NovaTech, thực hiện xác thực token JWT do Amazon Cognito cấp.
- **Redis App Cache**: Bộ nhớ đệm giúp lưu trữ tạm thời các truy vấn sản phẩm và giảm tải cho Database.

##### C. Cơ sở dữ liệu và Mạng riêng tư (AWS Cloud VPC)
- **AWS Site-to-Site VPN**: Cung cấp đường truyền mã hóa (IPSec VPN Tunnel) giữa Customer Gateway (phía DigitalOcean) và Virtual Private Gateway (phía AWS).
- **Amazon RDS PostgreSQL (Multi-AZ)**: Đặt hoàn toàn trong Private Subnet. Phiên bản Primary ở AZ A sẽ xử lý ghi/đọc chính, trong khi phiên bản Standby ở AZ B sẽ tự động đồng bộ hóa để sẵn sàng khôi phục khi xảy ra sự cố (High Availability).

##### D. Lớp dịch vụ AI Chatbot & Trợ lý ảo (Regional AWS Services)
- **Amazon Cognito**: Quản lý đăng ký, đăng nhập và xác thực của người dùng (hỗ trợ liên kết tài khoản Google OAuth 2.0).
- **Amazon Bedrock**: Dịch vụ chạy mô hình ngôn ngữ lớn (LLM) xử lý câu hỏi của khách hàng.
- **Amazon S3 (RAG Context)**: Lưu trữ các tài liệu FAQ, tài liệu hướng dẫn và dữ liệu sản phẩm để cung cấp ngữ cảnh chính xác cho Bedrock AI.
- **AWS Secrets Manager**: Quản lý tập trung thông tin đăng nhập Database và các mã khóa API của cổng thanh toán.

##### E. Giám sát & Vận hành (Security & Observability)
- **Amazon CloudWatch**: Ghi nhận logs hệ thống và các chỉ số tài nguyên.
- **AWS CloudTrail**: Ghi lại nhật ký các API call trên tài khoản AWS để phục vụ kiểm toán bảo mật.
- **Amazon SES (Simple Email Service)**: Tự động gửi email xác thực tài khoản, hóa đơn đơn hàng cho khách hàng.
- **CI/CD Pipeline**: Tự động kiểm thử, build và deploy từ GitLab/GitHub Repo đến DigitalOcean VPS thông qua SSH deploy.
- **Cổng thanh toán PayOS**: Tích hợp cổng thanh toán trực tuyến của Việt Nam giúp xử lý giao dịch đơn hàng qua ngân hàng nhanh chóng.
