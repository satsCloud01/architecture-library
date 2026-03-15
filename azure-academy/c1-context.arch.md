---
name: "C1 - System Context"
project: "Azure Academy"
project_slug: "azure-academy"
category: "learning"
type: "c4-context"
icon: "🔷"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'azure']
---

# C1 — System Context Diagram

## C1 · System Context

```
┌─────────────────────────────────────────────────────────┐
│                     Azure Academy                        │
│       Interactive Azure Concepts Learning Platform        │
│                                                          │
│  42 lessons · 14 categories · 15 quiz questions          │
│  16 papers & documentation references                    │
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
- Azure documentation links open in new tabs (read-only external navigation).

**Key Properties:**
- 100% offline-capable after initial page load
- No authentication required
- No user data persisted server-side (progress stored in browser localStorage)
- No AI API keys — zero `.env` dependencies


