---
title: "Tuần 5 Worklog"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---
### Mục tiêu Tuần 5:

* Phát triển hệ thống REST API giỏ hàng ở Backend bằng Spring Boot và thiết kế cơ sở dữ liệu.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Thiết kế lược đồ quan hệ thực thể cho các bảng Cart và CartItem liên kết người dùng với sản phẩm. | 13/05/2026 | 13/05/2026 |  |
| 3 | Khai báo các lớp thực thể JPA Entities và giao diện Spring Data Repository để thao tác dữ liệu. | 14/05/2026 | 14/05/2026 | <https://spring.io/projects/spring-data-jpa> |
| 4 | Viết Service Layer hiện thực hóa nghiệp vụ giỏ hàng và lưu vết thông tin xuống database. | 15/05/2026 | 15/05/2026 |  |
| 5 | Cung cấp các API endpoint thông qua CartController bảo mật bằng cơ chế JWT của Spring Security. | 18/05/2026 | 18/05/2026 | <https://spring.io/projects/spring-security> |
| 6 | Viết mã kiểm thử tự động kiểm tra tham số đầu vào và định tuyến API bằng MockMvc. | 19/05/2026 | 19/05/2026 |  |

### Kết quả đạt được trong Tuần 5:

* Thiết kế cơ sở dữ liệu quan hệ hoàn tất và đã được nghiệm thu cấu trúc thực thể.
* Xây dựng thành công hệ thống REST API bảo mật cho phép quản lý giỏ hàng đồng bộ trên máy chủ.
* Lớp nghiệp vụ (Service) xử lý chính xác việc thêm sản phẩm, tính toán giá trị và kiểm tra số lượng tồn kho.
