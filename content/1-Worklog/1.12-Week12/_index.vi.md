---
title: "Worklog Tuần 12"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.12 </b> "
---


### Mục tiêu tuần 12:
* Hoàn tất Epic 5: Tương tác Audio và Tính năng chia sẻ.
* Nghiệm thu và bàn giao tài liệu kỹ thuật.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Task 2.2 (*Story 5-4*): Triển khai Audio Podcast <br>&emsp; + Dùng LLM sinh kịch bản hội thoại JSON 2 chuyên gia | 03/07/2026 | 03/07/2026 | LLM Prompting |
| 3 | - Tích hợp Text-to-Speech (Google Cloud TTS / OpenAI Audio) sinh file MP3 cho 2 giọng nam/nữ <br> - Lưu MP3 vào S3 | 04/07/2026 | 04/07/2026 | TTS API Docs |
| 4 | - Task 2.3 (*Story 5-5*): Tạo trang chơi trắc nghiệm công khai `fe/app/share/quiz/[quizId]` <br> - Mở quyền API read-only cho route chia sẻ | 05/07/2026 | 05/07/2026 | Next.js App Router |
| 5 | - Tối ưu UI trang chơi Quiz chia sẻ trên mobile, không yêu cầu đăng nhập | 06/07/2026 | 06/07/2026 | Tailwind CSS |
| 6 | - Chạy Playwright E2E Test toàn bộ hệ thống <br> - Đóng gói dự án và hoàn thiện tài liệu Handover | 07/07/2026 | 07/07/2026 | Playwright Test |

### Kết quả đạt được tuần 12:
* Tính năng tự sinh Podcast học thuật 2 giọng đọc hoàn thiện, tạo trải nghiệm nghe thú vị.
* Người dùng đã có thể share link Quiz public cho bạn bè.
* Dự án Luminary hoàn tất xuất sắc theo thiết kế ban đầu, toàn bộ kiến trúc Serverless hoạt động ổn định trên môi trường Production.