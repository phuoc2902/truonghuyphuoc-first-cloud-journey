---
title: "Tuần 2 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---
### Mục tiêu Tuần 2:

* Cấu hình Axios Interceptors phục vụ việc đính kèm Request ID truy vết và tự động xử lý JWT token.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Nghiên cứu mô hình Axios Interceptor và kiến trúc luồng làm mới token tự động. | 22/04/2026 | 22/04/2026 | <https://axios-http.com/docs/interceptors> |
| 3 | Khởi tạo Axios instance, viết Request Interceptor để tự động chèn mã RequestID vào header. | 23/04/2026 | 23/04/2026 | <https://axios-http.com/docs/interceptors> |
| 4 | Viết Response Interceptor bắt lỗi HTTP 401 Unauthorized khi token của người dùng hết hạn. | 24/04/2026 | 24/04/2026 |  |
| 5 | Xây dựng logic bất đồng bộ gọi API refresh token để lấy token mới và gửi lại request lỗi trước đó. | 27/04/2026 | 27/04/2026 |  |
| 6 | Tạo các phản hồi HTTP giả lập và viết bài kiểm thử cho cấu hình Axios client. | 28/04/2026 | 28/04/2026 |  |

### Kết quả đạt được trong Tuần 2:

* Cấu hình hoàn chỉnh HTTP Client Axios dùng chung phục vụ liên lạc với máy chủ API.
* Tích hợp thành công cơ chế gán mã RequestID duy nhất trên mỗi request gửi đi để phục vụ log tracing.
* Tự động hóa luồng làm mới access token bị hết hạn dưới nền mà không gây gián đoạn trải nghiệm người dùng.
