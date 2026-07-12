---
title: "5.3. Dịch vụ Lưu trữ hình ảnh Amazon S3 & CloudFront"
date: 2026-07-06
weight: 3
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán:** Một trang thương mại điện tử như NovaTech yêu cầu hiển thị hàng ngàn hình ảnh sản phẩm độ phân giải cao. Nếu lưu trữ toàn bộ các tệp tĩnh (static assets) này trực tiếp trên ổ cứng của máy chủ ứng dụng (DigitalOcean Droplet), băng thông (bandwidth) tải trang sẽ bị nghẽn, làm chậm thời gian tải web (Page Load Time), ảnh hưởng lớn đến trải nghiệm người dùng (UX) và SEO. Hơn nữa, ổ cứng VPS thường có giới hạn dung lượng và đắt đỏ khi mở rộng.

**Giải pháp:** Tách rời việc lưu trữ tệp tĩnh ra khỏi máy chủ ứng dụng bằng cách sử dụng **Amazon S3 (Simple Storage Service)** kết hợp mạng lưới phân phối nội dung toàn cầu **Amazon CloudFront (CDN)**. 
- S3 cung cấp khả năng lưu trữ không giới hạn (Object Storage) với chi phí cực kỳ rẻ.
- CloudFront giúp lưu trữ bộ nhớ đệm (caching) hình ảnh tại các Edge Location gần người dùng nhất để tăng tốc độ tải, đồng thời sử dụng **Origin Access Control (OAC)** để giữ cho S3 Bucket luôn ở chế độ riêng tư (Private), chỉ cho phép truy cập hình ảnh thông qua CloudFront CDN.

Dưới đây là phần **Minh chứng triển khai** cấu hình hạ tầng lưu trữ và phân phối hình ảnh:

---

#### 2. Khởi tạo Mạng riêng ảo (VPC) và S3 Gateway Endpoint

Để đảm bảo các tài nguyên chạy trong mạng nội bộ AWS (ví dụ: các ứng dụng hoặc cơ sở dữ liệu nội bộ) có thể giao tiếp bảo mật với Amazon S3 mà không cần đi qua môi trường Internet công cộng, chúng ta thiết lập một S3 Gateway Endpoint ngay khi tạo VPC:
1. Sử dụng trình hướng dẫn khởi tạo VPC trên AWS Console để tạo VPC tự động.
2. Trình khởi tạo sẽ thiết lập các Subnet, Internet Gateway, Bảng định tuyến (Route tables) và tự động tạo/liên kết một **S3 Gateway Endpoint** cho bảng định tuyến nội bộ:

![Khởi tạo VPC và S3 Endpoint](/images/5-Workshop/vpc_creation_wizard.png)

---

#### 3. Khởi tạo Buckets trên Amazon S3 Console

1. Trên AWS Console, tìm kiếm dịch vụ **S3** và nhấn **Create bucket**.
2. Thiết lập Bucket lưu trữ ảnh sản phẩm:
   - **Bucket name**: `novatech-product-images` (Tên bucket phải là duy nhất trên toàn cầu).
   - **AWS Region**: Chọn khu vực gần bạn (ví dụ: `ap-southeast-1` Singapore).
3. Tại mục **Block Public Access settings for this bucket**: Giữ nguyên trạng thái **Block all public access** (Bảo mật tuyệt đối, chỉ cho phép CloudFront truy cập).
4. Nhấn **Create bucket**.

![Danh sách S3 Buckets](/images/5-Workshop/s3_buckets_list.png)

---

#### 4. Cấu hình phân phối hình ảnh qua Amazon CloudFront CDN & OAC

Để đảm bảo S3 Bucket không công khai nhưng người dùng vẫn có thể xem được hình ảnh sản phẩm tốc độ cao, chúng ta tạo một CloudFront Distribution và cấu hình cơ chế bảo mật OAC:

1. **Thiết lập OAC trên CloudFront:**
   - Tại màn hình cấu hình Origin của CloudFront Distribution, chọn S3 Bucket làm Origin domain.
   - Tại phần **Origin access**: Chọn **Origin access control settings (recommended)**.
   - Nhấn **Create new OAC** để tạo một cấu hình OAC mới.

   ![Cấu hình CloudFront OAC](/images/5-Workshop/cloudfront_oac_config.png)

2. **Cập nhật Bucket Policy cho S3:**
   - Sau khi tạo Distribution, nhấn **Copy policy** ở thanh thông báo hướng dẫn để sao chép mã chính sách (JSON Policy).
   - Đi tới S3 Bucket -> tab **Permissions** -> mục **Bucket Policy** và chọn **Edit**, sau đó dán đoạn JSON đã sao chép để cấp quyền đọc (`s3:GetObject`) cho CloudFront Service Principal:

   ![S3 Bucket Policy cho CloudFront](/images/5-Workshop/cloudfront_s3_policy.png)

3. **Xác nhận cấu hình Distribution thành công:**
   - Đợi quá trình deploy hoàn thành (Trạng thái chuyển sang Available). Bạn sẽ sử dụng **Distribution domain name** này để truy cập tài nguyên:

   ![Cấu hình Distribution thành công](/images/5-Workshop/cloudfront_distribution_success.png)

---

#### 5. Cài đặt thư viện AWS SDK S3 trong Backend (Spring Boot)

Thêm dependency của AWS SDK S3 vào file `pom.xml` của dự án Backend:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 6. Mã nguồn tích hợp tải ảnh lên S3

Dưới đây là mã nguồn thực tế lớp [CloudStorageServiceImpl.java](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/java/com/novatech/nova_tech_be/shared/services/impl/CloudStorageServiceImpl.java) xử lý nghiệp vụ upload/delete ảnh sản phẩm từ Client thông qua API:

```java
package com.novatech.nova_tech_be.shared.services.impl;

import com.novatech.nova_tech_be.shared.services.ICloudStorageService;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.DeleteObjectRequest;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

@Service
public class CloudStorageServiceImpl implements ICloudStorageService {

    private final S3Client s3Client;

    @Value("${S3_BUCKET_NAME}")
    private String bucketName;

    @Value("${AWS_REGION}")
    private String region;

    public CloudStorageServiceImpl(S3Client s3Client) {
        this.s3Client = s3Client;
    }

    @Override
    public Map upload(MultipartFile file, String folder) {
        try {
            String fileName = file.getOriginalFilename();
            String objectKey = folder + "/" + UUID.randomUUID().toString() + "-" + fileName;

            PutObjectRequest putObjectRequest = PutObjectRequest.builder()
                    .bucket(bucketName)
                    .key(objectKey)
                    .contentType(file.getContentType())
                    .build();

            // Thực hiện tải file lên Amazon S3 qua AWS SDK
            s3Client.putObject(putObjectRequest, RequestBody.fromInputStream(file.getInputStream(), file.getSize()));

            // Tạo đường dẫn URL thông qua CloudFront CDN thay vì link S3 trực tiếp
            String url = String.format("https://d1amspckjkbrez.cloudfront.net/%s", objectKey);

            Map<String, Object> result = new HashMap<>();
            result.put("url", url);
            result.put("public_id", objectKey);
            
            return result;
        } catch (IOException e) {
            throw new RuntimeException("Lỗi khi đọc dữ liệu file upload: " + e.getMessage(), e);
        }
    }

    @Override
    public Map delete(String publicId) {
        DeleteObjectRequest deleteObjectRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(publicId)
                .build();

        // Xóa file khỏi S3
        s3Client.deleteObject(deleteObjectRequest);

        Map<String, Object> result = new HashMap<>();
        result.put("result", "ok");
        return result;
    }
}
```

---

#### 7. Cấu hình biến môi trường kết nối

Cấu hình các tham số truy cập AWS trong file cấu hình máy chủ App Server:
- `S3_BUCKET_NAME`: `novatech-product-images`
- `AWS_REGION`: `ap-southeast-1`
- `AWS_ACCESS_KEY_ID`: *Access Key ID của tài khoản IAM được cấp quyền S3*
- `AWS_SECRET_ACCESS_KEY`: *Secret Access Key tương ứng*
