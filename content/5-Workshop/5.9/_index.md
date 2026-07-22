---
title: "Deploy EC2, ALB, Target Group, and End-to-End Testing"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

This is the stage to package all integrations into a final release running behind the ALB. Only when the Target Group is Healthy and business scenarios pass should you proceed to CloudFront/WAF/Monitoring.

5.9.1. Launch Template and Auto Scaling
The Launch Template uses an appropriate AMI, the instance profile `delivery-ec2-role`, `delivery-ec2-sg`, and a sufficient root volume. The ASG is placed across two application subnets; to save costs in the workshop, use min/desired/max = 1/1/1 or max = 2 when scaling tests are required.

5.9.2. Target Group and ALB

| Component | Configuration |
|---|---|
| Target Group | HTTP:5000; health path `/health`; success code 200. |
| ALB | Internet-facing; public-a and public-b; `delivery-alb-sg`. |
| ASG | app-a and app-b; attach `delivery-app-tg`. |
| EC2 service | `delivery.service`; executable `/opt/delivery/current/WedNightFury`. |

5.9.3. Symlink-based Release Deployment
```bash
RELEASE=20260722-01
PACKAGE=WedNightFury-linux-x64-v9.tar.gz
BUCKET=<BUCKET_NAME>

sudo aws s3 cp s3://$BUCKET/deployments/$PACKAGE /opt/delivery/packages/$PACKAGE --region ap-southeast-1
sudo mkdir -p /opt/delivery/releases/$RELEASE
sudo tar -xzf /opt/delivery/packages/$PACKAGE -C /opt/delivery/releases/$RELEASE
sudo chown -R deliveryapp:deliveryapp /opt/delivery/releases/$RELEASE
sudo chmod +x /opt/delivery/releases/$RELEASE/WedNightFury

PREVIOUS_RELEASE=$(readlink -f /opt/delivery/current)
sudo ln -sfn /opt/delivery/releases/$RELEASE /opt/delivery/current.next
sudo mv -Tf /opt/delivery/current.next /opt/delivery/current
sudo systemctl restart delivery.service
```

5.9.4. Health Checks and Rollbacks
```bash
readlink -f /opt/delivery/current
sudo systemctl status delivery.service --no-pager -l
curl -i http://127.0.0.1:5000/health
sudo journalctl -u delivery.service --since "10 minutes ago" --no-pager

# Rollback
sudo ln -sfn "$PREVIOUS_RELEASE" /opt/delivery/current.next
sudo mv -Tf /opt/delivery/current.next /opt/delivery/current
sudo systemctl restart delivery.service
```

| Target Status | First Action |
|---|---|
| Unhealthy: timeout | Check the service, port 5000, SG chain, and route/NACL. |
| Unhealthy: 404 | Incorrect health path or endpoint not mapped. |
| Unhealthy: 301/302 | `/health` is redirecting to HTTPS/login; separate the anonymous endpoint. |
| Unhealthy: 500 | Check `journalctl`; the health endpoint should not rely on external dependencies. |

5.9.5. End-to-End Testing

| Scenario | Passing Evidence |
|---|---|
| Login | No HTTP 500; session/cookie works through the ALB |
| Create order | Order contains address and Lat/Lng from Amazon Location |
| Successful delivery | POD and signature have object keys in DB, objects exist in S3 |
| Failed delivery | Evidence is stored under `failed-evidence/{orderId}/` |
| Order event | SQS receives the correct eventType/orderId from the actual action |
| Email | Lambda logs success; SES logs the event; message is deleted |
| Health | Local `/health` is HTTP 200 and Target Group is `Healthy` |

Gate to section 5.10: Begin CloudFront/WAF/Advanced Monitoring only when: the service is stable, the Target Group is Healthy, RDS/S3/SQS/Lambda/SES/Location have smoke tests, and rollback mechanisms work.