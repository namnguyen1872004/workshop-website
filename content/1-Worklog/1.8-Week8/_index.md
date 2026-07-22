---
title: "Week 8 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---


### Week 8 Objectives:
* Complete Epic 3: Smart Chat Assistant (Agentic Chat Logic).

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Integrate `gemini-2.5-flash` model for user query processing <br> - Set up system prompt | 05/06/2026 | 05/06/2026 | Google Gemini API |
| 3 | - Develop Function Calling (Tool Use) for LLM: `vectorSearch` (query Qdrant) | 06/06/2026 | 06/06/2026 | Function Calling Docs |
| 4 | - Develop `fetchAdjacentParagraphs` tool to prevent context loss <br> - Develop `readExecutiveSummary` tool from DynamoDB | 07/06/2026 | 07/06/2026 | Internal |
| 5 | - Write logic to auto-attach citation backlinks `[Paragraph X]` into responses <br> - Configure Scroll-to-view on FE | 08/06/2026 | 08/06/2026 | DOM Manipulation |
| 6 | - Comprehensive testing of AI Tutor response quality <br> - Handle streaming responses (Server-Sent Events) | 09/06/2026 | 09/06/2026 | Web Streams API |

### Week 8 Achievements:
* AI Tutor exhibits deep document comprehension, autonomously invoking tools to fetch context.
* Smooth Q&A experience; responses include `[Paragraph X]` citations that auto-scroll to the exact location in the paper upon clicking.