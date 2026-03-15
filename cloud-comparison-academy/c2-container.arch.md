---
name: "C2 - Container Diagram"
project: "Cloud Comparison Academy"
project_slug: "cloud-comparison-academy"
category: "learning"
type: "c4-container"
icon: "⚖️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'multi-cloud']
---

# C2 — Container Diagram

## C2 · Container Diagram

```
┌──────────────────────────────┐     ┌──────────────────────────────┐
│      React 18 Frontend       │     │      FastAPI Backend          │
│      (Vite + Tailwind)       │────▶│      (Python 3.12)           │
│                              │     │                              │
│  Port: 5193                  │ /api│  Port: 8036                  │
│  6 pages + tour overlay      │     │  3 routers, static data      │
│  Recharts for quiz & calc    │     │  No database, no AI          │
│  localStorage for progress   │     │                              │
└──────────────────────────────┘     └──────────────────────────────┘
```

| Container | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | React 18 + Vite + Tailwind CSS + Recharts | Interactive UI with lesson viewer, quiz, papers, calculator |
| Backend | FastAPI + Uvicorn | REST API serving lesson content, quiz scoring |
| Static Data | Python dicts in `data/lessons.py` + router files | All content authored in Python — no database |

**No database.** All 28 lessons, 15 quiz questions, and 12 papers are defined as Python data structures. The weighted calculator is frontend-only logic.


