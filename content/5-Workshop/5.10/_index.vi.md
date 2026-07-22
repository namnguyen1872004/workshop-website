---
title: "CloudFront, WAF, Monitoring và quản trị"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

Các thành phần edge và observability được triển khai sau cùng. Cách làm này giảm số lớp trung gian khi debug core flow và tránh đánh dấu hoàn tất khi chưa có evidence thật.

5.10.1. Điều kiện bắt đầu
- [ ] ALB hoạt động ổn định và Target Group Healthy.
- [ ] Cookie/session hoạt động đúng khi request đi qua reverse proxy.
- [ ] Các API động không bị cache ngoài ý muốn.
- [ ] Đã chốt domain/certificate nếu sử dụng custom domain.
- [ ] Đã có danh sách metric và log cần theo dõi; không bật dịch vụ chỉ để có trong sơ đồ.

5.10.2. CloudFront và WAF

| Hạng mục | Cấu hình khuyến nghị |
|---|---|
| Origin | ALB DNS; HTTPS origin khi ALB có certificate. |
| Viewer protocol | Redirect HTTP to HTTPS. |
| Dynamic routes | Caching disabled hoặc cache policy phù hợp; forward cookie/query/header cần thiết. |
| Static assets | Cache dài hơn, fingerprint filename và invalidate khi release. |
| WAF | Managed rule groups, rate-based rule và scope đúng với CloudFront/ALB. |
| Evidence | Distribution status, origin, behavior, Web ACL association và request test. |

Trạng thái trình bày: Nếu chưa có distribution/Web ACL thật, ghi `PLANNED`. Sơ đồ mục tiêu không phải bằng chứng triển khai.

5.10.3. Monitoring

| Lớp | Log/Metric nên có |
|---|---|
| EC2/application | systemd log, application error, request latency, health status. |
| ALB | HTTPCode_Target_5XX_Count, TargetResponseTime, UnHealthyHostCount. |
| RDS | CPU, connections, free storage, failed connections. |
| SQS | ApproximateAgeOfOldestMessage, messages visible/not visible, DLQ count. |
| Lambda | Errors, Throttles, Duration, ConcurrentExecutions. |
| SES | Send/bounce/complaint evidence phù hợp phạm vi account. |
| Chi phí | AWS Budgets cảnh báo trước ngưỡng ngân sách workshop. |

5.10.4. Audit, alert và troubleshooting
Đẩy application log lên CloudWatch Logs nếu phạm vi cho phép.
Tạo alarm cho Target Unhealthy, ALB 5xx, SQS oldest message, DLQ message và Lambda errors.
CloudTrail/Config chỉ ghi hoàn tất khi đã bật và có evidence; không tự nhận là đã audit production.
Mỗi alarm phải có owner, ngưỡng, hành động và cách kiểm thử.

```bash
sudo journalctl -u delivery.service -f
sudo journalctl -u delivery.service --since "10 minutes ago" --no-pager | grep -Ei 'AmazonLocation|SQS|S3|RdsSecretProvider|error|fail'
```