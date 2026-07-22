---
title: "Mạng, IAM và nền tảng bảo mật"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

Phần này giữ lại network baseline hiện tại của dự án. Mục tiêu là bảo đảm luồng ALB → EC2 → RDS đúng tầng, trong khi EC2 gọi AWS services bằng IAM Role thay vì access key tĩnh.

5.3.1. Tổng quan VPC
Tạo VPC `delivery-dev-vpc` với CIDR `10.0.0.0/16`, bật DNS resolution và DNS hostnames. VPC được chia thành public, private application và private database subnets.
![Hinh2](/workshop-website/images/5-Workshop/image2.png)
![Hinh3](/workshop-website/images/5-Workshop/image3.png)
![Hinh4](/workshop-website/images/5-Workshop/image4.png)
Điều kiện đạt: VPC ở trạng thái Available; DNS resolution và DNS hostnames đều Enabled.

5.3.2. Subnets

| Subnet | Availability Zone | CIDR | Tự gán public IPv4 |
|---|---|---|---|
| delivery-public-a | ap-southeast-1a | 10.0.1.0/24 | Bật |
| delivery-public-b | ap-southeast-1b | 10.0.2.0/24 | Bật |
| delivery-app-a | ap-southeast-1a | 10.0.11.0/24 | Tắt |
| delivery-app-b | ap-southeast-1b | 10.0.12.0/24 | Tắt |
| delivery-db-a | ap-southeast-1a | 10.0.21.0/24 | Tắt |
| delivery-db-b | ap-southeast-1b | 10.0.22.0/24 | Tắt |

![Hinh5](/workshop-website/images/5-Workshop/image5.png)
![Hinh6](/workshop-website/images/5-Workshop/image6.png)
Lưu ý chi phí/HA: Workshop dùng hai Availability Zones nhưng có thể chỉ dùng một NAT Gateway để tiết kiệm. Đây là lựa chọn demo; kiến trúc production cần cân nhắc NAT Gateway theo AZ để tránh phụ thuộc chéo AZ.

5.3.3. Route Tables
Tạo ba route table riêng: `delivery-public-rt`, `delivery-app-rt` và `delivery-db-rt`. Database route table chỉ có local route; application route table đi outbound qua NAT.
![Hinh9](/workshop-website/images/5-Workshop/image9.png)
![Hinh10](/workshop-website/images/5-Workshop/image10.png)
![Hinh11](/workshop-website/images/5-Workshop/image11.png)
![Hinh12](/workshop-website/images/5-Workshop/image12.png)
![Hinh13](/workshop-website/images/5-Workshop/image13.png)
![Hinh14](/workshop-website/images/5-Workshop/image14.png)
![Hinh15](/workshop-website/images/5-Workshop/image15.png)
![Hinh16](/workshop-website/images/5-Workshop/image16.png)
5.3.4. Internet Gateway
1. Tạo `delivery-dev-igw`.
2. Attach Internet Gateway vào `delivery-dev-vpc`.
3. Thêm `0.0.0.0/0 → delivery-dev-igw` trong `delivery-public-rt`.
![Hinh7](/workshop-website/images/5-Workshop/image7.png)
![Hinh8](/workshop-website/images/5-Workshop/image8.png)
5.3.5. NAT Gateway
1. Tạo `delivery-nat-a` trong `delivery-public-a` và cấp Elastic IP.
2. Chờ trạng thái Available.
3. Thêm `0.0.0.0/0 → NAT Gateway` trong `delivery-app-rt`.
4. Không thêm NAT route vào database route table.
Kiểm tra: Không để route trỏ vào NAT đã xóa; sau cleanup phải release Elastic IP riêng.
![Hinh17](/workshop-website/images/5-Workshop/image17.png)
5.3.6. Security Groups

| Security Group | Inbound tối thiểu | Nguồn |
|---|---|---|
| delivery-alb-sg | HTTP 80; HTTPS 443 khi bật TLS | Internet hoặc CloudFront theo giai đoạn |
| delivery-ec2-sg | TCP 5000 | Chỉ từ delivery-alb-sg |
| delivery-rds-sg | MySQL 3306 | Chỉ từ delivery-ec2-sg |
![Hinh18](/workshop-website/images/5-Workshop/image18.png)
![Hinh19](/workshop-website/images/5-Workshop/image19.png)
![Hinh20](/workshop-website/images/5-Workshop/image20.png)
Bảo mật: Không mở cổng 5000 hoặc 3306 từ `0.0.0.0/0`. Quản trị EC2 ưu tiên Session Manager thay cho SSH public.

5.3.7. IAM Role cho EC2
Tạo `delivery-ec2-role` với trust principal `ec2.amazonaws.com`. Gắn `AmazonSSMManagedInstanceCore`; quyền ứng dụng cho S3, SQS, Secrets Manager, KMS và GeoPlaces được cấp bằng customer-managed policy giới hạn theo ARN.
![Hinh21](/workshop-website/images/5-Workshop/image21.png)
![Hinh22](/workshop-website/images/5-Workshop/image22.png)
![Hinh23](/workshop-website/images/5-Workshop/image23.png)
![Hinh24](/workshop-website/images/5-Workshop/image24.png)
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

Điều kiện đạt: Caller identity phải là assumed-role/delivery-ec2-role và file env không chứa AWS access key.