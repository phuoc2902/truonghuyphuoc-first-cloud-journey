---
title: "Tuần 9 Worklog"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---
### Mục tiêu Tuần 9:

* Xây dựng bộ Consumer lắng nghe hàng đợi để xử lý gửi Email và Thông báo hệ thống.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Cấu hình thư viện JavaMailSender kết nối máy chủ gửi thư SMTP (như Gmail hoặc AWS SES). | 10/06/2026 | 10/06/2026 | <https://spring.io/guides/gs/sending-email/> |
| 3 | Viết class `EmailEventConsumer` sử dụng decorator `@RabbitListener` lắng nghe queue để lấy dữ liệu gửi mail. | 11/06/2026 | 11/06/2026 | <https://spring.io/projects/spring-amqp> |
| 4 | Thiết kế các mẫu email HTML chuyên nghiệp phản hồi tốt trên điện thoại bằng Thymeleaf. | 12/06/2026 | 12/06/2026 | <https://www.thymeleaf.org/> |
| 5 | Xây dựng `NotificationEventConsumer` lưu vết thông báo thông báo hệ thống vào database. | 15/06/2026 | 15/06/2026 |  |
| 6 | Kiểm thử đầu cuối hệ thống: Đặt hàng từ Next.js -> Đẩy MQ -> Consumer nhận tin -> Nhận email thực tế. | 16/06/2026 | 16/06/2026 |  |

### Kết quả đạt được trong Tuần 9:

* Hệ thống xử lý bất đồng bộ hoạt động tự động và ổn định, tự động gửi lại tin nhắn khi gặp sự cố tạm thời.
* Biên dịch thành công các email định dạng HTML động qua Thymeleaf và gửi thư điện tử nhanh chóng.
* Các thông báo hệ thống được ghi nhận đầy đủ, sẵn sàng cho việc đẩy real-time bằng WebSockets.
