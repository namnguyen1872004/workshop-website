---
title: "Week 4 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---


### Week 4 Objectives:
* Start Epic 2: User Authentication.
* Protect Backend APIs and manage sessions.

### Tasks to Deploy This Week:
| Day | Task | Start Date | Completion Date | Documentation Source |
| --- | --- | --- | --- | --- |
| 2 | - Integrate `NextAuth.js` (v5.0.0-beta.31) into Frontend <br> - Set up Google OAuth Provider | 08/05/2026 | 08/05/2026 | NextAuth Docs |
| 3 | - Set up Email OTP authentication (using HMAC signature) <br> - Create Login / Register pages | 09/05/2026 | 09/05/2026 | NextAuth Email Auth |
| 4 | - Write Custom Lambda Authorizer on Backend <br> - Apply `crypto.subtle` for JWT verification (HS256 algorithm) | 10/05/2026 | 10/05/2026 | AWS Lambda Authorizer |
| 5 | - Configure API Gateway to use the newly created Lambda Authorizer <br> - Enforce Frontend to attach Bearer Token | 11/05/2026 | 11/05/2026 | AWS API Gateway |
| 6 | - Test unauthenticated access restriction flows (Next.js Middleware) <br> - Block upload features for unauthenticated users | 12/05/2026 | 12/05/2026 | Next.js Middleware |

### Week 4 Achievements:
* The Stateless JWT login system operates successfully with Google OAuth and OTP.
* Backend is securely protected via Custom Lambda Authorizer without external dependencies.
* API Gateway successfully validates authorized requests from Frontend.