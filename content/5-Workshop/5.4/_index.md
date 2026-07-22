---
title: "Application Standardization and EC2 Backend Runtime"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

This is the first integration step. The goal is not to deploy the final version immediately but to create a stable runtime contract so all subsequent integrations behave exactly the same on local, staging EC2, and Target Group.

5.4.1. Mandatory Runtime Contract

| Category | Requirement |
|---|---|
| Port | The application listens on `http://0.0.0.0:5000`. |
| Liveness | `GET /health` returns HTTP 200 when the process is active; does not depend on RDS/S3/SQS. |
| Readiness | Recommended `GET /health/ready` for dependency checks for operations, do not use for initial ALB liveness. |
| Forwarded headers | Handle `X-Forwarded-For` and `X-Forwarded-Proto` before authentication/redirect. |
| VNPay | Disable via feature flag in this workshop; do not delete the code to easily re-enable it in a payment workshop. |
| Secrets | Do not hard-code credentials; use IAM Roles and secret identifiers. |
| Service | `delivery.service` runs using a dedicated `deliveryapp` user, auto-restarts on failure. |

5.4.2. Program.cs Configuration
```csharp
using Microsoft.AspNetCore.HttpOverrides;

var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<ForwardedHeadersOptions>(options =>
{
    options.ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
    options.ForwardLimit = 1;
    // Only use when EC2 port 5000 is restricted by SG and only accepts traffic from the ALB.
    options.KnownNetworks.Clear();
    options.KnownProxies.Clear();
});

builder.Services.AddHealthChecks();

var app = builder.Build();

app.UseForwardedHeaders();
app.MapHealthChecks("/health");

// You can add /health/ready with specific dependency checks.
app.Run();
```

Middleware ordering: `UseForwardedHeaders()` must run before HTTPS redirection, authentication, and logic that creates URLs/cookies based on the scheme.

5.4.3. Environment Variables and Disabling VNPay
```env
ASPNETCORE_ENVIRONMENT=Production
ASPNETCORE_URLS=http://0.0.0.0:5000
AWS_REGION=ap-southeast-1
Features__VnPayEnabled=false
Security__AllowInsecureHttpForTesting=true
```
`Security__AllowInsecureHttpForTesting=true` is only used during the ALB HTTP phase. Once ALB/CloudFront has HTTPS and forwarded headers working, switch cookies back to Secure and turn off the testing variable.

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

5.4.5. Pre-integration Smoke Test
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now delivery.service
sudo systemctl status delivery.service --no-pager -l
ss -lntp | grep ':5000'
curl -i http://127.0.0.1:5000/health
sudo journalctl -u delivery.service -n 100 --no-pager
```

| Check | Expectation |
|---|---|
| Build/publish | No errors; artifact has full dependencies for linux-x64. |
| systemd | Active (running), restart does not create a crash loop. |
| Port | Process listens on 0.0.0.0:5000. |
| Health | HTTP 200 and a small body; does not redirect to login. |
| VNPay | Payment UI/endpoints are not invoked during the workshop. |
| Credential | No access keys in the source or `/etc/delivery/delivery.env`. |

Evidence to add before submission: Screenshots of `systemctl status`, `ss -lntp`, `curl /health` results, and the application page showing VNPay disabled. Do not use screenshots containing secrets or full account IDs.