---
title: "5.7. Resource Cleanup"
date: 2026-07-06
weight: 7
---

After completing your experiments and taking screenshots for the internship report, you must clean up the infrastructure resources initialized on AWS to avoid recurring costs on your personal account.

---

#### 1. Delete Amazon RDS PostgreSQL Database
1. Go to the **RDS $\rightarrow$ Databases** console.
2. Select the `novatech-db` database instance.
3. Click the **Actions** dropdown menu on the top right and select **Delete**.
4. Uncheck the option to **Create final snapshot** and confirm deletion.
5. Enter `delete me` in the confirmation field to proceed with the deletion.

![RDS Database Deletion Confirmation](/images/5-Workshop/rds_delete_confirm.png)
![RDS Database Deleting Status](/images/5-Workshop/rds_deleting.png)


#### 2. Delete S3 Buckets
1. Open the **Amazon S3 $\rightarrow$ Buckets** console.
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
1. Navigate to the **Cognito $\rightarrow$ User Pools** console.
2. Select the `novatech-user-pool` instance.
3. Click **Delete user pool** in the top right corner and follow the instructions to confirm deletion.

   ![Confirm Cognito User Pool Deletion](/images/5-Workshop/cognito_delete_confirm.png)
   ![Cognito User Pool Deleted Successfully](/images/5-Workshop/cognito_delete_success.png)

