---
title: "5.1. Architecture & Overview"
date: 2026-07-06
weight: 1
---

#### 1. Business Context

**NovaTech** is a specialized e-commerce platform selling tech products (Laptops, Accessories). The biggest pain point for customers when buying laptops online is that they **don't understand complex technical specifications** (like U vs H series CPUs, RAM bus speeds, etc.), which leads to hesitation and high bounce rates.

**The Solution:** Integrate an AI Chatbot Assistant directly into the storefront. Instead of Googling for answers, customers simply ask *"Recommend a laptop for programming under $1000"*, and the AI analyzes their needs to suggest the exact products available in the store's inventory.

#### 2. Hybrid Cloud Architecture

Below is the actual system architecture of the project, designed using a Hybrid Cloud model:

![NovaTech Hybrid Cloud Architecture](/images/2-Proposal/novatech_aws_do_mvp_with_ai_chatbot.drawio.png)

#### 3. Why Hybrid Cloud?

Instead of deploying 100% of the application on AWS (using EC2/ECS/EKS) which requires complex operational skills and high monthly maintenance costs for an MVP/startup project, NovaTech adopts a Decoupling strategy:
- **Core Application (Always-on logic):** Hosted on **DigitalOcean** virtual servers (Droplets) with a fixed, affordable cost that's easy to scale when needed.
- **Data & AI (High Availability, Security):** Core services that cannot compromise on security and stability, such as the Database (RDS), Image Storage (S3), User Identity (Cognito), and Artificial Intelligence (Bedrock), are entirely entrusted to **AWS Cloud**.

#### 4. Data Flow Analysis

The system operates through a tight integration of these services:

1. **Client (Browser/Frontend)**:
   - Users access the application through a web interface built with **Next.js**. This frontend can be deployed on platforms like Vercel or AWS Amplify.
   - For user registration and login, the browser communicates directly with **Amazon Cognito** to obtain a JSON Web Token (JWT) for secure authentication.

2. **Application Server (Backend - DigitalOcean)**:
   - Runs a **Spring Boot API** that handles the core business logic of the website (managing products, cart functionality, and processing payments via the PayOS API).
   - The Backend API uses environment variables containing AWS Access Keys to securely call AWS services.

3. **Cloud Database (Amazon RDS PostgreSQL)**:
   - Deployed within a secure AWS VPC.
   - To secure data, the database is configured to only accept incoming connections originating from the static Elastic IP of the **DigitalOcean App Server** via **Security Group** ingress rules.

4. **Object Storage (Amazon S3)**:
   - Consists of two main S3 buckets:
     - `novatech-product-images`: Used to store and serve laptop product images.
     - `novatech-chatbot-rag`: Stores knowledge base documents (e.g., `.pdf`, `.txt` files) for RAG training.

5. **Generative AI Chatbot (Amazon Bedrock)**:
   - The Spring Boot backend interacts with Amazon Bedrock using the **Claude 3 Haiku/Sonnet** model to generate responses to customer queries regarding laptop specifications, utilizing the RAG knowledge documents stored in S3.
