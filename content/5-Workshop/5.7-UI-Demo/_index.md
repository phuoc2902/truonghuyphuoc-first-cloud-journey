---
title: "5.7. Real-world Application Demo"
date: 2026-07-06
weight: 7
---

This section details the user interfaces of the **NovaTech Laptop E-Commerce** platform and demonstrates how frontend components interact with the underlying **AWS Hybrid Cloud** infrastructure (Amazon RDS, S3, CloudFront, Cognito, Bedrock) and the **DigitalOcean VPS**.

---

#### 1. Customer Storefront Website (Next.js SPA)

##### A. Homepage Interface
The homepage is the first touchpoint for customers, designed to optimize load times by decoupling static assets from backend API processing.

*   **Infrastructure Integration:**
    *   Next.js static files (HTML, CSS, JS, Icons) are securely stored in an **Amazon S3 Bucket (Private SPA Origin)** and globally distributed using **Amazon CloudFront CDN**.
    *   Promotional banners and logos are served directly via CloudFront edge locations, ensuring page-load latencies remain under 100ms.
*   **Data Flow:** Users visit the domain → DNS is resolved via **Amazon Route 53** → Request goes through **AWS WAF** (shielding against malicious bots and SQLi) → CloudFront serves static files from the nearest edge location.

![Homepage Interface](/images/5-Workshop/demo/homepage.jpg)

---

##### B. Register & Login Pages (Cognito Authentication)
The system eliminates storing user passwords in local databases to maintain strict data compliance and security.

*   **Infrastructure Integration:**
    *   Utilizes **Amazon Cognito User Pool** as the primary Identity Provider (IdP).
    *   The Next.js frontend uses the **AWS Amplify SDK Auth** module to communicate directly with Cognito endpoints.
*   **Data Flow:**
    1.  User enters credentials → Calls Cognito User Pool signup/signin APIs.
    2.  Cognito validates password policies and sends OTPs to the user's email via **Amazon SES**.
    3.  On success, Cognito issues a **JSON Web Token (JWT)** containing *Access Token*, *Id Token*, and *Refresh Token*.
    4.  The token is stored securely in cookies or local storage and sent in the `Authorization: Bearer <Token>` header for all backend API calls.

![Register Page](/images/5-Workshop/demo/register.jpg)
![Login Page](/images/5-Workshop/demo/login.jpg)

---

##### C. User Profile Page
Allows customers to update their personal profile details and track purchase histories.

*   **Infrastructure Integration:**
    *   User records and historical orders are persisted in the **Amazon RDS PostgreSQL** instance inside the Private Subnet of the AWS VPC.
*   **Data Flow:**
    *   Client requests profile details → Nginx reverse proxy (DigitalOcean) routes request to Spring Boot backend → Backend decodes and validates the Cognito JWT token → Queries are sent securely via the **AWS Site-to-Site VPN** tunnel to **RDS PostgreSQL** → Response returns back to the client.

![User Profile](/images/5-Workshop/demo/user_profile.jpg)

---

##### D. Product Detail Page
Displays specifications (CPU, RAM, GPU, storage), pricing, and cart buttons.

*   **Infrastructure Integration:**
    *   Product metadata (text descriptions/specifications) are retrieved from **RDS PostgreSQL**.
    *   Images are stored in an **Amazon S3** bucket (`novatech-product-images`). 
    *   To prevent unauthorized image hotlinking, S3 Block Public Access is enabled. Image delivery is allowed only through **Amazon CloudFront** using **Origin Access Control (OAC)**.
*   **Data Flow:** Clicking a product prompts the frontend to fetch specs from the backend → Returns a secure CloudFront URL: `https://d1amspckjkbrez.cloudfront.net/products/filename.jpg`. CloudFront verifies the OAC request and delivers the cached asset.

![Product Detail](/images/5-Workshop/demo/product_detail.jpg)

---

##### E. Order Checkout & Payment Page (PayOS QR Code)
Automated online payments to optimize checkout conversion rates.

*   **Infrastructure Integration:**
    *   Integrates **PayOS SDK** (a Vietnamese payment gateway serving VietQR payment codes).
*   **Data Flow:**
    1.  Customer adds laptops to the cart and clicks **Pay**.
    2.  Spring Boot API creates an order record in **RDS PostgreSQL** marked as `PENDING_PAYMENT`.
    3.  Backend calls PayOS API to generate a unique checkout link with the order code and amount → Returns a dynamic VietQR payment QR Code.
    4.  Customer scans the QR code using their banking application to transfer funds.

![Checkout and Payment](/images/5-Workshop/demo/checkout_payment.jpg)

---

##### F. Order Success Page
Instantly displays the order completion screen without requiring manual receipt uploads.

*   **Infrastructure Integration:**
    *   Uses **PayOS Webhooks** targeting Spring Boot API endpoints.
*   **Data Flow:**
    1.  Upon bank transfer confirmation, PayOS posts a signed HTTP webhook to the DigitalOcean VPS.
    2.  Spring Boot validates the webhook signature → Updates the order status to `PAID` in **RDS PostgreSQL**.
    3.  The frontend (using WebSocket or polling) receives the updated state → Redirects to the success page and sends a PDF invoice via **Amazon SES**.

![Order Success](/images/5-Workshop/demo/order_success.jpg)

---

#### 2. Admin Dashboard Website

##### A. Admin Dashboard Overview
Enables administrators to monitor store sales, total orders, new users, and interactive revenue trends.

*   **Infrastructure Integration:**
    *   Aggregates data via SQL functions (`SUM`, `COUNT`) executed in **RDS PostgreSQL**.
    *   To prevent database overloading from frequent dashboard refreshes, aggregated metrics are cached in a local **Redis Cache** on the DigitalOcean VPS with a 15-minute Time-To-Live (TTL).

![Admin Dashboard](/images/5-Workshop/demo/admin_dashboard.jpg)

---

##### B. Product Management
Allows admins to register laptops, configure technical specs, and manage inventory levels.

*   **Infrastructure Integration:**
    *   Uses **AWS SDK for Java S3 Client** to upload images directly from the Admin Panel to S3.
*   **Data Flow:** Admin uploads a laptop image → Spring Boot executes a `PutObjectRequest` to `novatech-product-images` bucket → Returns the Object Key → Key and spec data are saved in the `products` table in **RDS PostgreSQL**.

![Product Management](/images/5-Workshop/demo/admin_products.jpg)

---

##### C. Order Management
Tracks order statuses (Pending, Paid, Delivered, Cancelled) and customer details.

*   **Infrastructure Integration:**
    *   The `orders` and `order_items` tables are stored in **RDS PostgreSQL** with foreign key constraints to ensure transactional integrity.

![Order Management](/images/5-Workshop/demo/admin_orders.jpg)

---

##### D. User Management
Lists customer registration details.

*   **Infrastructure Integration:**
    *   Though auth details are stored securely in **Amazon Cognito**, basic identifiers (Display name, phone, shipping address) are synchronized to the local `users` table in **RDS PostgreSQL** to associate orders.

![User Management](/images/5-Workshop/demo/admin_users.jpg)

---

##### E. Voucher Management
Allows creation of percentage-based or flat discount codes.

*   **Infrastructure Integration:**
    *   Persisted in the `vouchers` table in **RDS PostgreSQL**.
    *   Spring Boot Backend validates voucher expiration dates and remaining limits during checkout.

![Voucher Management](/images/5-Workshop/demo/admin_vouchers.jpg)

---

##### F. Reports & Sales Statistics
Generates interactive line and bar charts tracking sales performance.

*   **Infrastructure Integration:**
    *   Aggregates transaction logs from **RDS PostgreSQL** payments table and feeds JSON payloads to Chart.js frontend components.

![Reports](/images/5-Workshop/demo/admin_reports.jpg)
