---
title: "Worklog Tuần 11"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---



### Mục tiêu tuần 11:
* Triển khai Epic 5 (Tác nhân nâng cao): Chế độ Khám phá & Tìm kiếm mở rộng.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Triển khai *Explore Mode (Story 5.2)*: Khởi tạo Lambda sinh bài giảng chi tiết theo chủ đề <br> - Code logic Self-Healing cho Mermaid/Math | 26/06/2026 | 26/06/2026 | Agentic Self-Correction |
| 3 | - Task 2.1 (*Story 5-3*): Lên cấu trúc AI Agent cho Scholar Search Agent <br> - Tích hợp API Semantic Scholar để tìm bài báo động | 27/06/2026 | 27/06/2026 | Semantic Scholar API |
| 4 | - Viết handler `scholar-search.ts` <br> - Cấu hình tích hợp tìm kiếm vào Tool của AI Tutor trong khung Chat | 28/06/2026 | 28/06/2026 | Function Calling |
| 5 | - Xây dựng tính năng Đối chiếu tổng hợp chéo nhiều tài liệu (Cross-paper Synthesis Chat) | 29/06/2026 | 29/06/2026 | Multi-document RAG |
| 6 | - Kiểm thử sự phối hợp của các Agent, theo dõi Log trên CloudWatch để bắt lỗi Self-Healing | 30/06/2026 | 30/06/2026 | AWS CloudWatch |

### Kết quả đạt được tuần 11:
* Chế độ Khám phá (Explore Mode) hoàn thiện, tự động sinh bài giảng có sửa lỗi cú pháp.
* AI Tutor có thể linh hoạt tìm kiếm kiến thức bên ngoài PDF (Scholar Search Agent) và tổng hợp nhiều bài báo.