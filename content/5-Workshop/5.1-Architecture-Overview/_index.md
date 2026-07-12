---
title: "5.1. Architecture Diagram & Overview"
date: 2026-07-06
weight: 1
---

#### 1. Business Context

**NovaTech** is a specialized e-commerce platform selling technology products (laptops, accessories). The biggest pain point for customers shopping for laptops online is that **they do not understand complex technical specifications** (such as U vs H series CPUs, RAM bus speeds, etc.), leading to hesitation and high cart abandonment rates.

**The Solution:** Integrate an AI Chatbot virtual assistant directly into the store interface. Instead of googling specifications, customers can simply ask: *"Recommend a programming laptop under $1000"*, and the AI will analyze their needs and suggest matching products available in the store inventory.

---

#### 2. Hybrid Cloud Architecture

Below is the actual system architecture diagram of the project, designed using a Hybrid Cloud model combining AWS Cloud services with a DigitalOcean virtual private server:

![NovaTech Hybrid Cloud Architecture](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.svg)

---

#### 3. Core Components:

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
