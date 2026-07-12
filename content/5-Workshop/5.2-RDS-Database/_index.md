---
title: "5.2. Configure RDS PostgreSQL Database"
date: 2026-07-06
weight: 2
---

#### 1. Business Case & Architecture Solution

**The Problem:** The core data of an E-commerce system (such as Products, Orders, and Transaction History) requires absolute Data Integrity and consistency (ACID). If the database is installed directly on the same VPS as the application, a server overload or hardware failure could lead to the loss or corruption of all transaction data.

**The Solution:** Decouple the data layer from the application layer by using the managed cloud relational database service **Amazon RDS PostgreSQL**.
- **High Availability:** AWS automatically handles backups and infrastructure management.
- **Security:** The database is placed inside a Virtual Private Cloud (VPC) and blocks all public internet access. It only opens a single connection port (Port 5432) for the application server's IP address via **Security Groups**.

Below is the **Implementation Proof** for configuring Amazon RDS PostgreSQL:

#### 2. Create DB Instance on Amazon RDS
1. In the AWS Console, search for **RDS** and click **Create database**.
2. Choose the **Standard create** database creation method.
3. Under Engine options, select **PostgreSQL**.
4. Choose the appropriate Template (select **Free Tier** if you are experimenting with a personal account to avoid costs).
5. Enter the following details:
   - **DB instance identifier**: `novatech-db`
   - **Master username**: `postgres`
   - **Master password**: *Choose a secure password*
6. In the **Connectivity** section:
   - Select the target VPC.
   - For **Public access**, select **No** (Only allow internal access and traffic originating from specified Security Groups for security).
7. Click **Create database** and wait for the status to change to Available. Once created, you will see the Endpoint & Port details:

![RDS Endpoint Details](/images/5-Workshop/rds_endpoint_details.png)

---

#### 3. Configure Database Security Group
To ensure safety, we only allow the Spring Boot application server to connect to the RDS Database.
1. Go to the **Security Groups** configuration for the newly created database.
2. Select **Inbound rules** and add a new rule:
   - **Type**: `PostgreSQL` (Port 5432)
   - **Source**: Select the Private IP address of the App Server routing through the **AWS Site-to-Site VPN** (or choose the public static IP of the DigitalOcean server if connecting directly outside the private network).

![RDS Security Group Configuration](/images/5-Workshop/rds_security_groups.png)
![Security Group Rule Details](/images/5-Workshop/novatech_db_security_group.png)

---

#### 4. Configure Backend Source Code for Database Connection
In the Spring Boot project source code, instead of hardcoding database credentials in properties, environment variables are used to securely inject connection details.

Here is the database configuration block in [application.yml](file:///Users/tranthikieuoanh/Documents/FCJ-Workshop/nova-tech/nova-tech-be/src/main/resources/application.yml):

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
```

**Environment Variables configured on the App Server:**
- `DB_URL`: Formatted as `jdbc:postgresql://<RDS-Endpoint>:5432/<Database-Name>`
- `DB_USERNAME`: Database master username.
- `DB_PASSWORD`: Database connection password.
