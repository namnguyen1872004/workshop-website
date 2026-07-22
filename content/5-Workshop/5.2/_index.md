---
title: "Prerequisites and Naming Conventions"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Objective
Prepare the correct account, region, tools, source code, and resource conventions so that deployment steps are repeatable, hand-off ready, and safely cleanable.

AWS Account and Region
Use the correct AWS account for the workshop and confirm the region `ap-southeast-1` before creating regional resources.
Required permissions: VPC, EC2, ELB, Auto Scaling, RDS, Secrets Manager, S3, SQS, Lambda, SES, IAM, KMS, CloudWatch, and Systems Manager.
Do not use the root user for daily operations. Deployment accounts must have MFA if classroom/account policies allow.

Local Tools and Code Baseline

| Tool/Condition | Requirement |
|---|---|
| .NET SDK | .NET 9; `dotnet restore`, `dotnet build`, and critical tests must pass. |
| AWS CLI | AWS CLI v2; do not store access keys in the repository. |
| MySQL client | Used to check schemas, backups, and post-migration data. |
| PowerShell / Bash | Run publish scripts, upload artifacts, and smoke tests. |
| Git | Releases must be tagged/committed for traceability and rollback. |
| EC2 | SSM Agent active; instance profile `delivery-ec2-role` attached. |

Naming Conventions

| Resource Type | Suggested Name |
|---|---|
| VPC | delivery-dev-vpc |
| Public subnets | delivery-public-a / delivery-public-b |
| Application subnets | delivery-app-a / delivery-app-b |
| Database subnets | delivery-db-a / delivery-db-b |
| ALB / Target Group | delivery-alb / delivery-app-tg |
| Launch Template / ASG | delivery-ec2-lt / delivery-ec2-asg |
| RDS | delivery-dev-rds |
| S3 | delivery-dev-pod-<unique-suffix> |
| SQS / DLQ | delivery-order-events / delivery-order-events-dlq |
| Lambda | delivery-email-worker |
| IAM Role | delivery-ec2-role |

Placeholders and Secret Safety
Use `<ACCOUNT_ID>`, `<SECRET_ARN>`, `<KMS_KEY_ARN>`, `<ALB_DNS>`, and `<BUCKET_NAME>` in public documentation.
Do not include AWS access keys, secret keys, DB passwords, private keys, cookies, tokens, or secret values in screenshots and source code.
The file `/etc/delivery/delivery.env` only contains secret identifiers and non-sensitive configurations; database passwords are fetched at runtime.
Do not capture the Secret value tab of Secrets Manager. Only capture metadata and masked ARNs.

Verification Milestones and Status Logging
The existing data milestone after the latest restoration is 3 users, 5 hubs, and 13 orders; the migration history includes `InitialCreate`, `AddVnpayOrderDrafts`, and `SyncLatestModel`. This is a testing milestone, not a mandatory seed.

| Label | Meaning |
|---|---|
| DONE | Actual configuration, passing tests, and evidence are available. |
| IN PROGRESS | Coding/configuring; insufficient evidence to finalize. |
| PLANNED | Included in the target architecture but not yet deployed. |
| N/A | Not applicable for the current workshop iteration. |

Completion Checklist
- [ ] Correct account and region.
- [ ] Source builds successfully before creating dependent resources.
- [ ] Consistent resource names with Project/Environment/Owner tags.
- [ ] No real secrets in source, screenshots, or documentation.
- [ ] Budgets/cost alerts appropriate for the workshop scope are set up.