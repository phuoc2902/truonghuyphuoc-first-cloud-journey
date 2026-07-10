---
title: "5. Deploying NovaTech E-Commerce MVP & AI Chatbot on AWS"
linkTitle: "Workshop"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Overview

This section serves as a **Project Presentation Report**, guiding you through the deployment and analysis of the cloud infrastructure for the **NovaTech E-Commerce** project. The core highlight of the project is the integration of an **AI Chatbot Assistant** leveraging a Hybrid Cloud architecture (combining the power of AWS and a DigitalOcean application server).

Rather than providing simple "click-by-click" instructions, this documentation dives deep into the **Business Case** and the **"Why"** behind every architectural decision. It also provides actual source code as **Implementation Proof**.

#### Target Audience
- Individuals looking to use this project for a graduation thesis, a defense presentation, or a job portfolio.
- Software engineers wanting to understand System Design thinking and how to solve real-world business problems.

#### Prerequisites
- **AWS Account:** Active (Free Tier recommended to avoid costs).
- **Basic Knowledge:** Familiarity with Linux (Ubuntu), SSH connections, Java (Spring Boot), and React (Next.js).
- **Tools:** A database management tool installed (e.g., DBeaver, pgAdmin, DataGrip).

#### Implementation & System Analysis

1. [Architecture & Business Case](5.1-Architecture-Overview/)
2. [Amazon RDS PostgreSQL (Data Integrity)](5.2-RDS-Database/)
3. [Amazon S3 Image Storage (Web Performance)](5.3-S3-Storage/)
4. [Amazon Cognito Authentication (Identity Security)](5.4-Cognito-Auth/)
5. [Amazon Bedrock AI Chatbot (Project Highlight)](5.5-Bedrock-AI/)
6. [DigitalOcean & PayOS (Completing the Flow)](5.6-Non-AWS-Services/)
7. [Resource Cleanup (Cost Optimization)](5.7-Cleanup/)