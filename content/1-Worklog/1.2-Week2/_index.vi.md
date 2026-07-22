---
title: "Worklog Tuần 2"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---



### Mục tiêu tuần 2:
* Triển khai Epic 1: Xây dựng giao diện tải lên tài liệu PDF.
* Tích hợp Amazon S3 để upload file.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Xây dựng UI Upload kéo thả file PDF (Frontend) <br> - Viết logic kiểm tra kích thước file (tối đa 30MB-50MB) | 24/04/2026 | 24/04/2026 | React Dropzone |
| 3 | - Cấu hình S3 Bucket (`Upload Bucket`) bằng AWS CDK <br> - Cấp quyền IAM Role cho API Gateway | 25/04/2026 | 25/04/2026 | AWS CDK S3 |
| 4 | - Viết Lambda Function sinh S3 Presigned URL <br> - Tích hợp API gọi Lambda vào Frontend | 26/04/2026 | 26/04/2026 | AWS SDK S3 Presigned |
| 5 | - Phát triển cơ chế tự động thử lại (auto-retry) trên Frontend khi upload bị đứt mạng | 27/04/2026 | 27/04/2026 | Axios Retry |
| 6 | - Kiểm thử luồng Upload PDF từ Client lên S3 Bucket <br> - Fix bug hiển thị tiến trình (progress bar) | 28/04/2026 | 28/04/2026 | Nội bộ |

### Kết quả đạt được tuần 2:
* Giao diện tải file PDF hoạt động ổn định, mượt mà.
* Người dùng có thể upload file lớn trực tiếp lên S3 thông qua Presigned URL.
* Xử lý tốt các ngoại lệ về mạng (auto-retry) và validate file (dung lượng, định dạng).


