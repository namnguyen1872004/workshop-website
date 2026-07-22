---
title: "Resource Cleanup"
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

Cleanup is a mandatory part of the workshop. Before deleting resources, you must save evidence, backup data worth retaining, and explicitly document which resources were deleted, retained, or never created.

5.11.1. Safety Principles
Confirm you are in the correct AWS account and region; CloudFront/WAF resources have global or specific scopes.
Stop producers and new business actions before manipulating SQS, Lambda, and databases.
Archive screenshots, logs, migration history, queue/DLQ states, and submission data outside the resources slated for deletion.
Do not purge queues/DLQs without checking messages first; purging cannot be undone.
RDS must have a final snapshot/logical backup if there's any chance a restore is needed.
Do not force-delete secrets; use a recovery window.

5.11.2. Cleanup Order by Dependency

| Order | Resource Group | Actions |
|---|---|---|
| 0 | Freeze changes and save evidence | Stop new data creation; export logs, screenshots, queue/DLQ states; backup required submission data. |
| 1 | Edge (if created) | Disable CloudFront, wait for deployment; disassociate WAF; delete distribution/OAC per dependencies. Mark `N/A` if not deployed. |
| 2 | Compute | Set ASG desired/min to 0 or delete ASG; terminate EC2 instances only after saving logs and desired release packages. |
| 3 | Load Balancing | Delete listeners/rules, then the ALB, and finally the Target Group. |
| 4 | Serverless and Messaging | Disable SQS triggers first; delete the Lambda function; verify the DLQ; delete the main queue, then the DLQ. |
| 5 | Storage | Archive POD/evidence data; delete object versions and delete markers if present; delete the bucket when absolutely certain. |
| 6 | Database | Take a final snapshot and a logical backup; delete the RDS instance; only delete the snapshot when no restore is required. |
| 7 | Secrets | Schedule deletion after the app is stopped and the RDS snapshot is secure; use a recovery window. |
| 8 | Monitoring | Export log/alarm evidence; delete alarms, dashboards, log groups, and trails/topics if no longer used. |
| 9 | Network | Delete VPC endpoints → NAT Gateway → release EIP → route tables → subnets → SGs → detach/delete IGW → VPC. |
| 10 | IAM/KMS and Costs | Delete unused policies/roles; only schedule KMS key deletion when no dependent data remains; review Cost Explorer/Budgets. |

5.11.3. Reference Commands/Checklists
```bash
# Compute / service evidence
sudo systemctl status delivery.service --no-pager -l
sudo journalctl -u delivery.service -n 200 --no-pager

# Queue state
aws sqs get-queue-attributes --queue-url <QUEUE_URL> --attribute-names All --region ap-southeast-1

# Check for resources still incurring costs
aws ec2 describe-nat-gateways --region ap-southeast-1
aws elbv2 describe-load-balancers --region ap-southeast-1
aws rds describe-db-instances --region ap-southeast-1
```

5.11.4. Network Teardown Checklist
- [ ] VPC endpoints deleted.
- [ ] NAT Gateway in Deleted state.
- [ ] Elastic IP of the NAT released independently.
- [ ] No `0.0.0.0/0` routes pointing to deleted NATs and no blackhole routes.
- [ ] Custom route tables, subnets, and security groups have no remaining dependencies.
- [ ] Internet Gateway detached before deletion.
- [ ] VPC deleted or explicitly documented as retained.

5.11.5. Cleanup Report

| Resource | Status | Evidence/Notes |
|---|---|---|
| CloudFront/WAF | N/A / DELETED / RETAINED | |
| ASG/EC2 | N/A / DELETED / RETAINED | |
| ALB/Target Group | N/A / DELETED / RETAINED | |
| Lambda/SQS/DLQ | N/A / DELETED / RETAINED | |
| S3 | N/A / DELETED / RETAINED | Specify archived data. |
| RDS/Snapshot | N/A / DELETED / RETAINED | Specify final snapshot identifier. |
| Secrets Manager | N/A / SCHEDULED / RETAINED | Do not document secret values. |
| CloudWatch/CloudTrail | N/A / DELETED / RETAINED | |
| NAT/EIP/VPC | N/A / DELETED / RETAINED | |
| IAM/KMS | N/A / DELETED / RETAINED | |