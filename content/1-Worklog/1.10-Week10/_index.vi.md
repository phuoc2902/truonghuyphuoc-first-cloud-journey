---
title: "Tuần 10 Worklog"
date: 2024-01-01
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---
### Mục tiêu Tuần 10:

* Viết mã kiểm thử Unit Test và Integration Test đảm bảo độ ổn định của toàn hệ thống Backend.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Thiết lập thư viện JUnit 5, Mockito và Spring Boot Test trong file cấu hình dự án Maven. | 17/06/2026 | 17/06/2026 | <https://junit.org/junit5/> |
| 3 | Viết unit test cho các service xử lý giỏ hàng dùng Mockito mô phỏng tầng kết nối repository. | 18/06/2026 | 18/06/2026 | <https://site.mockito.org/> |
| 4 | Viết các test case kiểm tra nghiệp vụ đồng bộ giỏ hàng và các trường hợp lỗi dữ liệu đầu vào. | 19/06/2026 | 19/06/2026 |  |
| 5 | Xây dựng các test case giả lập đầu ra của RabbitTemplate để kiểm tra định dạng dữ liệu gửi đi. | 22/06/2026 | 22/06/2026 |  |
| 6 | Xây dựng các kịch bản kiểm thử tích hợp (Integration Test) API Controllers bằng MockMvc. | 23/06/2026 | 23/06/2026 |  |

### Kết quả đạt được trong Tuần 10:

* Đạt tỷ lệ bao phủ kiểm thử (Code Coverage) trên 80% cho toàn bộ các lớp logic nghiệp vụ chính.
* Sử dụng mock database giúp kịch bản kiểm thử chạy độc lập không ảnh hưởng dữ liệu thực tế.
* Tích hợp các kịch bản kiểm thử tự động hạn chế tối đa các lỗi cũ phát sinh trở lại.
