---
title: "Tuần 7 Worklog"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---
### Mục tiêu Tuần 7:

* Cài đặt, cấu hình và khởi tạo hạ tầng hàng đợi tin nhắn Message Queue (RabbitMQ/Kafka).

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | So sánh RabbitMQ và Kafka, chọn RabbitMQ vì phù hợp tiêu chuẩn AMQP và có cơ chế định tuyến linh hoạt. | 27/05/2026 | 27/05/2026 |  |
| 3 | Viết mã nguồn Docker Compose khởi chạy container RabbitMQ tích hợp giao diện quản trị trực quan. | 28/05/2026 | 28/05/2026 | <https://www.docker.com/> |
| 4 | Định cấu hình Connection Factory, các loại Exchange, Queue và Binding Key trong file cấu hình Spring Boot. | 29/05/2026 | 29/05/2026 | <https://spring.io/projects/spring-amqp> |
| 5 | Cấu hình thư viện Jackson Message Converter để tự động mã hóa dữ liệu DTO sang định dạng JSON gửi đi. | 01/06/2026 | 01/06/2026 |  |
| 6 | Xác thực cấu hình hàng đợi và mối kết kết nối bằng cách theo dõi trang quản trị HTTP Console của RabbitMQ. | 02/06/2026 | 02/06/2026 |  |

### Kết quả đạt được trong Tuần 7:

* Dựng thành công dịch vụ RabbitMQ chạy ổn định bằng Docker phục vụ kết nối hàng đợi tin nhắn.
* Cấu hình thành công Spring Boot AMQP cho phép định tuyến tin nhắn linh hoạt qua các Exchange.
* Quy chuẩn hóa cấu hình chuyển đổi dữ liệu JSON tự động giúp việc truyền tải tin nhắn ổn định.
