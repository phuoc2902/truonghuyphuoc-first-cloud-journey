---
title: "5.3. Configure Amazon S3 & CloudFront Image Storage"
date: 2026-07-06
weight: 3
---

#### 1. Business Case & Architecture Solution

**The Problem:** An E-commerce site like NovaTech requires displaying thousands of high-resolution product images. If all these static assets are stored directly on the application server's hard drive (DigitalOcean Droplet), the bandwidth will quickly bottleneck, significantly increasing Page Load Time and negatively impacting User Experience (UX) and SEO. Furthermore, VPS storage is limited and expensive to scale.

**The Solution:** Decouple static asset storage from the application server by using **Amazon S3 (Simple Storage Service)** paired with a global Content Delivery Network (CDN) **Amazon CloudFront**. 
- S3 provides virtually unlimited Object Storage at a very low cost.
- CloudFront caches images at edge locations nearest to the users for faster loading times, and uses **Origin Access Control (OAC)** to keep the S3 Bucket completely private, allowing image access only through the CloudFront CDN.

Below is the **Implementation Proof** detailing Amazon S3 configuration, CDN setup, and connection code:

---

#### 2. Create Virtual Private Cloud (VPC) & S3 Gateway Endpoint

To ensure that resources running inside the AWS private network (e.g. backend applications or internal databases) can securely communicate with Amazon S3 without traversing the public Internet, we set up an S3 Gateway Endpoint during VPC creation:
1. Use the VPC wizard on the AWS Console to create a VPC automatically.
2. The wizard will configure subnets, internet gateway, route tables, and automatically attach an **S3 Gateway Endpoint** to the internal route tables:

![VPC and S3 Endpoint Creation](/images/5-Workshop/vpc_creation_wizard.png)

---

#### 3. Create Buckets on Amazon S3 Console

1. In the AWS Console, search for **S3** and click **Create bucket**.
2. Set up the product image storage bucket:
   - **Bucket name**: `novatech-product-images` (Bucket names must be globally unique).
   - **AWS Region**: Select your nearest region (e.g., `ap-southeast-1` Singapore).
3. Under **Block Public Access settings for this bucket**: Keep **Block all public access** enabled (Ensuring security by only allowing CloudFront access).
4. Click **Create bucket**.

![S3 Buckets List](/images/5-Workshop/s3_buckets_list.png)

---

#### 4. Configure Image Delivery via Amazon CloudFront CDN & OAC

To allow users to view product images at high speeds while keeping the S3 bucket private, we create a CloudFront Distribution and configure OAC security:

1. **Set up OAC on CloudFront:**
   - In the Origin configuration screen of the CloudFront Distribution, set the S3 Bucket as the Origin domain.
   - For **Origin access**, select **Origin access control settings (recommended)**.
   - Click **Create new OAC** to generate a new control configuration.

   ![Configure CloudFront OAC](/images/5-Workshop/cloudfront_oac_config.png)

2. **Update S3 Bucket Policy:**
   - Once the Distribution is created, click **Copy policy** on the prompt banner to copy the JSON Policy.
   - Go to S3 Bucket -> **Permissions** tab -> **Bucket Policy** section, click **Edit**, and paste the copied JSON to grant read permissions (`s3:GetObject`) to the CloudFront Service Principal:

   ![S3 Bucket Policy for CloudFront](/images/5-Workshop/cloudfront_s3_policy.png)

3. **Verify Distribution Deployment:**
   - Wait for the deployment to finish (Status changes to Available). You will use this **Distribution domain name** to retrieve images:

   ![Distribution Successfully Configured](/images/5-Workshop/cloudfront_distribution_success.png)

---

#### 5. Install AWS SDK S3 in Backend (Spring Boot)

Add the AWS SDK S3 dependency to the backend project's `pom.xml`:

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <version>2.25.15</version>
</dependency>
```

---

#### 6. Source Code for S3 Integration

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

            // Generate CDN URL to access the image instead of S3 direct URL
            String url = String.format("https://d1amspckjkbrez.cloudfront.net/%s", objectKey);

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

#### 7. Configure Connection Environment Variables

Configure AWS access parameters in the App Server configuration file:
- `S3_BUCKET_NAME`: `novatech-product-images`
- `AWS_REGION`: `ap-southeast-1`
- `AWS_ACCESS_KEY_ID`: *Access Key ID for the IAM account granted S3 permissions*
- `AWS_SECRET_ACCESS_KEY`: *The corresponding Secret Access Key*
