---
title: "5.3. Dịch vụ Lưu trữ hình ảnh Amazon S3"
date: 2026-07-06
weight: 3
---

#### 1. Bài toán nghiệp vụ & Giải pháp kiến trúc

**Bài toán:** Một trang thương mại điện tử như NovaTech yêu cầu hiển thị hàng ngàn hình ảnh sản phẩm độ phân giải cao. Nếu lưu trữ toàn bộ các tệp tĩnh (static assets) này trực tiếp trên ổ cứng của máy chủ ứng dụng (DigitalOcean Droplet), băng thông (bandwidth) tải trang sẽ bị nghẽn, làm chậm thời gian tải web (Page Load Time), ảnh hưởng lớn đến trải nghiệm người dùng (UX) và SEO. Hơn nữa, ổ cứng VPS thường có giới hạn dung lượng và đắt đỏ khi mở rộng.

**Giải pháp:** Tách rời việc lưu trữ tệp tĩnh ra khỏi máy chủ ứng dụng bằng cách sử dụng **Amazon S3 (Simple Storage Service)**. 
- S3 cung cấp khả năng lưu trữ không giới hạn (Object Storage) với chi phí cực kỳ rẻ.
- Việc tải ảnh từ Frontend sẽ được trỏ thẳng tới URL của S3, giúp máy chủ DigitalOcean hoàn toàn rảnh tay để tập trung xử lý logic nghiệp vụ và tính toán.

Dưới đây là phần **Minh chứng triển khai** cấu hình Amazon S3 và mã nguồn kết nối:

#### 2. Khởi tạo Buckets trên Amazon S3 Console
1. Trên AWS Console, tìm kiếm dịch vụ **S3** và nhấn **Create bucket**.
2. Thiết lập Bucket lưu trữ ảnh sản phẩm:
   - **Bucket name**: `novatech-product-images` (Tên bucket phải là duy nhất trên toàn cầu).
   - **AWS Region**: Chọn khu vực gần bạn (ví dụ: `ap-southeast-1` Singapore).
3. Tại mục **Object Ownership**: Chọn **ACLs enabled** (Hoặc quản lý bằng Bucket Policy để cho phép đọc công khai tệp ảnh).
4. Tại mục **Block Public Access settings for this bucket**:
   - Nếu bạn muốn hình ảnh hiển thị trực tiếp lên trình duyệt của khách hàng, bỏ tích chọn **Block all public access** (Bỏ chặn truy cập công khai).
   - *Lưu ý*: Hãy đọc kỹ và tick xác nhận cảnh báo của AWS về việc mở công khai bucket.
5. Nhấn **Create bucket**.

![Danh sách S3 Buckets](/images/5-Workshop/s3_buckets_list.png)

---

#### 3. Cài đặt thư viện AWS SDK S3 trong Backend (Spring Boot)
Thêm dependency của AWS SDK S3 vào file `pom.xml` của dự án Backend:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 4. Mã nguồn tích hợp tải ảnh lên S3
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

            // Tạo đường dẫn URL công khai để truy cập ảnh
            String url = String.format("https://%s.s3.%s.amazonaws.com/%s", bucketName, region, objectKey);

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

#### 5. Cấu hình biến môi trường kết nối
Cấu hình các tham số truy cập AWS trong file cấu hình máy chủ App Server:
- `S3_BUCKET_NAME`: `novatech-product-images`
- `AWS_REGION`: `ap-southeast-1`
- `AWS_ACCESS_KEY_ID`: *Access Key ID của tài khoản IAM được cấp quyền S3*
- `AWS_SECRET_ACCESS_KEY`: *Secret Access Key tương ứng*
