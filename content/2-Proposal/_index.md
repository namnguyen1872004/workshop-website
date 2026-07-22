---
title: "Proposal: AWS Delivery Management System"
date: 2026-07-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

In this section, the document summarizes the core contents of the "Deployment of Delivery Management System on AWS" project, which the development team plans to build and deploy. It is structured to seamlessly integrate into the workshop hands-on template.

# AWS Delivery Management System
## Multi-Tier Cloud Infrastructure Architecture & Business Process Automation on AWS

### 1. Executive Summary
The project focuses on deploying a cloud infrastructure for a Delivery Management System utilizing the AWS ecosystem. The platform is built on an ASP.NET Core application, combined with core services such as RDS MySQL, S3, SQS, Lambda, SES, and Amazon Location. The objective is to transform a set of discrete configuration records into a standardized, repeatable process: from private network preparation (VPC) and multi-layer security to auto-scaling application deployment and background event processing. This backend architecture provides a robust API system, ready to support integration with client-side mobile applications (such as Android or Flutter) in the future, creating an impressive technical highlight for in-depth interviews.

### 2. Problem Statement
*Current Problem*
Traditional delivery management systems often face significant scalability bottlenecks when order volume surges unexpectedly. Storing Proof of Delivery (POD) images and driver signatures directly on local servers, or hardcoding credentials into source code, creates severe security vulnerabilities. Furthermore, synchronously processing heavy tasks—such as sending emails or calculating location routes—degrades overall system response speeds.

*Solution*
The project addresses these issues through a decoupled and highly automated cloud architecture. Users interact with the system solely via an Application Load Balancer (ALB), while EC2 instances and RDS databases are completely isolated within private subnets without public IPs. The application securely retrieves configuration details via AWS Secrets Manager. POD images and signatures are uploaded directly to Amazon S3 through an S3 Gateway VPC Endpoint. Notification tasks are pushed into an SQS queue and asynchronously consumed by AWS Lambda to send emails via SES.

*Benefits & Return on Investment (ROI)*
The Auto Scaling Group (ASG) architecture automatically adjusts server capacity based on actual traffic, optimizing operational costs. By adhering to the principle of least privilege using IAM Roles and enforcing S3/RDS encryption via KMS, the platform ensures maximum data security while minimizing data leakage risks.

### 3. Solution Architecture
The platform is deployed in the AWS Region `ap-southeast-1` (Singapore) using a Multi-Availability Zone (Multi-AZ) distributed network design to ensure high availability.

![Architecture](/workshop-website/images/2-Proposal/Picture1.png)

*AWS Services & Core Technologies*
- *AWS IAM & Security*: Access management via IAM Roles for EC2/Lambda and credential security using AWS Secrets Manager.
- *Networking*: VPC featuring 6 subnets, Internet Gateway, NAT Gateway, and Application Load Balancer for routing HTTP traffic.
- *Compute*: EC2 (hosting the ASP.NET Core application) managed by a Launch Template and Auto Scaling Group.
- *Database*: Amazon RDS MySQL 8.4 deployed inside Private Database Subnets.
- *Storage & Messaging*: Amazon S3 (for media files and deployment packages) and Amazon SQS (Standard Queue with DLQ for event processing).
- *Serverless & External*: AWS Lambda (Email processing worker), Amazon SES (Email service), and Amazon Location GeoPlaces V2 (Geocoding search).

*Component Design*
- *Security Group Chain*: Traffic flow: Internet -> ALB (Port 80) -> EC2 (Port 5000) -> RDS (Port 3306), preventing direct external access to the database.
- *Event-Driven Workflow*: Upon committing an order to the database, the application emits a JSON event to SQS. The Lambda function retrieves the event, processes the email notification, and deletes the message from the queue only upon successful execution.

### 4. Technical Implementation
*Implementation Phases (Divided by Labs)*
The project is divided into 7 core Labs, ideal for presentation as modular topics:
1. *Lab 1 (Network)*: Establish VPC `10.0.0.0/16` with 6 subnets, public/private route tables, and a NAT Gateway.
2. *Lab 2 (Security)*: Build a Security Group chain and IAM Roles (attaching SSM, S3, SQS, and Location permissions) for EC2.
3. *Lab 3 (Storage)*: Create an S3 Bucket, set up folder prefixes for POD/Signatures, and configure an S3 Gateway Endpoint.
4. *Lab 4 (Database)*: Deploy DB Subnet Group, RDS MySQL, AWS Secrets Manager, and execute Database Migration via CLI.
5. *Lab 5 (Events)*: Initialize SQS (with DLQ), configure Lambda Worker triggers, and set up SES Identity.
6. *Lab 6 (Location)*: Transition the location search mechanism to Amazon Location GeoPlaces V2 using IAM Roles instead of static API Keys.
7. *Lab 7 (Compute)*: Package the Launch Template, attach the Target Group to ALB, and activate the Auto Scaling Group.

*Technical Guidelines*
- Never store Access Keys or sensitive credentials in source code; all service interactions must rely on execution resource IAM Roles.
- Always create Snapshots and Logical Backups prior to executing any Migration operations that alter the database schema.

### 5. Roadmap & Testing Milestones
- *Milestone 1 (Network & Database)*: Ensure EC2 successfully communicates with RDS by fetching secrets from Secrets Manager, verified via HTTP 200 responses from internal Health Checks.
- *Milestone 2 (Storage & Location)*: Users can query addresses (SearchText) to resolve coordinates (Lat/Lng), and the application successfully saves POD files to S3 without triggering `AccessDenied`.
- *Milestone 3 (Event Workflow Integration)*: Upon order creation, SQS captures events with accurate `eventType`, Lambda Worker consumes events, and SES sends emails successfully without leaving stranded messages in the Dead-Letter Queue (DLQ).
- *Milestone 4 (Platform Finalization)*: Deploy new releases using Symlinks, test Auto Scaling capabilities, and plan future enhancements such as CloudFront, WAF, and CloudWatch.

### 6. Risk Assessment
*Risk Matrix & Mitigation Plan*
- *Network Configuration Issues (Target Unhealthy Error)*: Frequently caused by unstarted applications or misconfigured Security Groups. *Mitigation*: Run local `curl` commands to inspect port 5000 and review the Target Group's health check details.
- *Lambda Worker Timeout/Errors*: Repeated SQS message retries. *Mitigation*: Set `maxReceiveCount` to 3, implement idempotent processing using `eventId` within Lambda, and ensure Lambda holds proper `sqs:DeleteMessage` permissions.
- *S3 Access Errors (AccessDenied)*: Missing Put/Delete permissions or KMS issues. *Mitigation*: Inspect EC2's active IAM Role via `aws sts get-caller-identity` to verify assigned policies.

### 7. Expected Outcomes
*Technical Improvements*: Successfully build a standardized, automated, and fault-tolerant architecture on AWS. The system eliminates risks associated with public IP assignments to internal servers or manual key handling.
*Long-Term Value*: Beyond ensuring smooth ASP.NET Core application operations (handling 13 sample orders, 5 hubs, and 3 test users), this deployment documentation serves as a practical reference framework, ready to be showcased on personal documentation sites to highlight cloud architecture expertise.