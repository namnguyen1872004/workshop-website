---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---


### Week 6 Objectives:
* Start Epic 3: Build Translation & Math Formatting Pipeline using AWS Step Functions.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Define State Machine on AWS Step Functions using AWS CDK <br> - Set up S3 Event Notification to trigger pipeline | 22/05/2026 | 22/05/2026 | AWS Step Functions |
| 3 | - Write `Extract` Lambda: Use `pdfjs-dist` to extract text from PDF <br> - Parse basic structure | 23/05/2026 | 23/05/2026 | PDF.js Docs |
| 4 | - Write `Translate` Lambda: Integrate `groq-sdk` (Llama 3.3) for academic contextual translation <br> - Detect LaTeX | 24/05/2026 | 24/05/2026 | Groq API Docs |
| 5 | - Write `Merge` Lambda: Merge translation and source into a bilingual Markdown file <br> - Insert hidden `{#chunk-index}` anchors | 25/05/2026 | 25/05/2026 | Node.js File System |
| 6 | - Set up Retry/Catch mechanisms in Step Functions when AI APIs timeout <br> - Run end-to-end pipeline testing | 26/05/2026 | 26/05/2026 | AWS CDK Step Functions |

### Week 6 Achievements:
* Completed a robust asynchronous processing system with Step Functions.
* Uploaded documents are automatically processed through the workflow: Extract -> Translate -> Merge.
* Effectively handled third-party API downtime risks using the Retry mechanism.
