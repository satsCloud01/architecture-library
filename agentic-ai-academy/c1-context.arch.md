---
name: "C1 - System Context"
project: "Agentic AI Academy"
project_slug: "agentic-ai-academy"
category: "learning"
type: "c4-context"
icon: "🤖"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'agentic-ai']
---

# C1 — System Context Diagram

## C1 · System Context

```
┌─────────────────────────────────────────────────────────┐
│                  Agentic AI Academy                      │
│       Interactive Agentic AI Protocols Platform           │
│                                                          │
│  36 lessons · 12 categories · 15 quiz questions          │
│  14 papers & specifications                              │
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
- **Learner** — Developer, architect, or engineer using a modern web browser.

**External Systems:**
- None. Fully self-contained. No database, no AI API keys, no external services at runtime.
- Specification links open in new tabs (read-only external navigation).

**Key Properties:**
- 100% offline-capable after initial page load
- No authentication required
- No user data persisted server-side (progress stored in browser localStorage)
- No AI API keys — zero `.env` dependencies


