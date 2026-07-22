---
title: "Week 11 Worklog"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 1.11. </b> "
---


### Week 11 Objectives:
* Deploy Epic 5 (Advanced Agents): Explore Mode & Extended Search.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Implement *Explore Mode (Story 5.2)*: Initialize Lambda to generate detailed topic lectures <br> - Code Self-Healing logic for Mermaid/Math syntax | 26/06/2026 | 26/06/2026 | Agentic Self-Correction |
| 3 | - Task 2.1 (*Story 5-3*): Structure the AI Agent for Scholar Search Agent <br> - Integrate Semantic Scholar API for dynamic paper lookup | 27/06/2026 | 27/06/2026 | Semantic Scholar API |
| 4 | - Write `scholar-search.ts` handler <br> - Configure search integration into AI Tutor Tools within Chat | 28/06/2026 | 28/06/2026 | Function Calling |
| 5 | - Build Cross-paper Synthesis Chat feature | 29/06/2026 | 29/06/2026 | Multi-document RAG |
| 6 | - Test multi-agent collaboration, monitor CloudWatch Logs to capture Self-Healing errors | 30/06/2026 | 30/06/2026 | AWS CloudWatch |

### Week 11 Achievements:
* Explore Mode is fully operational, automatically generating lectures with syntax self-correction.
* AI Tutor can flexibly search external knowledge beyond the PDF (Scholar Search Agent) and synthesize multi-document findings.