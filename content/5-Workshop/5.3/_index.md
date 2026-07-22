---
title: "Network, IAM, and Security Baseline"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

This section retains the current network baseline of the project. The goal is to ensure the ALB → EC2 → RDS flow is correctly tiered, while EC2 calls AWS services using an IAM Role instead of static access keys.

5.3.1. VPC Overview
Create VPC `delivery-dev-vpc` with CIDR `10.0.0.0/16`, enable DNS resolution and DNS hostnames. The VPC is divided into public, private application, and private database subnets.
Success condition: VPC is in Available state; DNS resolution and DNS hostnames are both Enabled.

5.3.2. Subnets

| Subnet | Availability Zone | CIDR | Auto-assign public IPv4 |
|---|---|---|---|
| delivery-public-a | ap-southeast-1a | 10.0.1.0/24 | Enable |
| delivery-public-b | ap-southeast-1b | 10.0.2.0/24 | Enable |
| delivery-app-a | ap-southeast-1a | 10.0.11.0/24 | Disable |
| delivery-app-b | ap-southeast-1b | 10.0.12.0/24 | Disable |
| delivery-db-a | ap-southeast-1a | 10.0.21.0/24 | Disable |
| delivery-db-b | ap-southeast-1b | 10.0.22.0/24 | Disable |

Cost/HA note: The workshop uses two Availability Zones but may use only one NAT Gateway to save costs. This is a demo choice; production architectures should consider NAT Gateways per AZ to avoid cross-AZ dependencies.

5.3.3. Route Tables
Create three separate route tables: `delivery-public-rt`, `delivery-app-rt`, and `delivery-db-rt`. The database route table only has local routes; the application route table routes outbound traffic through NAT.

5.3.4. Internet Gateway
1. Create `delivery-dev-igw`.
2. Attach the Internet Gateway to `delivery-dev-vpc`.
3. Add `0.0.0.0/0 → delivery-dev-igw` in `delivery-public-rt`.

5.3.5. NAT Gateway
1. Create `delivery-nat-a` in `delivery-public-a` and allocate an Elastic IP.
2. Wait for the Available state.
3. Add `0.0.0.0/0 → NAT Gateway` in `delivery-app-rt`.
4. Do not add a NAT route to the database route table.
Check: Do not leave routes pointing to deleted NATs; after cleanup, release the Elastic IP separately.

5.3.6. Security Groups

| Security Group | Minimum Inbound | Source |
|---|---|---|
| delivery-alb-sg | HTTP 80; HTTPS 443 if TLS enabled | Internet or CloudFront based on stage |
| delivery-ec2-sg | TCP 5000 | Only from delivery-alb-sg |
| delivery-rds-sg | MySQL 3306 | Only from delivery-ec2-sg |

Security: Do not open ports 5000 or 3306 from `0.0.0.0/0`. EC2 administration prefers Session Manager over public SSH.

5.3.7. IAM Role for EC2
Create `delivery-ec2-role` with trust principal `ec2.amazonaws.com`. Attach `AmazonSSMManagedInstanceCore`; application permissions for S3, SQS, Secrets Manager, KMS, and GeoPlaces are granted via customer-managed policies restricted by ARN.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DeliveryMedia",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": [
        "arn:aws:s3:::delivery-dev-pod-<suffix>/pod/*",
        "arn:aws:s3:::delivery-dev-pod-<suffix>/failed-evidence/*"
      ]
    },
    {
      "Sid": "OrderEvents",
      "Effect": "Allow",
      "Action": ["sqs:SendMessage", "sqs:GetQueueUrl", "sqs:GetQueueAttributes"],
      "Resource": "arn:aws:sqs:ap-southeast-1:<ACCOUNT_ID>:delivery-order-events"
    },
    {
      "Sid": "GeoPlacesV2",
      "Effect": "Allow",
      "Action": [
        "geo-places:SearchText",
        "geo-places:ReverseGeocode",
        "geo-places:GetPlace",
        "geo-places:Suggest"
      ],
      "Resource": "arn:aws:geo-places:ap-southeast-1::provider/default"
    }
  ]
}
```

```bash
sudo -u deliveryapp aws sts get-caller-identity --region ap-southeast-1
sudo -u deliveryapp aws s3api head-bucket --bucket <BUCKET_NAME>
sudo -u deliveryapp aws sqs get-queue-url --queue-name delivery-order-events --region ap-southeast-1
```

Success condition: Caller identity must be assumed-role/delivery-ec2-role and the env file does not contain AWS access keys.