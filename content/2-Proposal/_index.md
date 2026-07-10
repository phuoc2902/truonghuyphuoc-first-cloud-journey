---
title: "Project Proposal: NovaTech E-Commerce Laptop Shop & AI Chatbot MVP"
linkTitle: "Proposal"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## Hybrid Cloud Integration (AWS Hybrid Integration) & Generative AI Solution

---

### 1. Executive Summary

The **NovaTech** project is a comprehensive electronic commerce platform specializing in laptop sales (Laptop E-Commerce Shop). To optimize initial infrastructure costs during the Minimum Viable Product (MVP) phase while maintaining high security, strict data availability, and intelligent user interaction, we propose a **Hybrid Cloud** architecture combining **Amazon Web Services (AWS)** and **DigitalOcean VPS**.

The core technical highlight is the integration of an **AI Chatbot** powered by **Amazon Bedrock**, which allows customers to automatically query specifications, compare laptops, and receive 24/7 shopping assistance based on a Retrieval-Augmented Generation (RAG) knowledge base securely stored in AWS S3.

---

### 2. Problem Statement

#### Challenges of current E-commerce systems:
- **Customer Support**: Providing 24/7 customer service requires a large support team, leading to high operational costs and inconsistent product advice.
- **Security & Data Integrity**: Databases containing user info, orders, and payment records are vulnerable if hosted directly on public web servers.
- **Infrastructure Cost**: Renting high-performance servers on major cloud providers during the MVP stage can be financially inefficient when traffic fluctuates.

#### NovaTech Proposed Solutions:
1. **Cost-Optimized Hybrid Architecture**: Use a **DigitalOcean VPS** (low fixed cost) to host the Web Server / Backend API, while deploying the critical **PostgreSQL** database inside an **AWS RDS (Multi-AZ)** Private Subnet, protecting it from the public internet.
2. **Encrypted VPN Connection**: Connect the two environments securely via an **AWS Site-to-Site VPN** tunnel.
3. **AI Chatbot Virtual Assistant**: Utilize **Amazon Bedrock** alongside **RAG (Retrieval-Augmented Generation)** to access FAQ/Context documents stored in **Amazon S3**, helping users find the right laptop model quickly.
4. **Edge Delivery**: Deploy the Next.js frontend to **Amazon S3** and distribute it globally via **Amazon CloudFront** combined with **AWS WAF** to increase page loading speed and defend against DDoS/SQL Injection attacks.

---

### 3. Solution Architecture

Below is the detailed architecture diagram for the **NovaTech** E-Commerce platform:

![NovaTech AWS Hybrid MVP Architecture](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.drawio.png)

#### Core Components:

##### A. Edge Front Door (Outside VPC)
- **Amazon Route 53**: Handles DNS resolution.
- **AWS WAF (Web Application Firewall)**: Filters malicious payloads and protects the API against OWASP Top 10 exploits.
- **Amazon CloudFront**: CDN for delivering the static web interface (Next.js SPA) and routing API requests.
- **Amazon S3 (Private SPA origin)**: Securely hosts the compiled frontend code.
- **AWS Certificate Manager (ACM)**: Manages and issues SSL/TLS certificates automatically.

##### B. Web Application Server (DigitalOcean VPS)
- **Nginx Reverse Proxy**: Receives traffic from CloudFront and routes it to the backend application.
- **Spring Boot API (Java)**: Handles main business logic and validates Cognito JWT tokens.
- **Redis App Cache**: Caches product catalogs to reduce query latency and DB load.

##### C. Private Database Network (AWS Cloud VPC)
- **AWS Site-to-Site VPN**: Establishes an encrypted IPSec VPN tunnel between the Customer Gateway (DigitalOcean) and the Virtual Private Gateway (AWS).
- **Amazon RDS PostgreSQL (Multi-AZ)**: Hosted entirely inside a Private Subnet. The Primary instance in AZ A processes read/write operations, while the Standby instance in AZ B synchronizes data to guarantee automatic failover (High Availability).

##### D. AI Chatbot & Auxiliary Services (Regional AWS Services)
- **Amazon Cognito**: Handles user registration, sign-ins, and federated Google login.
- **Amazon Bedrock**: Runs Large Language Models (LLM) to process customer conversations.
- **Amazon S3 (RAG Context)**: Stores product specifications and FAQ guides to provide accurate context for Bedrock AI.
- **AWS Secrets Manager**: Secures database credentials and payment gateway API keys.

##### E. Security, Observability & External Services
- **Amazon CloudWatch**: Collects system logs and resource metrics.
- **AWS CloudTrail**: Records AWS API calls for security auditing.
- **Amazon SES (Simple Email Service)**: Sends account verification and order receipt emails.
- **CI/CD Pipeline**: Automates build, test, and deployment steps from GitHub/GitLab to the DO VPS via SSH.
- **PayOS Payment Gateway**: Integrates a Vietnamese payment gateway for processing order transactions.

---

### 4. Technical Implementation

#### Implementation Phases:
1. **Design & Initialization (Week 1)**:
   - Design detailed ERD database schemas for the Laptop Shop.
   - Provision AWS VPC, Private Subnets, and configure AWS Site-to-Site VPN to DigitalOcean.
2. **Network Setup & Security (Week 2)**:
   - Configure Site-to-Site VPN between the two environments.
   - Set up Route 53, WAF configuration, and integrate Amazon Cognito User Pool authentication.
3. **Backend Development & RDS Configuration (Weeks 3 - 5)**:
   - Write Spring Boot APIs for product management, shopping carts, orders, and checkout.
   - Configure Amazon RDS PostgreSQL Multi-AZ and link credentials with AWS Secrets Manager.
   - Secure endpoints using Cognito User Pool and Spring Security.
4. **Frontend Development & Chatbot Integration (Weeks 6 - 8)**:
   - Build Next.js interfaces for users and the admin dashboard.
   - Integrate Chatbot UI on the web frontend, calling Amazon Bedrock via AWS SDK for RAG chat.
   - Integrate PayOS payment gateway.
5. **Testing, CI/CD & Deployment (Week 9)**:
   - Set up automated pipelines via GitLab CI/CD.
   - Deploy Frontend to S3/CloudFront and configure Route 53 and WAF.
   - Conduct network latency tests over VPN and evaluate chatbot accuracy.

---

### 5. Roadmap & Key Milestones

| Timeline | Milestone | Deliverable |
| --- | --- | --- |
| **Week 1** | Research & Architecture Design | Hybrid Cloud architecture diagram & Database ERD schema. |
| **Week 2** | Network Setup & Security | Operational VPN connection, Route 53, and Cognito setup. |
| **Weeks 3 - 5** | Backend API & Database | Functional Spring Boot backend connected to RDS, PayOS integration. |
| **Weeks 6 - 8** | Frontend & AI Chatbot Integration | Polished Next.js frontend, AI Chatbot responding to product queries. |
| **Week 9** | Testing, CI/CD & Delivery | Production environment launched, final project handover. |

---

### 6. Budget Estimation (Expected Monthly Cost)

Here is a monthly cost breakdown for the MVP phase:

| Service | Provider | Role | Estimated Cost / Month |
| --- | --- | --- | --- |
| **Amazon RDS PostgreSQL** | AWS | Primary Database (db.t4g.micro Multi-AZ) | ~ $30.00 |
| **Amazon Cognito** | AWS | User Authentication (Free for first 50,000 MAUs) | $0.00 |
| **Amazon Bedrock** | AWS | AI Chatbot Processing (Claude 3 Haiku LLM) | ~ $5.00 (Pay-per-use token billing) |
| **Amazon CloudFront & S3** | AWS | Static Frontend hosting & RAG storage | ~ $2.00 |
| **AWS Site-to-Site VPN** | AWS | Encrypted connection | ~ $36.00 ($0.05/hour per tunnel) |
| **DigitalOcean VPS** | DigitalOcean | Backend host & Redis Cache | $12.00 (2 vCPUs, 2GB RAM, 60GB SSD) |
| **Amazon SES & WAF** | AWS | Email notifications & Edge security | ~ $5.00 |
| **Total** | | **MVP Hybrid Infrastructure Cost** | **~ $90.00 / Month** |

---

### 7. Risk Assessment & Mitigation

- **Risk 1: VPN Disconnection between VPS and RDS**
  - *Impact*: Backend unable to reach PostgreSQL database, stopping services.
  - *Mitigation*: Establish dual IPSec VPN tunnels on AWS and configure automatic failover routing at the Customer Gateway.
- **Risk 2: Spikes in Amazon Bedrock API costs**
  - *Impact*: Malicious chatbot spam leading to high AWS bills.
  - *Mitigation*: Implement rate limiting on API Gateway / WAF and set maximum chat message quotas per user session.
- **Risk 3: Latency over the VPN tunnel**
  - *Impact*: Slow page response times for product queries.
  - *Mitigation*: Utilize a local Redis cache on the DigitalOcean VPS to cache static product listings, querying RDS only for writes (e.g., checkout).

---

### 8. Expected Outcomes

- **Real-World Application**: A fully functional, production-ready hybrid cloud e-commerce site.
- **Smart Experience**: Enhanced user experience via an AI-assisted virtual chatbot capable of product consulting.
- **Enterprise-Grade Security**: Customer data is secured inside private subnet layers, minimizing data leakage risks.