---
title: "Tuần 8 Worklog"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---
### Mục tiêu Tuần 8:

* Xây dựng mô hình Publisher gửi sự kiện (Email, Notification) bất đồng bộ sang Message Queue.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Thiết kế các thực thể Event DTO chứa thông tin sự kiện như đặt hàng thành công và tạo tài khoản. | 03/06/2026 | 03/06/2026 |  |
| 3 | Viết service lớp gửi tin `EmailEventPublisher` sử dụng `RabbitTemplate` để đẩy tin nhắn định dạng JSON. | 04/06/2026 | 04/06/2026 | <https://spring.io/projects/spring-amqp> |
| 4 | Tạo service `NotificationEventPublisher` chuyên gửi các sự kiện tạo thông báo nội bộ hệ thống. | 05/06/2026 | 05/06/2026 |  |
| 5 | Tái cấu trúc các API chính (như API thanh toán) để kích hoạt Publisher thay vì xử lý tuần tự trực tiếp. | 08/06/2026 | 08/06/2026 |  |
| 6 | Kiểm thử tải để đảm bảo việc xuất bản tin nhắn chạy ngầm không cản trở tốc độ phản hồi của API. | 09/06/2026 | 09/06/2026 |  |

### Kết quả đạt được trong Tuần 8:

* Tách biệt hoàn toàn API giao dịch chính khỏi các tác vụ ngoại vi tốn thời gian (kết nối máy chủ SMTP).
* Đảm bảo tính toàn vẹn dữ liệu: tin nhắn chỉ được gửi đi khi các thao tác database trước đó lưu thành công.
* Giảm thiểu thời gian chờ đợi phản hồi của khách hàng khi thanh toán từ đơn vị giây xuống mili-giây.
