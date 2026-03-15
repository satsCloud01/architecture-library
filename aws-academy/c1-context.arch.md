---
name: "C1 - System Context"
project: "AWS Academy"
project_slug: "aws-academy"
category: "learning"
type: "c4-context"
icon: "☁️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'aws']
---

# C1 — System Context Diagram

## C1 · System Context

```
┌─────────────────────────────────────────────────────────┐
│                      AWS Academy                         │
│       Interactive AWS Concepts Learning Platform          │
│                                                          │
│  42 lessons · 14 categories · 15 quiz questions          │
│  14 papers & references                                  │
└─────────────────────────────────────────────────────────┘
                          ▲
                          │ HTTPS
                          │
                    ┌─────┴─────┐
                    │   Learner  │
                    │ (Browser)  │
                    └───────────┘
```

**Actors:**
- **Learner** — Developer, cloud engineer, solutions architect, or student using a modern web browser.

**External Systems:**
- None. Fully self-contained. No database, no AI API keys, no external services at runtime.
- AWS documentation links open in new tabs (read-only external navigation).

**Key Properties:**
- 100% offline-capable after initial page load
- No authentication required
- No user data persisted server-side (progress stored in browser localStorage)
- No AI API keys — zero `.env` dependencies


