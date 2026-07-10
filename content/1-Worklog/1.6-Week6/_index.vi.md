---
title: "Tuần 6 Worklog"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu Tuần 6:

* Hiện thực hóa cơ chế đồng bộ giỏ hàng (Cart Sync) từ client lên server khi đăng nhập thành công.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Định nghĩa quy luật giải quyết xung đột khi gộp giỏ hàng (cộng dồn số lượng, cập nhật đơn giá mới). | 20/05/2026 | 20/05/2026 |  |
| 3 | Xây dựng hàm xử lý đồng bộ ở backend nhận mảng dữ liệu từ client và thực thi gộp vào database. | 21/05/2026 | 21/05/2026 |  |
| 4 | Cấu hình luồng đăng nhập phía frontend để gửi giỏ hàng ở Local Storage lên API đồng bộ. | 22/05/2026 | 22/05/2026 |  |
| 5 | Viết các logic xác thực bổ sung để xử lý các sản phẩm hết hàng hoặc thay đổi trạng thái lúc đồng bộ. | 25/05/2026 | 25/05/2026 |  |
| 6 | Thiết lập cơ chế xóa sạch giỏ hàng local chỉ khi API phản hồi trạng thái đồng bộ hoàn thành. | 26/05/2026 | 26/05/2026 |  |

### Kết quả đạt được trong Tuần 6:

* Đồng bộ hóa hoàn toàn giỏ hàng của người dùng khi chuyển từ khách vãng lai sang thành viên đã đăng nhập.
* Tối ưu hóa số lượng truy vấn cơ sở dữ liệu khi đồng bộ thông qua các hàm lưu trữ hàng loạt (batch save).
* Giỏ hàng cục bộ được làm sạch tự động sau khi gửi dữ liệu lên server, tránh trùng lặp hiển thị.
