---
title: "5.7. Dọn dẹp tài nguyên"
date: 2026-07-06
weight: 7
---

Sau khi hoàn thành thử nghiệm và chụp ảnh báo cáo, bạn cần thực hiện dọn dẹp các tài nguyên hạ tầng đã khởi tạo trên AWS để đảm bảo không phát sinh chi phí duy trì không đáng có trên tài khoản cá nhân.

---

#### 1. Xóa cơ sở dữ liệu Amazon RDS PostgreSQL
1. Truy cập trang điều khiển **RDS $\rightarrow$ Databases**.
2. Chọn Database instance `novatech-db`.
3. Nhấp chọn mục **Actions** ở góc phải và chọn **Delete**.
4. Bỏ tích chọn mục **Create final snapshot** (tạo bản sao lưu cuối cùng) và xác nhận đồng ý xóa.
5. Gõ chữ `delete me` vào ô xác nhận để tiến hành xóa database.

![Xác nhận xóa RDS Database](/images/5-Workshop/rds_delete_confirm.png)
![Trạng thái đang xóa RDS Database](/images/5-Workshop/rds_deleting.png)


#### 2. Xóa các S3 Buckets
1. Truy cập dịch vụ **Amazon S3 $\rightarrow$ Buckets**.
2. Với mỗi bucket (`novatech-product-images` và `novatech-chatbot-rag`):
   - Bạn cần nhấn chọn bucket đó và nhấn nút **Empty** để xóa sạch các tệp tin lưu trữ bên trong trước.
   - Nhập `permanently delete` để xác nhận dọn dẹp đối tượng.

     ![Xác nhận dọn dẹp S3 Bucket](/images/5-Workshop/s3_empty_confirm.png)
     ![Dọn dẹp S3 Bucket thành công](/images/5-Workshop/s3_empty_success.png)

   - Quay lại trang danh sách buckets, chọn bucket và nhấn nút **Delete**.
   - Nhập tên của bucket để xác nhận xóa vĩnh viễn.

     ![Xác nhận xóa S3 Bucket](/images/5-Workshop/s3_delete_confirm.png)
     ![Xóa S3 Bucket thành công](/images/5-Workshop/s3_delete_success.png)


#### 3. Xóa Amazon Cognito User Pool
1. Truy cập dịch vụ **Cognito $\rightarrow$ User Pools**.
2. Chọn Pool `novatech-user-pool`.
3. Chọn nút **Delete user pool** ở góc phải và làm theo hướng dẫn xác nhận xóa để hủy bỏ hệ thống định danh người dùng.

   ![Xác nhận xóa Cognito User Pool](/images/5-Workshop/cognito_delete_confirm.png)
   ![Xóa Cognito User Pool thành công](/images/5-Workshop/cognito_delete_success.png)

