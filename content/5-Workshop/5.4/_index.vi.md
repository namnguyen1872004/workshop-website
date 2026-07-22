---
title: "Chuẩn hóa ứng dụng và Backend EC2 Runtime"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Đây là bước tích hợp đầu tiên. Mục tiêu không phải deploy bản cuối ngay lập tức mà là tạo một runtime contract ổn định để mọi tích hợp phía sau chạy giống nhau trên local, staging EC2 và Target Group.

5.4.1. Hợp đồng runtime bắt buộc

| Hạng mục | Yêu cầu |
|---|---|
| Port | Ứng dụng listen `http://0.0.0.0:5000`. |
| Liveness | `GET /health` trả HTTP 200 khi process hoạt động; không phụ thuộc RDS/S3/SQS. |
| Readiness | Khuyến nghị `GET /health/ready` kiểm tra dependency để vận hành, không dùng làm ALB liveness ban đầu. |
| Forwarded headers | Xử lý `X-Forwarded-For` và `X-Forwarded-Proto` trước authentication/redirect. |
| VNPay | Tắt bằng feature flag trong workshop; không xóa code để dễ bật lại ở workshop thanh toán. |
| Secrets | Không hard-code credential; dùng IAM Role và secret identifier. |
| Service | `delivery.service` chạy bằng user riêng `deliveryapp`, tự restart khi lỗi. |

5.4.2. Cấu hình Program.cs
```csharp
using Microsoft.AspNetCore.HttpOverrides;

var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.ForwardLimit = 1;
    // Chỉ dùng khi EC2 port 5000 bị khóa bằng SG và chỉ nhận traffic từ ALB.
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

builder.Services.AddHealthChecks();

var app = builder.Build();

app.UseForwardedHeaders();
app.MapHealthChecks("/health");

// Có thể bổ sung /health/ready với dependency checks riêng.
app.Run();
```

Thứ tự middleware: `UseForwardedHeaders()` phải chạy trước HTTPS redirection, authentication và các logic tạo URL/cookie dựa trên scheme.

5.4.3. Biến môi trường và tắt VNPay
```env
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS=http://0.0.0.0:5000
AWS_REGION=ap-southeast-1
Features__VnPayEnabled=false
Security__AllowInsecureHttpForTesting=true
```
`Security__AllowInsecureHttpForTesting=true` chỉ dùng trong giai đoạn ALB HTTP. Khi ALB/CloudFront đã có HTTPS và forwarded headers hoạt động, chuyển cookie về Secure và tắt biến thử nghiệm.

5.4.4. systemd service
```ini
[Unit]
Description=NightFury Express ASP.NET Core
After=network-online.target
Wants=network-online.target

[Service]
User=deliveryapp
Group=deliveryapp
WorkingDirectory=/opt/delivery/current
EnvironmentFile=/etc/delivery/delivery.env
ExecStart=/opt/delivery/current/WedNightFury
Restart=always
RestartSec=5
KillSignal=SIGINT
SyslogIdentifier=delivery

[Install]
WantedBy=multi-user.target
```

5.4.5. Smoke test trước tích hợp AWS
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now delivery.service
sudo systemctl status delivery.service --no-pager -l
ss -lntp | grep ':5000'
curl -i http://127.0.0.1:5000/health
sudo journalctl -u delivery.service -n 100 --no-pager
```

| Kiểm tra | Kỳ vọng |
|---|---|
| Build/publish | Không lỗi; artifact đủ dependency cho linux-x64. |
| systemd | Active (running), restart không tạo crash loop. |
| Port | Process listen trên 0.0.0.0:5000. |
| Health | HTTP 200 và body nhỏ; không redirect login. |
| VNPay | UI/endpoint thanh toán không được gọi trong workshop. |
| Credential | Không có access key trong source hoặc `/etc/delivery/delivery.env`. |

Bằng chứng cần bổ sung trước khi nộp: Ảnh `systemctl status`, `ss -lntp`, kết quả `curl /health` và trang ứng dụng khi VNPay đã tắt. Không dùng screenshot chứa secret hoặc full account ID.