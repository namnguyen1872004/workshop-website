---
title: "RDS MySQL, Secrets Manager và migration"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Database được tích hợp ngay sau khi runtime ổn định vì toàn bộ nghiệp vụ, S3 object key và event metadata đều phụ thuộc vào dữ liệu chính xác.

5.5.1. DB subnet group và RDS
Tạo `delivery-dev-db-subnet-group` từ hai database subnets ở hai Availability Zones. RDS MySQL 8.4 dùng private access, Single-AZ cho workshop và `delivery-rds-sg`.
![Hinh28](/workshop-website/images/5-Workshop/image28.png)
![Hinh29](/workshop-website/images/5-Workshop/image29.png)
![Hinh30](/workshop-website/images/5-Workshop/image30.png)
Phạm vi: Single-AZ là lựa chọn giảm chi phí cho workshop; không trình bày như Multi-AZ production nếu chưa bật thật.

5.5.2. Secret contract
```json
{
  "engine": "mysql",
  "host": "<RDS_ENDPOINT>",
  "port": 3306,
  "username": "<DB_USER>",
  "password": "<SECRET_VALUE>",
  "dbname": "nightfury"
}
```
```env
Database__RdsSecretId=<RDS_SECRET_ARN>
Database__Name=nightfury
AWS_REGION=ap-southeast-1
```
Ứng dụng dùng default AWS credential provider chain. IAM Role cần `secretsmanager:GetSecretValue` và `DescribeSecret` trên đúng secret ARN; nếu secret dùng customer-managed KMS key thì bổ sung `kms:Decrypt` trên đúng key.

5.5.3. Quy trình migration an toàn
1. Tạo manual snapshot và logical backup trước thay đổi schema có rủi ro.
2. Đối chiếu model, migration files và bảng `__EFMigrationsHistory`.
3. Chỉ tạo migration mới khi model thật sự thay đổi; không tạo migration để sửa lỗi endpoint hoặc credential.
4. Publish đúng runtime, nạp `/etc/delivery/delivery.env` và chạy migration bằng user `deliveryapp`.
5. Kiểm tra migration history, số lượng dữ liệu và các truy vấn nghiệp vụ chính.

```bash
sudo -u deliveryapp bash -c 'set -a; . /etc/delivery/delivery.env; set +a; cd /opt/delivery/current; ./WedNightFury --migrate'
```

5.5.4. Kiểm thử và rollback
```bash
sudo journalctl -u delivery.service --since "10 minutes ago" --no-pager | grep -Ei 'RdsSecretProvider|migrat|fail|error'
mysql -h <RDS_ENDPOINT> -u <DB_USER> -p -e "USE nightfury; SELECT * FROM __EFMigrationsHistory;"
```

| Điểm kiểm chứng | Kỳ vọng |
|---|---|
| Secret | Ứng dụng đọc secret thành công bằng IAM Role. |
| Network | RDS Public access = No; 3306 chỉ từ EC2 SG. |
| Migration | Schema up-to-date; không duplicate column/table. |
| Dữ liệu | Mốc 3 users, 5 hubs, 13 orders còn nguyên nếu dùng đúng DB. |
| Rollback | Có snapshot/backup và biết release/migration cần quay lại. |