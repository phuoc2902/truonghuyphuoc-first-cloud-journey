---
title: "Bản đề xuất dự án: Hệ thống Thương mại Điện tử Laptop NovaTech & AI Chatbot MVP"
linkTitle: "Bản đề xuất"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Giải pháp Tích hợp Đám mây Lai (AWS Hybrid Integration) & Trí tuệ Nhân tạo sinh (Generative AI)

---

### 1. Tóm tắt điều hành

Dự án **NovaTech** là một hệ thống thương mại điện tử chuyên kinh doanh máy tính xách tay (Laptop E-Commerce Shop). Nhằm tối ưu hóa chi phí vận hành trong giai đoạn khởi đầu (MVP) nhưng vẫn đảm bảo tính bảo mật cao, độ sẵn sàng dữ liệu tuyệt đối và khả năng tương tác thông minh với khách hàng, chúng tôi đề xuất kiến trúc **Đám mây lai (Hybrid Cloud)** kết hợp giữa **Amazon Web Services (AWS)** và **DigitalOcean VPS**.

Điểm nhấn công nghệ của dự án là việc tích hợp **Trí tuệ nhân tạo (AI Chatbot)** thông qua dịch vụ **Amazon Bedrock**, cho phép khách hàng tự động hỏi đáp về thông số kỹ thuật, so sánh laptop và nhận tư vấn mua sắm 24/7 dựa trên cơ sở tri thức (RAG) được lưu trữ riêng tư.

---

### 2. Tuyên bố vấn đề

#### Thách thức của các hệ thống E-commerce hiện nay:
- **Tư vấn khách hàng**: Việc phản hồi khách hàng 24/7 đòi hỏi lượng lớn nhân sự trực page, dẫn đến chi phí vận hành cao nhưng độ chính xác về thông số sản phẩm không đồng đều.
- **Bảo mật và toàn vẹn dữ liệu**: Cơ sở dữ liệu (Database) chứa thông tin người dùng, đơn hàng và thanh toán rất dễ bị tấn công nếu đặt chung trên máy chủ web công cộng.
- **Chi phí hạ tầng**: Việc thuê các cấu hình máy chủ lớn trên các đám mây công cộng trong giai đoạn MVP có thể gây lãng phí ngân sách lớn nếu lượng truy cập ban đầu chưa ổn định.

#### Giải pháp đề xuất từ NovaTech:
1. **Kiến trúc Lai tối ưu chi phí**: Sử dụng máy chủ ảo **DigitalOcean VPS** (chi phí cố định thấp) để chạy Web Server / API Backend. Cơ sở dữ liệu quan trọng **PostgreSQL** được đưa vào vùng mạng riêng tư (**Private Subnet**) trên **AWS RDS (Multi-AZ)** giúp bảo vệ dữ liệu khỏi internet công cộng.
2. **Đường truyền VPN mã hóa**: Kết nối hai môi trường thông qua đường truyền **AWS Site-to-Site VPN** bảo mật tuyệt đối.
3. **AI Chatbot trợ lý ảo**: Sử dụng **Amazon Bedrock** kết hợp phương pháp **RAG (Retrieval-Augmented Generation)** để chatbot truy xuất dữ liệu từ các tài liệu FAQ/Context lưu trữ trên **Amazon S3**, hỗ trợ người dùng tìm kiếm sản phẩm thông minh và nhanh chóng.
4. **Phân phối toàn cầu ở rìa**: Triển khai frontend Next.js lên **Amazon S3** và phân phối qua **Amazon CloudFront** kết hợp **AWS WAF** giúp tăng tốc độ tải trang và ngăn chặn các cuộc tấn công DDoS/SQL Injection.

---

### 3. Kiến trúc giải pháp

Dưới đây là mô hình kiến trúc chi tiết của hệ thống thương mại điện tử **NovaTech**:

![NovaTech AWS Hybrid MVP Architecture](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.drawio.png)

#### Các thành phần chính trong kiến trúc:

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

---

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai dự án:
1. **Thiết kế & Khởi tạo (Tuần 1)**:
   - Thiết kế ERD cơ sở dữ liệu chi tiết cho hệ thống Laptop Shop.
   - Khởi tạo VPC, Private Subnets và thiết lập kết nối AWS Site-to-Site VPN nối sang VPS DigitalOcean.
2. **Thiết lập Network & Bảo mật (Tuần 2)**:
   - Cấu hình VPN Site-to-Site thông suốt giữa hai môi trường.
   - Thiết lập cấu hình Route 53, WAF và tích hợp xác thực Amazon Cognito User Pool.
3. **Phát triển Backend & Cấu hình RDS (Tuần 3 - 5)**:
   - Viết các API Spring Boot quản lý sản phẩm, giỏ hàng, đơn hàng và thanh toán.
   - Cấu hình Amazon RDS PostgreSQL Multi-AZ và liên kết thông tin qua AWS Secrets Manager.
   - Cấu hình xác thực Cognito User Pool kết hợp Spring Boot Security để bảo mật các endpoint.
4. **Phát triển Frontend & Tích hợp AI Chatbot (Tuần 6 - 8)**:
   - Xây dựng giao diện Next.js cho người dùng và bảng quản trị (Admin Dashboard).
   - Tích hợp khung Chatbot UI trên Web Frontend, kết nối đến Amazon Bedrock sử dụng AWS SDK để xử lý hội thoại RAG.
   - Tích hợp cổng thanh toán PayOS để hoàn thiện luồng mua sắm.
5. **Kiểm thử, Đóng gói & Triển khai (Tuần 9)**:
   - Triển khai CI/CD tự động bằng GitLab CI/CD.
   - Deploy Frontend lên S3/CloudFront và cấu hình Route 53, WAF.
   - Kiểm thử hiệu năng hệ thống qua VPN và độ chính xác của AI Chatbot.

---

### 5. Lộ trình & Mốc triển khai

| Mốc thời gian | Hoạt động chính | Kết quả mong đợi |
| --- | --- | --- |
| **Tuần 1** | Nghiên cứu và vẽ kiến trúc hệ thống | Hoàn thành sơ đồ kiến trúc Hybrid Cloud & sơ đồ ERD DB. |
| **Tuần 2** | Thiết lập Network & Bảo mật | Thiết lập VPN Site-to-Site thông suốt, cấu hình Route 53, Cognito. |
| **Tuần 3 - 5** | Phát triển API Backend & Database | Hoàn thành Spring Boot API kết nối RDS PostgreSQL, tích hợp PayOS. |
| **Tuần 6 - 8** | Phát triển Frontend & Tích hợp Bedrock AI | Giao diện Next.js hoàn chỉnh, chatbot tư vấn hoạt động ổn định. |
| **Tuần 9** | Kiểm thử bảo mật, CI/CD & Bàn giao | Hệ thống chạy chính thức trên môi trường production, báo cáo nghiệm thu. |

---

### 6. Ước tính ngân sách (Dự kiến hàng tháng)

Dưới đây là bảng ước tính chi phí hạ tầng vận hành cho giai đoạn MVP (lượng người dùng trung bình):

| Dịch vụ | Nhà cung cấp | Vai trò | Chi phí ước tính / Tháng |
| --- | --- | --- | --- |
| **Amazon RDS PostgreSQL** | AWS | Cơ sở dữ liệu chính (db.t4g.micro Multi-AZ) | ~ $30.00 |
| **Amazon Cognito** | AWS | Xác thực người dùng (Miễn phí 50,000 MAUs đầu) | $0.00 |
| **Amazon Bedrock** | AWS | Xử lý hội thoại AI Chatbot (Mô hình Claude 3 Haiku) | ~ $5.00 (Tùy theo lượng token sử dụng) |
| **Amazon CloudFront & S3** | AWS | Phân phối Frontend tĩnh & Lưu trữ RAG | ~ $2.00 |
| **AWS Site-to-Site VPN** | AWS | Kết nối bảo mật AWS - DigitalOcean | ~ $36.00 ($0.05/giờ mỗi tunnel) |
| **DigitalOcean VPS** | DigitalOcean | Chạy Spring Boot Backend & Redis App Cache | $12.00 (2 vCPUs, 2GB RAM, 60GB SSD) |
| **Amazon SES & WAF** | AWS | Gửi mail & Bảo mật ứng dụng ở biên | ~ $5.00 |
| **Tổng cộng** | | **Chi phí vận hành hệ thống lai MVP** | **~ $90.00 / Tháng** |

---

### 7. Đánh giá rủi ro & Phương án dự phòng

- **Rủi ro 1: Mất kết nối VPN giữa VPS và AWS RDS**
  - *Ảnh hưởng*: Backend Spring Boot không thể kết nối tới Database PostgreSQL, hệ thống ngắt quãng dịch vụ.
  - *Khắc phục*: Thiết lập VPN dự phòng song song (Dual IPSec Tunnels) trên AWS và cấu hình tự động chuyển đổi luồng định tuyến (failover) ở phía Customer Gateway.
- **Rủi ro 2: Chi phí gọi API Amazon Bedrock tăng cao**
  - *Ảnh hưởng*: Khách hàng spam chatbot dẫn đến hóa đơn AWS tăng đột ngột.
  - *Khắc phục*: Triển khai Rate Limiting trên API Gateway / WAF và giới hạn số lượt chat tối đa trên mỗi phiên đăng nhập của người dùng.
- **Rủi ro 3: Trễ mạng (Latency) qua kết nối VPN**
  - *Ảnh hưởng*: Thời gian tải trang hoặc phản hồi thông tin sản phẩm bị chậm.
  - *Khắc phục*: Tận dụng Redis App Cache đặt trực tiếp tại DigitalOcean VPS để lưu trữ thông tin sản phẩm tĩnh, chỉ gọi Database RDS đối với các tác vụ ghi (đặt hàng, cập nhật tài khoản).

---

### 8. Kết quả kỳ vọng

- **Tính thực tiễn cao**: Xây dựng thành công một website thương mại điện tử hoàn chỉnh, vận hành thực tế trên môi trường đám mây lai.
- **Trải nghiệm mua sắm thông minh**: Nâng cao trải nghiệm người dùng thông qua trợ lý ảo AI hiểu sản phẩm và tư vấn cá nhân hóa.
- **Bảo mật chuẩn doanh nghiệp**: Dữ liệu khách hàng được bảo vệ trong lớp mạng nội bộ an toàn tuyệt đối, giảm thiểu rủi ro rò rỉ dữ liệu.