---
title: "Tuần 12 Worklog"
date: 2024-01-01
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---
### Mục tiêu Tuần 12:

* Tự động hóa đóng gói ứng dụng Docker Image và đẩy lên GitLab Container Registry.

### Các công việc thực hiện trong tuần:
| Ngày | Công việc thực hiện | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 2 | Viết tệp `Dockerfile` tối ưu hóa bằng phương pháp Multi-stage build để giảm dung lượng file chạy cuối cùng. | 01/07/2026 | 01/07/2026 | <https://docs.docker.com/develop/develop-images/multistage-build/> |
| 3 | Định cấu hình stage package sử dụng Executor Docker-in-Docker (dind) để build image từ file JAR. | 02/07/2026 | 02/07/2026 | <https://docs.gitlab.com/ee/ci/docker/using_docker_build.html> |
| 4 | Cấu hình tài khoản đăng nhập registry dưới dạng biến môi trường bảo mật trên giao diện GitLab. | 03/07/2026 | 03/07/2026 |  |
| 5 | Push Docker Image hoàn thiện lên registry và dán tag tương ứng với commit hash hiện tại. | 06/07/2026 | 06/07/2026 |  |
| 6 | Chạy nghiệm thu toàn bộ kịch bản triển khai container ứng dụng thực tế. | 07/07/2026 | 07/07/2026 |  |

### Kết quả đạt được trong Tuần 12:

* Quy trình tích hợp DevOps hoàn chỉnh: đẩy code -> tự động build -> chạy test -> build docker -> đẩy lên registry.
* Tối ưu kích thước Docker Image xuống mức tối thiểu (giảm 60% dung lượng) nhờ áp dụng multi-stage build.
* Đóng gói container tiêu chuẩn sẵn sàng tích hợp thẳng lên hệ thống CD triển khai thực tế.
