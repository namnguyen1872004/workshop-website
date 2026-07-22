---
title: "Điều kiện tiên quyết và quy tắc đặt tên"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Mục tiêu
Chuẩn bị đúng account, region, công cụ, source code và quy ước tài nguyên để các bước triển khai có thể lặp lại, bàn giao và dọn dẹp an toàn.

AWS account và region
Sử dụng đúng AWS account dành cho workshop và xác nhận region `ap-southeast-1` trước khi tạo tài nguyên regional.
Quyền cần thiết: VPC, EC2, ELB, Auto Scaling, RDS, Secrets Manager, S3, SQS, Lambda, SES, IAM, KMS, CloudWatch và Systems Manager.
Không dùng root user cho thao tác hằng ngày. Tài khoản triển khai phải có MFA nếu chính sách lớp học/account cho phép.

Công cụ local và code baseline

| Công cụ/điều kiện | Yêu cầu |
|---|---|
| .NET SDK | .NET 9; `dotnet restore`, `dotnet build` và test quan trọng phải pass. |
| AWS CLI | AWS CLI v2; không lưu access key trong repository. |
| MySQL client | Dùng kiểm tra schema, backup và dữ liệu sau migration. |
| PowerShell / Bash | Chạy script publish, upload artifact và smoke test. |
| Git | Release phải gắn commit/tag để truy vết và rollback. |
| EC2 | SSM Agent hoạt động; instance profile `delivery-ec2-role` được gắn. |

Quy tắc đặt tên

| Loại tài nguyên | Tên đề xuất |
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

Placeholder và an toàn secret
Dùng `<ACCOUNT_ID>`, `<SECRET_ARN>`, `<KMS_KEY_ARN>`, `<ALB_DNS>` và `<BUCKET_NAME>` trong tài liệu public.
Không đưa AWS access key, secret key, DB password, private key, cookie, token hoặc secret value vào ảnh chụp và source code.
File `/etc/delivery/delivery.env` chỉ chứa secret identifier và cấu hình không nhạy cảm; database password được lấy ở runtime.
Không chụp tab Secret value của Secrets Manager. Chỉ chụp metadata và ARN đã che.

Mốc kiểm chứng và cách ghi trạng thái
Mốc dữ liệu hiện có sau lần phục hồi gần nhất là 3 users, 5 hubs và 13 orders; migration history gồm `InitialCreate`, `AddVnpayOrderDrafts` và `SyncLatestModel`. Đây là mốc kiểm thử, không phải seed bắt buộc.

| Nhãn | Ý nghĩa |
|---|---|
| DONE | Đã có cấu hình thực tế, kiểm thử pass và evidence. |
| IN PROGRESS | Đang code/cấu hình; chưa đủ bằng chứng để chốt. |
| PLANNED | Có trong kiến trúc mục tiêu nhưng chưa triển khai. |
| N/A | Không áp dụng cho lần workshop hiện tại. |

Checklist hoàn thành
- [ ] Đúng account và đúng region.
- [ ] Source build thành công trước khi tạo tài nguyên phụ thuộc.
- [ ] Tên tài nguyên thống nhất và có tag Project/Environment/Owner.
- [ ] Không có secret thật trong source, ảnh chụp hoặc tài liệu.
- [ ] Đã thiết lập ngân sách/cảnh báo chi phí phù hợp phạm vi workshop.