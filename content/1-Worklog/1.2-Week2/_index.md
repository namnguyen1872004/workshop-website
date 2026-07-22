---
title: "Week 2 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---


### Week 2 Objectives:
* Implement Epic 1: Build the PDF document upload interface.
* Integrate Amazon S3 for file uploads.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Build drag-and-drop PDF upload UI (Frontend) <br> - Write logic to validate file size (max 30MB-50MB) | 24/04/2026 | 24/04/2026 | React Dropzone |
| 3 | - Configure S3 Bucket (`Upload Bucket`) using AWS CDK <br> - Grant IAM Role permissions for API Gateway | 25/04/2026 | 25/04/2026 | AWS CDK S3 |
| 4 | - Write Lambda Function to generate S3 Presigned URLs <br> - Integrate the Lambda-calling API into Frontend | 26/04/2026 | 26/04/2026 | AWS SDK S3 Presigned |
| 5 | - Develop an auto-retry mechanism on Frontend for network disconnections during upload | 27/04/2026 | 27/04/2026 | Axios Retry |
| 6 | - Test the PDF upload workflow from Client to S3 Bucket <br> - Fix progress bar display bugs | 28/04/2026 | 28/04/2026 | Internal |

### Week 2 Achievements:
* The PDF upload interface operates smoothly and stably.
* Users can upload large files directly to S3 via Presigned URLs.
* Effectively handled network exceptions (auto-retry) and file validation (size, format).