---
title: "Worklog Tuần 5"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---



### Mục tiêu tuần 5:
* Hoàn thành Epic 2: Quản lý Thư viện cá nhân (Personal Library).
* Giao tiếp với DynamoDB để lưu trữ Metadata.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khởi tạo bảng `Jobs` và `Document Metadata` trên DynamoDB bằng AWS CDK (chế độ PAY_PER_REQUEST) | 15/05/2026 | 15/05/2026 | AWS CDK DynamoDB |
| 3 | - Xây dựng API `GET /library` truy vấn danh sách tài liệu theo `userId` <br> - Xây dựng API `GET /job/{jobId}` | 16/05/2026 | 16/05/2026 | AWS SDK DynamoDB |
| 4 | - Cấu hình Next.js Proxy để Stream file trực tiếp từ S3 (`Results Bucket`) đính kèm JWT token | 17/05/2026 | 17/05/2026 | Next.js API Routes |
| 5 | - Xây dựng UI Thư viện cá nhân ở Sidebar và trang danh sách <br> - Tích hợp bộ lọc thời gian tải | 18/05/2026 | 18/05/2026 | Tailwind CSS |
| 6 | - Phát triển tính năng "Dịch lại" (Reprocess) trên giao diện <br> - Đảm bảo Reprocess kích hoạt đúng logic | 19/05/2026 | 19/05/2026 | Nội bộ |

### Kết quả đạt được tuần 5:
* Người dùng có Thư viện lưu trữ cá nhân, truy xuất tài liệu cũ tốc độ cao nhờ S3 Proxy.
* DynamoDB lưu trữ chính xác trạng thái job (Pending, Processing, Completed, Failed).
* Giao diện quản lý thư viện hoạt động trơn tru.

