---
title: "5.3. Configure Amazon S3 Image Storage"
date: 2026-07-06
weight: 3
---

#### 1. Business Case & Architecture Solution

**The Problem:** An E-commerce site like NovaTech requires displaying thousands of high-resolution product images. If all these static assets are stored directly on the application server's hard drive (DigitalOcean Droplet), the bandwidth will quickly bottleneck, significantly increasing Page Load Time and negatively impacting User Experience (UX) and SEO. Furthermore, VPS storage is limited and expensive to scale.

**The Solution:** Decouple static asset storage from the application server by using **Amazon S3 (Simple Storage Service)**. 
- S3 provides virtually unlimited Object Storage at a very low cost.
- Image requests from the Frontend will point directly to S3 URLs, freeing up the DigitalOcean server to entirely focus on processing business logic and computations.

Below is the **Implementation Proof** detailing Amazon S3 configuration and connection code:

#### 2. Create Buckets on Amazon S3 Console
1. In the AWS Console, search for **S3** and click **Create bucket**.
2. Set up the product image storage bucket:
   - **Bucket name**: `novatech-product-images` (Bucket names must be globally unique).
   - **AWS Region**: Select your nearest region (e.g., `ap-southeast-1` Singapore).
3. Under **Object Ownership**, choose **ACLs enabled** (Or manage access using a Bucket Policy to allow public read access).
4. Under **Block Public Access settings for this bucket**:
   - Uncheck **Block all public access** if you want images to be directly visible in the customer's browser.
   - *Note*: Ensure you acknowledge the AWS warning about making the bucket public.
5. Click **Create bucket**.

![S3 Buckets List](/images/5-Workshop/s3_buckets_list.png)

---

#### 3. Install AWS SDK S3 in Backend (Spring Boot)
Add the AWS SDK S3 dependency to the backend project's `pom.xml`:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 4. Source Code for S3 Integration
Below is the actual [CloudStorageServiceImpl.java](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/java/com/novatech/nova_tech_be/shared/services/impl/CloudStorageServiceImpl.java) source code that handles uploading and deleting product images via the API:

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

            // Execute upload to Amazon S3 via AWS SDK
            s3Client.putObject(putObjectRequest, RequestBody.fromInputStream(file.getInputStream(), file.getSize()));

            // Generate public URL to access the image
            String url = String.format("https://%s.s3.%s.amazonaws.com/%s", bucketName, region, objectKey);

            Map<String, Object> result = new HashMap<>();
            result.put("url", url);
            result.put("public_id", objectKey);
            
            return result;
        } catch (IOException e) {
            throw new RuntimeException("Error reading upload file data: " + e.getMessage(), e);
        }
    }

    @Override
    public Map delete(String publicId) {
        DeleteObjectRequest deleteObjectRequest = DeleteObjectRequest.builder()
                .bucket(bucketName)
                .key(publicId)
                .build();

        // Delete file from S3
        s3Client.deleteObject(deleteObjectRequest);

        Map<String, Object> result = new HashMap<>();
        result.put("result", "ok");
        return result;
    }
}
```

---

#### 5. Configure Connection Environment Variables
Configure AWS access parameters in the App Server configuration file:
- `S3_BUCKET_NAME`: `novatech-product-images`
- `AWS_REGION`: `ap-southeast-1`
- `AWS_ACCESS_KEY_ID`: *Access Key ID for the IAM account granted S3 permissions*
- `AWS_SECRET_ACCESS_KEY`: *The corresponding Secret Access Key*
