---
title: "Week 10 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.10. </b> "
---


### Week 10 Objectives:
* Complete Epic 4 (Mermaid Mindmap).
* Execute Group 1 Tasks: Tech Debt Cleanup.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Develop Backend logic to generate Mermaid.js summary code <br> - Integrate `mermaid` library (^11.15.0) on Frontend | 19/06/2026 | 19/06/2026 | Mermaid.js Docs |
| 3 | - Task 1.1: Fix 404 bug on Personal Library page (`fe/app/library/page.tsx`, `fe/middleware.ts`) | 20/06/2026 | 20/06/2026 | Next.js Routing |
| 4 | - Task 1.2: Protect `GET /job/{jobId}` endpoint <br> - Apply JWT Authorizer to `be/lib/be-stack.ts` | 21/06/2026 | 21/06/2026 | AWS CDK Auth |
| 5 | - Task 1.3: Code cleanup <br>&emsp; + Remove Debug Panel in `UploadView.tsx` <br>&emsp; + Un-hardcode AWS Secrets Manager ARN | 22/06/2026 | 22/06/2026 | AWS Secrets Manager |
| 6 | - Task 1.3: Enable Cache TTL 300s for Lambda Authorizer on Production <br> - Review all remaining technical debt | 23/06/2026 | 23/06/2026 | AWS API Gateway Caching |

### Week 10 Achievements:
* Mindmap rendering via Mermaid functions properly.
* Resolved all critical technical issues (JWT security for Job endpoint, 404 error on Library page).
* The project is secure and ready for multi-environment deployment (Dev/Prod).