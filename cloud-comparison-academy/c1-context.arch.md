---
name: "C1 - System Context"
project: "Cloud Comparison Academy"
project_slug: "cloud-comparison-academy"
category: "learning"
type: "c4-context"
icon: "⚖️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'multi-cloud']
---

# C1 — System Context Diagram

## C1 · System Context

```
┌─────────────────────────────────────────────────────────┐
│                Cloud Comparison Academy                  │
│     Interactive AWS vs Azure vs GCP Learning Platform     │
│                                                          │
│  28 lessons · 6 categories · 15 quiz questions           │
│  12 papers · weighted calculator                         │
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
- **Learner** — Cloud architect, engineering leader, or decision-maker using a modern web browser.

**External Systems:**
- None. Fully self-contained. No database, no AI API keys, no external services at runtime.
- Cloud documentation links open in new tabs (read-only external navigation).

**Key Properties:**
- 100% offline-capable after initial page load
- No authentication required
- No user data persisted server-side (progress stored in browser localStorage)
- No AI API keys — zero `.env` dependencies


