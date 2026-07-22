---
title: "Worklog Tuần 10"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---



### Mục tiêu tuần 10:
* Hoàn tất Epic 4 (Mermaid Mindmap).
* Thực thi Task Nhóm 1: Dọn dẹp nợ kỹ thuật (Tech Debt).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Phát triển logic Backend sinh mã Mermaid.js tóm tắt nội dung <br> - Tích hợp thư viện `mermaid` (^11.15.0) trên Frontend | 19/06/2026 | 19/06/2026 | Mermaid.js Docs |
| 3 | - Task 1.1: Sửa lỗi 404 trang Thư viện cá nhân (`fe/app/library/page.tsx`, `fe/middleware.ts`) | 20/06/2026 | 20/06/2026 | Next.js Routing |
| 4 | - Task 1.2: Bảo vệ Endpoint `GET /job/{jobId}` <br> - Áp dụng JWT Authorizer vào `be/lib/be-stack.ts` | 21/06/2026 | 21/06/2026 | AWS CDK Auth |
| 5 | - Task 1.3: Dọn dẹp mã nguồn <br>&emsp; + Xóa Debug Panel ở `UploadView.tsx` <br>&emsp; + Gỡ hardcode ARN AWS Secrets Manager | 22/06/2026 | 22/06/2026 | AWS Secrets Manager |
| 6 | - Task 1.3: Kích hoạt Cache TTL 300s cho Lambda Authorizer trên Production <br> - Review toàn bộ code tồn đọng | 23/06/2026 | 23/06/2026 | AWS API Gateway Caching |

### Kết quả đạt được tuần 10:
* Tính năng vẽ Mindmap bằng Mermaid hoạt động tốt.
* Giải quyết xong toàn bộ lỗi kỹ thuật nghiêm trọng (Bảo mật JWT cho Job endpoint, lỗi 404 trang Thư viện).
* Dự án sẵn sàng và an toàn để triển khai đa môi trường (Dev/Prod).