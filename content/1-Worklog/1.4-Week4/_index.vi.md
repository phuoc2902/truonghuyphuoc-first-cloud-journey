---
title: "Tuần 4 Worklog"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu Tuần 4:

* Xây dựng logic giỏ hàng (Cart) ở Frontend kết hợp lưu trữ Local Storage cho khách vãng lai.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Phân tích các yêu cầu nghiệp vụ giỏ hàng gồm tính toán giá, thêm sản phẩm và giới hạn số lượng kho. | 06/05/2026 | 06/05/2026 |  |
| 3 | Xây dựng cart store bằng Zustand kết hợp middleware persist để tự động lưu trạng thái xuống Local Storage. | 07/05/2026 | 07/05/2026 | <https://github.com/pmndrs/zustand> |
| 4 | Phát triển các sự kiện UI: thêm mới, xóa bỏ, làm trống giỏ hàng và cập nhật nhanh số lượng sản phẩm. | 08/05/2026 | 08/05/2026 |  |
| 5 | Thiết kế giao diện ngăn kéo giỏ hàng (Cart Drawer) tiện lợi và trang xem trước thanh toán. | 11/05/2026 | 11/05/2026 |  |
| 6 | Kiểm thử tính nhất quán của giỏ hàng khi Local Storage bị xóa hoặc bị thay đổi thủ công từ devtools. | 12/05/2026 | 12/05/2026 |  |

### Kết quả đạt được trong Tuần 4:

* Hoàn thành hệ thống giỏ hàng phía Client hoạt động độc lập không phụ thuộc vào độ trễ mạng.
* Đảm bảo dữ liệu giỏ hàng của khách vãng lai được lưu trữ an toàn qua Local Storage khi tải lại trang.
* Thiết kế giao diện ngăn kéo giỏ hàng (Cart Drawer) đẹp mắt có kèm các chuyển động mượt mà khi cập nhật.
