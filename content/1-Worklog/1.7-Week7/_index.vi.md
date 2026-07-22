---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---



### Mục tiêu tuần 7:
* Hoàn thành Epic 3: Vector hóa Dữ liệu (RAG Ingestion) và Cấu hình Agentic RAG.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết Lambda `Ingest` (bước cuối của Step Functions) <br> - Cấu hình phân tách (chunking) Markdown theo anchor | 29/05/2026 | 29/05/2026 | Logic RAG cơ bản |
| 3 | - Kết nối `@google/generative-ai` (`gemini-embedding-001`) để lấy vector 768 chiều cho từng chunk | 30/05/2026 | 30/05/2026 | Google Gemini API |
| 4 | - Kết nối `@qdrant/js-client-rest` lưu vector lên Qdrant Cloud <br> - Đính kèm metadata (userId, jobId) | 31/05/2026 | 31/05/2026 | Qdrant Docs |
| 5 | - Xây dựng AI Tutor Panel (cột thứ 3 của Workspace) <br> - Cấu hình giao diện chat UI cơ bản | 01/06/2026 | 01/06/2026 | Tailwind CSS |
| 6 | - Tích hợp Semantic Scholar API cho tab "Papers liên quan" hiển thị bài báo Open Access | 02/06/2026 | 02/06/2026 | Semantic Scholar API |

### Kết quả đạt được tuần 7:
* Hạ tầng RAG (Retrieval-Augmented Generation) hoàn chỉnh: Dữ liệu được chunking, embedding và upsert vào Vector DB Qdrant.
* Giao diện UI Workspace 3 cột hoàn tất (Thư viện - Reader - RAG Chat).
* Tính năng tra cứu tĩnh các bài báo liên quan hoạt động tốt.


