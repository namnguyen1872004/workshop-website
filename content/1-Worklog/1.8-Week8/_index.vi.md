---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---



### Mục tiêu tuần 8:
* Hoàn thành Epic 3: Trợ lý Chat thông minh (Agentic Chat Logic).

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tích hợp mô hình `gemini-2.5-flash` cho luồng xử lý câu hỏi của người dùng <br> - Thiết lập system prompt | 05/06/2026 | 05/06/2026 | Google Gemini API |
| 3 | - Phát triển Function Calling (Tool Use) cho LLM: `vectorSearch` (truy vấn Qdrant) | 06/06/2026 | 06/06/2026 | Function Calling Docs |
| 4 | - Phát triển Tool `fetchAdjacentParagraphs` chống mất ngữ cảnh <br> - Tool `readExecutiveSummary` từ DynamoDB | 07/06/2026 | 07/06/2026 | Nội bộ |
| 5 | - Viết logic tự động đính kèm liên kết trích dẫn ngược `[Đoạn X]` vào câu trả lời <br> - Cấu hình Scroll-to-view trên FE | 08/06/2026 | 08/06/2026 | DOM Manipulation |
| 6 | - Kiểm thử toàn diện chất lượng câu trả lời của AI Tutor <br> - Xử lý stream response (Server-Sent Events) | 09/06/2026 | 09/06/2026 | Web Streams API |

### Kết quả đạt được tuần 8:
* AI Tutor có khả năng đọc hiểu tài liệu sâu sắc, tự chủ động gọi các công cụ tìm kiếm ngữ cảnh.
* Trải nghiệm hỏi đáp mượt mà, phản hồi có trích dẫn `[Đoạn X]` bấm vào tự cuộn đến đúng vị trí bài báo.


