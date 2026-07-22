---
title: "Week 9 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.9. </b> "
---


### Week 9 Objectives:
* Deploy Epic 4: Active Learning Tools.
* Build Async Worker Pattern for Backend.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Set up Async Worker model: <br>&emsp; + API Handler writes to DynamoDB and returns jobId <br>&emsp; + Lambda triggers asynchronously (`Event`) | 12/06/2026 | 12/06/2026 | AWS Lambda Invocation |
| 3 | - Build continuous Polling mechanism `GET /job/{jobId}` on Frontend to update status | 13/06/2026 | 13/06/2026 | React Query / SWR |
| 4 | - Develop AI Quiz Generator Lambda <br> - Design interactive quiz modal UI on Web | 14/06/2026 | 14/06/2026 | Prompt Engineering |
| 5 | - Develop AI Flashcard Generator Lambda <br> - Design flip card memory UI (Swiper UI) | 15/06/2026 | 15/06/2026 | Swiper.js |
| 6 | - Integration testing for Quiz and Flashcard generation flows <br> - Ensure DynamoDB stores the returned JSON structure correctly | 16/06/2026 | 16/06/2026 | Internal |

### Week 9 Achievements:
* Successfully applied Async Worker pattern, resolving the 29s API Gateway Timeout issue for heavy AI tasks.
* Users can generate multiple-choice quizzes and flashcards directly from scientific papers.