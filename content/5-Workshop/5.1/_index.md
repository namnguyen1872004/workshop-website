---
title: "Introduction and Solution Architecture"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Problem Statement
NightFury Express is a delivery management system with business flows for creating orders, assigning drivers, updating statuses, recording proof of delivery, and sending notifications. When moving from a local environment to AWS, the following weaknesses need to be addressed:
The application lacks a unified runtime contract for EC2 and ALB: missing a dedicated health endpoint, dynamic ports, and incomplete forwarded headers handling.
Database connection information risks relying on local configurations; migrations can misalign the schema if run uncontrollably.
POD images, signatures, and evidence stored on the local disk will be lost when instances are replaced or Auto Scaling expands.
Sending emails directly within web requests increases response time and makes retries difficult when external services fail.
Nominatim is an external dependency not included in the target AWS architecture.
Configuring CloudFront, WAF, and monitoring too early can mask errors at the application layer, increasing debugging scope and costs.
The workshop addresses these issues by clearly separating runtime, data, media, messaging, serverless, location, edge, and monitoring layers. Each layer is considered complete only when corresponding tests and evidence are provided.

Solution Architecture
The system is deployed in a dedicated VPC across two Availability Zones. The ALB receives requests and forwards them to ASP.NET Core on EC2 port 5000. EC2 uses an IAM Role to read RDS secrets, write media to S3, send events to SQS, and call Amazon Location. Lambda receives messages from SQS and sends emails via SES.

Request and Data Flow
1. Users access the application via CloudFront/WAF in the final stage or directly via ALB during the core flow phase.
2. ALB forwards requests to the Target Group on port 5000; `/health` is used for liveness checks.
3. The application reads database credentials from Secrets Manager and connects to RDS MySQL via a Security Group chain.
4. POD, signatures, and evidence are written to S3; the database only stores object keys and metadata.
5. After the business transaction commits, the application publishes an event to SQS.
6. Lambda processes the message, ensures idempotency, and calls SES. Messages failing multiple times are sent to the DLQ.
7. Geocoding and reverse geocoding go through an internal API using Amazon Location GeoPlaces V2.

Architecture Diagram

*Figure 1. The current architecture diagram of NightFury Express is kept as the target architecture.*

How to read the diagram: The diagram remains unchanged as requested. However, the presence of CloudFront, WAF, Auto Scaling, or monitoring in the diagram does not mean these components are completed. The actual status must be cross-referenced with the status table and evidence in each section.

Key Technologies

| Component | Role in the system |
|---|---|
| ASP.NET Core / .NET 9 | Web application, REST API, business logic, and health endpoint. |
| Amazon EC2 + systemd | Application runtime; managed preferably via Systems Manager Session Manager. |
| Application Load Balancer | HTTP(S) entry point, request routing to Target Group, and health checks. |
| Amazon RDS for MySQL | Stores business data and migration history. |
| AWS Secrets Manager | Manages RDS credentials; read by the application via IAM Role. |
| Amazon S3 | Stores POD, signatures, evidence, and deployment packages. |
| Amazon SQS + DLQ | Decouples web requests from asynchronous email processing. |
| AWS Lambda + Amazon SES | Event consumer and email sender. |
| Amazon Location GeoPlaces V2 | Address search and reverse geocoding replacing Nominatim. |
| CloudFront + WAF + CloudWatch | Edge protection and observability, deployed only after the core flow. |

Sample Resource List

| Layer | Resource Name | Role |
|---|---|---|
| Network | delivery-dev-vpc | VPC 10.0.0.0/16 |
| Load balancing | delivery-alb / delivery-app-tg | Receives HTTP(S), health check `/health` on port 5000 |
| Compute | delivery-ec2-lt / delivery-ec2-asg | Launch Template, Auto Scaling, and ASP.NET Core runtime |
| Database | delivery-dev-rds | RDS MySQL 8.4 in private DB subnets |
| Storage | delivery-dev-pod-nm2026a | POD, signatures, evidence, and deployment packages |
| Messaging | delivery-order-events | Standard queue for order events |
| DLQ | delivery-order-events-dlq | Failed messages after configured retries |
| Serverless | delivery-email-worker | Lambda processing emails via SES |
| IAM | delivery-ec2-role | Runtime permissions following least privilege |

Current Deployment Status

| Category | Document Status | Presentation Principle |
|---|---|---|
| Network and security baseline | Content/evidence available | Keep current diagrams and configuration screenshots. |
| Standardize app, RDS, S3, SQS, Lambda/SES, Location | Integrating in order | Mark as complete only with smoke tests and evidence. |
| EC2 + ALB/Target Group | Final deployment stage | Target Group must be Healthy before moving to Edge. |
| CloudFront/WAF/Advanced Monitoring | Planned | Do not mark `Done` without actual configuration and screenshots. |

Optimal Integration Order

The order below is a dependency order, not just a task list. Each step creates a stable foundation for the next and reduces concurrent debugging.

| # | Stage | Reason for this position | Completion Criteria |
|---|---|---|---|
| 1 | Standardize application for EC2 | Establish a stable runtime contract: `/health`, port 5000, forwarded headers, systemd, env vars, and disable VNPay via feature flag. | Local curl returns HTTP 200; service restarts without errors; source does not rely on static credentials. |
| 2 | RDS via Secrets Manager & migration | Stabilize business data before moving media and events. Secrets are read via IAM Role, no passwords in source or public files. | Secret read successfully; migration up-to-date; verification data intact. |
| 3 | Move POD/signatures/evidence to S3 | Remove local disk dependency before scaling EC2. Database only stores object keys. | Upload/read/delete on correct prefix; private bucket; objects are encrypted. |
| 4 | Integrate SQS for order events | Finalize event contract after DB and media flows are stable; publish after business transaction commits. | A real order action creates a message with correct `eventType`, `eventId`, `orderId`. |
| 5 | Complete Lambda → SES | Deploy consumer only after event contract is stable to avoid multiple edits and sending wrong emails. | Lambda processes idempotently; successful messages deleted; failed messages go to DLQ after retries. |
| 6 | Replace Nominatim with Amazon Location | Execute before final end-to-end testing to verify addresses and coordinates on the target service. | Search and reverse geocode work via internal API; browser does not receive AWS credentials. |
| 7 | Deploy to EC2 and test Target Group | Package final release, deploy via symlink, attach ALB/Target Group, and check health. | Local `/health` HTTP 200; Target Group `Healthy`; rollback command available. |
| 8 | CloudFront, WAF, and Monitoring | Configure edge and advanced observability only after the backend is stable to avoid masking app errors. | Evidence of actual distribution/Web ACL/alarms; if not available, clearly mark `Planned`. |

Important note: Step 1 only standardizes the EC2 runtime contract, it's not the final deployment. Integrations (RDS, S3, SQS, Lambda/SES, Location) can be developed and smoke-tested locally or in staging; the complete release is only put behind the Target Group in Step 7.