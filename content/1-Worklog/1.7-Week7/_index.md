---
title: "Week 7 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---


### Week 7 Objectives:
* Complete Epic 3: Data Vectorization (RAG Ingestion) and Agentic RAG Configuration.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Write `Ingest` Lambda (final step of Step Functions) <br> - Configure Markdown chunking based on anchors | 29/05/2026 | 29/05/2026 | Basic RAG Logic |
| 3 | - Connect `@google/generative-ai` (`gemini-embedding-001`) to get 768-dimensional vectors for each chunk | 30/05/2026 | 30/05/2026 | Google Gemini API |
| 4 | - Connect `@qdrant/js-client-rest` to save vectors to Qdrant Cloud <br> - Attach metadata (userId, jobId) | 31/05/2026 | 31/05/2026 | Qdrant Docs |
| 5 | - Build AI Tutor Panel (3rd column of the Workspace) <br> - Configure basic chat UI | 01/06/2026 | 01/06/2026 | Tailwind CSS |
| 6 | - Integrate Semantic Scholar API for "Related Papers" tab to display Open Access articles | 02/06/2026 | 02/06/2026 | Semantic Scholar API |

### Week 7 Achievements:
* Completed RAG (Retrieval-Augmented Generation) infrastructure: Data is chunked, embedded, and upserted into Qdrant Vector DB.
* Finished the 3-column Workspace UI (Library - Reader - RAG Chat).
* Static lookups for related papers function properly.
