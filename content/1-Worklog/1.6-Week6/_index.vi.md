---
title: "Worklog Tuần 6"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---



### Mục tiêu tuần 6:
* Bắt đầu Epic 3: Xây dựng Pipeline Dịch thuật & Định dạng Toán bằng AWS Step Functions.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Định nghĩa State Machine trên AWS Step Functions bằng AWS CDK <br> - Thiết lập S3 Event Notification để trigger pipeline | 22/05/2026 | 22/05/2026 | AWS Step Functions |
| 3 | - Viết Lambda `Extract`: Sử dụng `pdfjs-dist` trích xuất văn bản từ PDF <br> - Bóc tách cấu trúc cơ bản | 23/05/2026 | 23/05/2026 | PDF.js Docs |
| 4 | - Viết Lambda `Translate`: Tích hợp `groq-sdk` (Llama 3.3) để dịch thuật ngữ cảnh học thuật <br> - Nhận diện LaTeX | 24/05/2026 | 24/05/2026 | Groq API Docs |
| 5 | - Viết Lambda `Merge`: Gộp bản dịch và bản gốc thành file Markdown song ngữ <br> - Chèn anchor ẩn `{#chunk-index}` | 25/05/2026 | 25/05/2026 | Node.js File System |
| 6 | - Thiết lập cơ chế Retry/Catch trong Step Functions khi API AI bị timeout <br> - Chạy kiểm thử toàn bộ luồng pipeline | 26/05/2026 | 26/05/2026 | AWS CDK Step Functions |

### Kết quả đạt được tuần 6:
* Hệ thống xử lý bất đồng bộ mạnh mẽ với Step Functions hoàn thiện.
* Tài liệu upload tự động chạy qua quy trình: Extract -> Translate -> Merge.
* Xử lý tốt các rủi ro sập API của Third-party nhờ cơ chế Retry.