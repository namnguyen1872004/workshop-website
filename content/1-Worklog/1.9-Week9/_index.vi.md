---
title: "Worklog Tuần 9"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---



### Mục tiêu tuần 9:
* Triển khai Epic 4: Các công cụ hỗ trợ tự học chủ động (Active Learning Tools).
* Xây dựng mô hình Async Worker Pattern cho Backend.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thiết lập mô hình Async Worker: <br>&emsp; + API Handler ghi DynamoDB trả jobId <br>&emsp; + Lambda kích hoạt bất đồng bộ (`Event`) | 12/06/2026 | 12/06/2026 | AWS Lambda Invocation |
| 3 | - Xây dựng cơ chế Polling `GET /job/{jobId}` liên tục trên Frontend để cập nhật trạng thái | 13/06/2026 | 13/06/2026 | React Query / SWR |
| 4 | - Phát triển AI Quiz Generator Lambda <br> - Thiết kế UI modal trắc nghiệm tương tác trên Web | 14/06/2026 | 14/06/2026 | Prompt Engineering |
| 5 | - Phát triển AI Flashcard Generator Lambda <br> - Thiết kế UI thẻ ghi nhớ dạng lật (Swiper UI) | 15/06/2026 | 15/06/2026 | Swiper.js |
| 6 | - Kiểm thử tích hợp luồng sinh nội dung Quiz và Flashcard <br> - Đảm bảo DynamoDB lưu đúng cấu trúc JSON trả về | 16/06/2026 | 16/06/2026 | Nội bộ |

### Kết quả đạt được tuần 9:
* Áp dụng thành công Async Worker, xử lý triệt để lỗi API Gateway Timeout 29s cho các tác vụ AI nặng.
* Người dùng có thể tự tạo bộ câu hỏi trắc nghiệm và thẻ từ vựng trực tiếp từ bài báo khoa học.

