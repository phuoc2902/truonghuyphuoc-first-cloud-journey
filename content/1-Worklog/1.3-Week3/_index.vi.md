---
title: "Tuần 3 Worklog"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---
### Mục tiêu Tuần 3:

* Hiện thực hóa quản lý trạng thái đăng nhập toàn cục và thiết kế khung giao diện (Layout Skeletons).

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Đánh giá các thư viện quản lý state và quyết định sử dụng Zustand vì tính gọn nhẹ và hiệu năng cao. | 29/04/2026 | 29/04/2026 | <https://github.com/pmndrs/zustand> |
| 3 | Xây dựng auth store quản lý thông tin profile, lưu trữ tokens và các action đăng nhập/đăng xuất. | 30/04/2026 | 30/04/2026 | <https://github.com/pmndrs/zustand> |
| 4 | Thiết kế các layout cơ sở của trang web bao gồm Header, Sidebar phản hồi thiết bị di động và Footer. | 01/05/2026 | 01/05/2026 |  |
| 5 | Liên kết Axios client interceptors với Zustand store để đồng bộ hóa trạng thái phiên làm việc. | 04/05/2026 | 04/05/2026 |  |
| 6 | Thiết kế các loading layout skeleton giúp chuyển đổi giao diện mượt mà khi hệ thống đang tải dữ liệu. | 05/05/2026 | 05/05/2026 |  |

### Kết quả đạt được trong Tuần 3:

* Tích hợp thành công Zustand state store giúp quản lý đồng bộ phiên đăng nhập của người dùng.
* Hoàn thiện các component loading skeleton tránh xê dịch giao diện và tăng tốc độ cảm nhận của người dùng.
* Kết nối đồng bộ trạng thái đăng nhập frontend với cookie bảo mật của trình duyệt.
