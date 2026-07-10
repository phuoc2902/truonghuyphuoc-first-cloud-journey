---
title: "Tuần 11 Worklog"
date: 2024-01-01
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---
### Mục tiêu Tuần 11:

* Thiết kế và cấu hình quy trình tích hợp liên tục tự động (CI/CD Pipeline) bằng GitLab CI.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Phân tích các cơ chế hoạt động của GitLab Runner và cấu hình runner chạy trên môi trường Docker. | 24/06/2026 | 24/06/2026 |  |
| 3 | Tạo file cấu hình `.gitlab-ci.yml` tại thư mục gốc của mã nguồn backend. | 25/06/2026 | 25/06/2026 | <https://docs.gitlab.com/ee/ci/> |
| 4 | Thiết lập stage build chạy Maven biên dịch mã nguồn và đóng gói thành file JAR. | 26/06/2026 | 26/06/2026 | <https://docs.gitlab.com/ee/ci/> |
| 5 | Thiết lập stage test tự động kích hoạt chạy toàn bộ unit tests mỗi khi có commit mới. | 29/06/2026 | 29/06/2026 |  |
| 6 | Tối ưu hóa bộ nhớ đệm (caching) và giải quyết các lỗi dependency của runner để tăng tốc độ chạy pipeline. | 30/06/2026 | 30/06/2026 |  |

### Kết quả đạt được trong Tuần 11:

* Tự động hóa hoàn toàn quy trình biên dịch mã nguồn backend đảm bảo code luôn trong trạng thái build được.
* Tự động kích hoạt chạy kiểm thử, ngăn chặn việc gộp (merge) code lỗi lên các nhánh chính.
* Cấu hình cache thư viện Maven giúp rút ngắn 50% thời gian thực thi của mỗi luồng pipeline.
