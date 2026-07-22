---
title: "Deploy EC2, ALB, Target Group và kiểm thử end-to-end"
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

Đây là bước đóng gói toàn bộ tích hợp thành một release có thể chạy sau ALB. Chỉ khi Target Group Healthy và các kịch bản nghiệp vụ pass mới chuyển sang CloudFront/WAF/Monitoring.

5.9.1. Launch Template và Auto Scaling
Launch Template dùng AMI phù hợp, instance profile `delivery-ec2-role`, `delivery-ec2-sg` và root volume đủ chỗ. ASG đặt trong hai application subnets; để tiết kiệm workshop có thể min/desired/max = 1/1/1 hoặc max = 2 khi cần thử scale.
![Hinh40](/workshop-website/images/5-Workshop/image40.png)
![Hinh43](/workshop-website/images/5-Workshop/image43.png)
5.9.2. Target Group và ALB

| Thành phần | Cấu hình |
|---|---|
| Target Group | HTTP:5000; health path `/health`; success code 200. |
| ALB | Internet-facing; public-a và public-b; `delivery-alb-sg`. |
| ASG | app-a và app-b; attach `delivery-app-tg`. |
| EC2 service | `delivery.service`; executable `/opt/delivery/current/WedNightFury`. |
![Hinh41](/workshop-website/images/5-Workshop/image41.png)
![Hinh42](/workshop-website/images/5-Workshop/image42.png)
5.9.3. Deploy release theo symlink
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

5.9.4. Health check và rollback
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

| Tình trạng Target | Hành động đầu tiên |
|---|---|
| Unhealthy: timeout | Kiểm tra service, port 5000, SG chain và route/NACL. |
| Unhealthy: 404 | Health path sai hoặc endpoint chưa map. |
| Unhealthy: 301/302 | `/health` đang redirect HTTPS/login; tách endpoint anonymous. |
| Unhealthy: 500 | Xem `journalctl`; health endpoint không nên phụ thuộc dependency ngoài. |

5.9.5. Kiểm thử end-to-end

| Kịch bản | Bằng chứng đạt |
|---|---|
| Đăng nhập | Không HTTP 500; session/cookie hoạt động qua ALB |
| Tạo đơn | Order có địa chỉ và Lat/Lng từ Amazon Location |
| Giao thành công | POD và chữ ký có object key trong DB, object tồn tại trong S3 |
| Giao thất bại | Evidence nằm dưới `failed-evidence/{orderId}/` |
| Sự kiện đơn hàng | SQS nhận đúng eventType/orderId từ thao tác thật |
| Email | Lambda log thành công; SES ghi nhận; message bị xóa |
| Health | Local `/health` HTTP 200 và Target Group `Healthy` |

Cổng qua bước 5.10: Chỉ bắt đầu CloudFront/WAF/Monitoring nâng cao khi: service ổn định, Target Group Healthy, RDS/S3/SQS/Lambda/SES/Location có smoke test, và có rollback release.