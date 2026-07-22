---
title: "CloudFront, WAF, Monitoring, and Governance"
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

Edge components and observability are deployed last. This approach reduces intermediary layers when debugging the core flow and prevents marking tasks complete without real evidence.

5.10.1. Prerequisites to Start
- [ ] ALB is stable and the Target Group is Healthy.
- [ ] Cookies/sessions behave correctly when requests pass through the reverse proxy.
- [ ] Dynamic APIs are not cached unintentionally.
- [ ] Domains/certificates are finalized if using a custom domain.
- [ ] You have a list of metrics and logs to monitor; do not enable services just to match the diagram.

5.10.2. CloudFront and WAF

| Category | Recommended Configuration |
|---|---|
| Origin | ALB DNS; HTTPS origin if ALB has a certificate. |
| Viewer protocol | Redirect HTTP to HTTPS. |
| Dynamic routes | Caching disabled or proper cache policy applied; forward necessary cookies/queries/headers. |
| Static assets | Longer cache durations, fingerprint filenames, and invalidate on release. |
| WAF | Managed rule groups, rate-based rules, and scope matching CloudFront/ALB. |
| Evidence | Distribution status, origin, behaviors, Web ACL associations, and test requests. |

Presentation status: If there is no real distribution/Web ACL, mark it as `PLANNED`. A target architecture diagram does not count as deployment evidence.

5.10.3. Monitoring

| Layer | Recommended Logs/Metrics |
|---|---|
| EC2/application | systemd logs, application errors, request latency, health status. |
| ALB | HTTPCode_Target_5XX_Count, TargetResponseTime, UnHealthyHostCount. |
| RDS | CPU, connections, free storage, failed connections. |
| SQS | ApproximateAgeOfOldestMessage, messages visible/not visible, DLQ count. |
| Lambda | Errors, Throttles, Duration, ConcurrentExecutions. |
| SES | Send/bounce/complaint evidence appropriate for the account scope. |
| Costs | AWS Budgets alert before the workshop budget threshold is reached. |

5.10.4. Auditing, Alerting, and Troubleshooting
Forward application logs to CloudWatch Logs if the scope permits.
Create alarms for Target Unhealthy, ALB 5xx, SQS oldest message, DLQ messages, and Lambda errors.
CloudTrail/Config should only be marked complete when actually enabled with evidence; do not pretend production auditing is in place.
Each alarm must have an owner, threshold, action, and a way to test it.

```bash
sudo journalctl -u delivery.service -f
sudo journalctl -u delivery.service --since "10 minutes ago" --no-pager | grep -Ei 'AmazonLocation|SQS|S3|RdsSecretProvider|error|fail'
```