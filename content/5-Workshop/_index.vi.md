---
title: "5. Hướng dẫn Triển khai NovaTech E-Commerce MVP & AI Chatbot trên AWS"
linkTitle: "Workshop"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Tổng quan (Overview)

Phần này đóng vai trò như một **Báo cáo Trình bày Dự án (Project Presentation)**, hướng dẫn và phân tích chi tiết cách thiết lập, triển khai hạ tầng đám mây cho dự án **NovaTech E-Commerce**. Điểm nhấn cốt lõi của dự án là việc tích hợp **Trợ lý ảo AI Chatbot** dựa trên kiến trúc Hybrid Cloud (kết hợp sức mạnh của AWS và máy chủ ứng dụng DigitalOcean). 

Thay vì chỉ đưa ra các bước "nhấp chuột" đơn thuần, tài liệu này sẽ đi sâu vào việc giải thích **Lý do (Why)** và **Bài toán nghiệp vụ (Business Case)** đằng sau mỗi quyết định lựa chọn công nghệ, đồng thời cung cấp mã nguồn thực tế như một **Minh chứng triển khai (Implementation Proof)**.

#### Đối tượng hướng tới
- Các cá nhân muốn dùng dự án này làm đồ án tốt nghiệp, báo cáo bảo vệ trước hội đồng hoặc portfolio xin việc.
- Kỹ sư phần mềm muốn hiểu rõ tư duy thiết kế hệ thống (System Design) và cách giải quyết bài toán nghiệp vụ thực tế.

#### Yêu cầu chuẩn bị (Prerequisites)
- **Tài khoản AWS:** Đang trong trạng thái hoạt động (khuyến nghị sử dụng tài khoản Free Tier để tránh phát sinh chi phí).
- **Kiến thức cơ bản:** Hiểu biết cơ bản về Linux (Ubuntu), cách kết nối SSH, nền tảng Java (Spring Boot) và React (Next.js).
- **Công cụ:** Đã cài đặt sẵn một phần mềm quản trị cơ sở dữ liệu (DBeaver, pgAdmin, DataGrip, v.v.).

#### Các bước thực hiện và Phân tích hệ thống

1. [Sơ đồ kiến trúc & Bài toán nghiệp vụ](5.1-Architecture-Overview/)
2. [Cơ sở dữ liệu Amazon RDS PostgreSQL (Bảo toàn Dữ liệu)](5.2-RDS-Database/)
3. [Dịch vụ Lưu trữ hình ảnh Amazon S3 (Tối ưu Hiệu suất Web)](5.3-S3-Storage/)
4. [Xác thực người dùng với Amazon Cognito (Bảo mật Danh tính)](5.4-Cognito-Auth/)
5. [Tích hợp Trợ lý ảo AI Chatbot Amazon Bedrock (Điểm nhấn Dự án)](5.5-Bedrock-AI/)
6. [Cấu hình máy chủ DigitalOcean & Cổng thanh toán PayOS (Hoàn tất Chu trình)](5.6-Non-AWS-Services/)
7. [Demo Giao diện Ứng dụng Thực tế (Minh chứng Giao diện)](5.7-UI-Demo/)
8. [Dọn dẹp tài nguyên (Tối ưu Chi phí)](5.8-Cleanup/)