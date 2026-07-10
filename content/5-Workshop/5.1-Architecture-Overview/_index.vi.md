---
title: "5.1. Sơ đồ kiến trúc & Tổng quan"
date: 2026-07-06
weight: 1
---

#### 1. Bối cảnh dự án (Business Context)

**NovaTech** là một nền tảng thương mại điện tử chuyên biệt bán các sản phẩm công nghệ (Laptop, Phụ kiện). Nỗi đau (pain point) lớn nhất của khách hàng khi mua laptop trực tuyến là họ **không hiểu rõ các thông số kỹ thuật phức tạp** (như CPU dòng U hay H, RAM bus bao nhiêu, v.v.) dẫn đến chần chừ và tỷ lệ thoát trang cao.

**Giải pháp:** Tích hợp một trợ lý ảo AI Chatbot ngay trên giao diện cửa hàng. Thay vì phải Google tìm hiểu, khách hàng chỉ cần hỏi *"Tư vấn laptop lập trình dưới 20 triệu"*, AI sẽ phân tích nhu cầu và đề xuất chính xác sản phẩm có trong kho dữ liệu của cửa hàng.

#### 2. Sơ đồ hạ tầng lai (Hybrid Cloud Architecture)

Dưới đây là sơ đồ kiến trúc hệ thống thực tế của dự án, được thiết kế theo mô hình đám mây lai (Hybrid Cloud):

![Kiến trúc Hybrid Cloud NovaTech](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.drawio.png)

#### 3. Tại sao lại lựa chọn Hybrid Cloud?

Thay vì đưa 100% ứng dụng lên AWS (sử dụng EC2/ECS/EKS) vốn đòi hỏi kỹ năng vận hành phức tạp và tốn kém chi phí duy trì hàng tháng cho các dự án khởi nghiệp/MVP, dự án NovaTech áp dụng chiến lược phân tách (Decoupling):
- **Core Application (Logic chạy liên tục):** Đặt trên máy chủ VPS ảo của **DigitalOcean** (Droplet) với mức phí cố định, rẻ và dễ dàng mở rộng khi cần thiết.
- **Dữ liệu & AI (Tính khả dụng cao, Bảo mật):** Các dịch vụ cốt lõi không thể thỏa hiệp về bảo mật và sự ổn định như Cơ sở dữ liệu (RDS), Lưu trữ ảnh (S3), Định danh người dùng (Cognito) và Trí tuệ nhân tạo (Bedrock) được giao phó hoàn toàn cho **AWS Cloud**.

#### 4. Phân tích luồng dữ liệu (Data Flow)

Hệ thống được vận hành với sự kết hợp chặt chẽ giữa các dịch vụ:

1. **Khách hàng (Client/Browser)**: 
   - Truy cập giao diện ứng dụng web được build bằng **Next.js**. Giao diện này có thể triển khai trên Vercel hoặc Amplify.
   - Khi người dùng đăng ký/đăng nhập, trình duyệt sẽ gửi yêu cầu trực tiếp đến **Amazon Cognito** để nhận mã Token xác thực bảo mật (JWT).

2. **Máy chủ ứng dụng (Application Server - DigitalOcean)**:
   - Chạy dịch vụ Backend **Spring Boot API** đóng vai trò xử lý toàn bộ logic nghiệp vụ của website (quản lý sản phẩm, giỏ hàng, thanh toán qua PayOS API).
   - Backend API sử dụng Access Key/Secret Key lấy từ môi trường bảo mật để kết nối an toàn đến các dịch vụ đám mây AWS.

3. **Cơ sở dữ liệu đám mây (Amazon RDS PostgreSQL)**:
   - Được thiết lập bên trong mạng VPC an toàn của AWS.
   - Nhằm đảm bảo bảo mật dữ liệu, cơ sở dữ liệu chỉ chấp nhận các truy vấn kết nối đến từ địa chỉ IP tĩnh (Elastic IP) của **DigitalOcean App Server** thông qua các quy tắc bảo mật của **Security Group**.

4. **Lưu trữ đám mây (Amazon S3)**:
   - Chứa 2 buckets chính:
     - `novatech-product-images`: Lưu trữ và phân phối hình ảnh sản phẩm laptop thương mại.
     - `novatech-chatbot-rag`: Lưu trữ tài liệu tri thức (các file `.pdf`, `.txt`) phục vụ quá trình huấn luyện/RAG cho chatbot AI.

5. **Trợ lý trí tuệ nhân tạo (Amazon Bedrock)**:
   - Backend Spring Boot sẽ gọi trực tiếp đến API Amazon Bedrock sử dụng mô hình **Claude 3 Haiku/Sonnet** để trả lời các câu hỏi về thông số kỹ thuật laptop từ người dùng một cách chính xác dựa trên cơ sở tri thức RAG được lưu trữ ở S3.
