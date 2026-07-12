---
title: "5.8. Resource Cleanup"
date: 2026-07-06
weight: 8
---

After completing your experiments and taking screenshots for the internship report, you must clean up the infrastructure resources initialized on AWS to avoid recurring costs on your personal account.

---

#### 1. Delete Amazon RDS PostgreSQL Database
1. Go to the **RDS → Databases** console.
2. Select the `novatech-db` database instance.
3. Click the **Actions** dropdown menu on the top right and select **Delete**.
4. Uncheck the option to **Create final snapshot** and confirm deletion.
5. Enter `delete me` in the confirmation field to proceed with the deletion.

![RDS Database Deletion Confirmation](/images/5-Workshop/rds_delete_confirm.png)
![RDS Database Deleting Status](/images/5-Workshop/rds_deleting.png)


#### 2. Delete S3 Buckets
1. Open the **Amazon S3 → Buckets** console.
2. For each bucket (`novatech-product-images` and `novatech-chatbot-rag`):
   - You must empty the bucket first by selecting it and clicking **Empty**.
   - Type `permanently delete` to acknowledge emptying objects.

     ![Confirm Emptying S3 Bucket](/images/5-Workshop/s3_empty_confirm.png)
     ![S3 Bucket Emptied Successfully](/images/5-Workshop/s3_empty_success.png)

   - Return to the Buckets list, select the bucket, and click **Delete**.
   - Enter the name of the bucket to confirm permanent deletion.

     ![Confirm S3 Bucket Deletion](/images/5-Workshop/s3_delete_confirm.png)
     ![S3 Bucket Deleted Successfully](/images/5-Workshop/s3_delete_success.png)


#### 3. Delete Amazon Cognito User Pool
1. Navigate to the **Cognito → User Pools** console.
2. Select the `novatech-user-pool` instance.
3. Click **Delete user pool** in the top right corner and follow the instructions to confirm deletion.

   ![Confirm Cognito User Pool Deletion](/images/5-Workshop/cognito_delete_confirm.png)
   ![Cognito User Pool Deleted Successfully](/images/5-Workshop/cognito_delete_success.png)


#### 4. Disable and Delete CloudFront Distribution
To clean up the created CDN distribution:
1. Navigate to the **Amazon CloudFront → Distributions** console.
2. Select the distribution configured for S3.
3. Click the **Disable** button and confirm to take the distribution offline:

   ![Disable CloudFront Distribution](/images/5-Workshop/cloudfront_disable_distribution.png)

4. Once the distribution status is successfully disabled, you will see a notification indicating it can be deleted:

   ![Distribution Disabled Status](/images/5-Workshop/cloudfront_disabled_status.png)

5. Select the distribution, click **Delete**, and confirm on the popup to permanently delete it:

   ![Confirm CloudFront Distribution Deletion](/images/5-Workshop/cloudfront_delete_distribution_confirm.png)
   ![CloudFront Distribution Deleted Successfully](/images/5-Workshop/cloudfront_delete_success.png)



