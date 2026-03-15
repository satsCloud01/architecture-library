---
name: "C2 - Container Diagram"
project: "Agentic AI Academy"
project_slug: "agentic-ai-academy"
category: "learning"
type: "c4-container"
icon: "🤖"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'agentic-ai']
---

# C2 — Container Diagram

## C2 · Container Diagram

```
┌──────────────────────────────┐     ┌──────────────────────────────┐
│      React 18 Frontend       │     │      FastAPI Backend          │
│      (Vite + Tailwind)       │────▶│      (Python 3.12)           │
│                              │     │                              │
│  Port: 5190                  │ /api│  Port: 8030                  │
│  7 pages + tour overlay      │     │  3 routers, static data      │
│  Recharts for quiz scores    │     │  No database, no AI          │
│  localStorage for progress   │     │                              │
└──────────────────────────────┘     └──────────────────────────────┘
```

| Container | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | React 18 + Vite + Tailwind CSS + Recharts | Interactive UI with lesson viewer, quiz, papers, simulator, patterns |
| Backend | FastAPI + Uvicorn | REST API serving lesson content, quiz scoring |
| Static Data | Python dicts in `data/lessons.py` + router files | All content authored in Python — no database |

**No database.** All 36 lessons, 15 quiz questions, and 14 papers are defined as Python data structures.


