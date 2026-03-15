---
name: "C3 - Component Diagram"
project: "Azure Academy"
project_slug: "azure-academy"
category: "learning"
type: "c4-component"
icon: "🔷"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'azure']
---

# C3 — Component Diagram

## C3 · Component Diagram

### Backend Components

```
azureacademy/
├── main.py                    # FastAPI app, CORS, router registration, health endpoint
├── data/
│   └── lessons.py             # LESSONS (42 items), QUIZ_QUESTIONS (15 items)
└── routers/
    ├── lessons.py             # GET /lessons/, GET /lessons/{id}, GET /lessons/categories/summary
    ├── quiz.py                # GET /quiz/questions (no answers), POST /quiz/check (scoring)
    └── papers.py              # GET /papers/, GET /papers/{id} — 16 docs with key_takeaways
```

| Router | Endpoints | Data Source |
|--------|----------|-------------|
| lessons | `GET /`, `GET /{id}`, `GET /categories/summary` | `LESSONS` list (42 items, 14 categories) |
| quiz | `GET /questions`, `POST /check` | `QUIZ_QUESTIONS` list (15 items) |
| papers | `GET /`, `GET /{id}` | `PAPERS` list (16 items with key_takeaways) |

### Frontend Components

```
src/
├── App.jsx                    # React Router v6 — 5 routes
├── components/
│   ├── layout/
│   │   └── Sidebar.jsx        # Fixed left nav with 5 links
│   └── ui/
│       └── TourOverlay.jsx    # 6-step guided tour modal
├── hooks/
│   └── useTour.js             # Tour state (localStorage persistence)
├── lib/
│   └── api.js                 # Fetch wrapper — all API calls
└── pages/
    ├── Landing.jsx            # Hero, stats, feature cards, curriculum grid, tour trigger
    ├── Learn.jsx              # Split layout: category nav + lesson content with progress tracking
    ├── Quiz.jsx               # 15 questions, per-category scoring, Recharts bar chart
    ├── Papers.jsx             # 16 Azure docs with key_takeaways, related_topics
    └── Settings.jsx           # System status, progress reset
```

### Lesson Content Types

Each lesson in `LESSONS` can include multiple content types:

| Field | Type | Rendered As |
|-------|------|------------|
| `description` | string | Intro paragraph |
| `keyPoints` | string[] | Bullet list with markers |
| `comparison` | object (headers + rows) | Styled table |
| `diagram` | object (nodes + edges) | Flow/architecture diagram |
| `code` / `codeLanguage` | string | Syntax-highlighted code block |


