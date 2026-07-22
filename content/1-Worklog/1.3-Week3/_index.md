---
title: "Week 3 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---


### Week 3 Objectives:
* Complete Epic 1: Build the Bilingual Side-by-Side Reader.
* Render specialized structures (LaTeX Mathematics).

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Design Bilingual Reader UI (2 parallel columns on Desktop, EN/VI tabs on Mobile) | 01/05/2026 | 01/05/2026 | Tailwind CSS |
| 3 | - Develop Sync Scroll feature between the source and translated columns | 02/05/2026 | 02/05/2026 | React Refs & Scroll |
| 4 | - Integrate `katex` library (^0.17.0) to render math formulas <br> - Handle parsing raw LaTeX code from Markdown | 03/05/2026 | 03/05/2026 | KaTeX Docs |
| 5 | - Develop "Quick Copy LaTeX code" action button for formulas | 04/05/2026 | 04/05/2026 | Browser Clipboard API |
| 6 | - Mock bilingual Markdown data to test Reader UI <br> - Optimize UI responsiveness for mobile devices | 05/05/2026 | 05/05/2026 | Internal |

### Week 3 Achievements:
* Completed the smart bilingual reader, responsive across multiple devices.
* Sync Scroll feature functions smoothly.
* Complex mathematical formulas in academic papers are correctly rendered in standard format via KaTeX.