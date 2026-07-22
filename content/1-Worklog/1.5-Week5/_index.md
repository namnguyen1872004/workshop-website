---
title: "Week 5 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.5. </b> "
---


### Week 5 Objectives:
* Complete Epic 2: Personal Library Management.
* Communicate with DynamoDB to store Metadata.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Initialize `Jobs` and `Document Metadata` tables on DynamoDB via AWS CDK (PAY_PER_REQUEST mode) | 15/05/2026 | 15/05/2026 | AWS CDK DynamoDB |
| 3 | - Build `GET /library` API to query document list by `userId` <br> - Build `GET /job/{jobId}` API | 16/05/2026 | 16/05/2026 | AWS SDK DynamoDB |
| 4 | - Configure Next.js Proxy to stream files directly from S3 (`Results Bucket`) with JWT token attached | 17/05/2026 | 17/05/2026 | Next.js API Routes |
| 5 | - Build Personal Library UI in Sidebar and list view <br> - Integrate upload time filter | 18/05/2026 | 18/05/2026 | Tailwind CSS |
| 6 | - Develop "Reprocess" feature on UI <br> - Ensure Reprocess triggers under the correct logic | 19/05/2026 | 19/05/2026 | Internal |

### Week 5 Achievements:
* Users have a Personal Library for storage, enabling high-speed retrieval via S3 Proxy.
* DynamoDB accurately stores job statuses (Pending, Processing, Completed, Failed).
* The library management interface operates seamlessly.