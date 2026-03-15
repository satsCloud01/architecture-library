---
title: Networking Academy — Architecture Documentation
scope: C1 System Context, C2 Container, C3 Component, C4 Code
status: current
---

# Networking Academy — Architecture

## C1 · System Context

```
┌─────────────────────────────────────────────────────────┐
│                    Networking Academy                     │
│         Interactive Networking Concepts Platform          │
│                                                          │
│  50 lessons · 13 categories · 15 quiz questions          │
│  4 simulators · 10 patterns · 17 RFCs/papers             │
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
- **Learner** — Developer, network engineer, student, or architect using a modern web browser.

**External Systems:**
- None. Fully self-contained. No database, no AI API keys, no external services at runtime.
- RFC/paper links open in new tabs (read-only external navigation).

**Key Properties:**
- 100% offline-capable after initial page load
- No authentication required
- No user data persisted server-side (progress stored in browser localStorage)
- No AI API keys — zero `.env` dependencies


## C2 · Container Diagram

```
┌──────────────────────────────┐     ┌──────────────────────────────┐
│      React 18 Frontend       │     │      FastAPI Backend          │
│      (Vite + Tailwind)       │────▶│      (Python 3.12)           │
│                              │     │                              │
│  Port: 5182                  │ /api│  Port: 8026                  │
│  6 pages + tour overlay      │     │  4 routers, static data      │
│  Recharts for quiz scores    │     │  No database, no AI          │
│  localStorage for progress   │     │                              │
└──────────────────────────────┘     └──────────────────────────────┘
```

| Container | Technology | Purpose |
|-----------|-----------|---------|
| Frontend | React 18 + Vite + Tailwind CSS + Recharts | Interactive UI with lesson viewer, simulator, quiz, patterns, papers |
| Backend | FastAPI + Uvicorn | REST API serving lesson content, quiz scoring, simulator steps |
| Static Data | Python dicts in `data/lessons.py` + router files | All content authored in Python — no database |

**No database.** All 50 lessons, 15 quiz questions, 4 simulator scenarios, 10 patterns, and 17 papers are defined as Python data structures.


## C3 · Component Diagram

### Backend Components

```
networkacademy/
├── main.py                    # FastAPI app, CORS, router registration, health endpoint
├── data/
│   └── lessons.py             # LESSONS (50 items), QUIZ_QUESTIONS (15 items)
└── routers/
    ├── lessons.py             # GET /lessons/, GET /lessons/{id}, GET /lessons/categories/summary
    ├── quiz.py                # GET /quiz/questions (no answers), POST /quiz/check (scoring)
    ├── simulator.py           # GET /simulator/scenarios, GET /scenarios/{id}, POST /step
    └── papers.py              # GET /papers/, GET /papers/{id} — 17 RFCs with key_takeaways
```

| Router | Endpoints | Data Source |
|--------|----------|-------------|
| lessons | `GET /`, `GET /{id}`, `GET /categories/summary` | `LESSONS` list (50 items, 13 categories) |
| quiz | `GET /questions`, `POST /check` | `QUIZ_QUESTIONS` list (15 items) |
| simulator | `GET /scenarios`, `GET /scenarios/{id}`, `POST /step` | `SCENARIOS` list (4 scenarios, ~30 steps) |
| papers | `GET /`, `GET /{id}` | `PAPERS` list (17 items with key_takeaways) |

### Frontend Components

```
src/
├── App.jsx                    # React Router v6 — 7 routes
├── components/
│   ├── layout/
│   │   └── Sidebar.jsx        # Fixed left nav with 7 links
│   └── ui/
│       └── TourOverlay.jsx    # 6-step guided tour modal
├── hooks/
│   └── useTour.js             # Tour state (localStorage persistence)
├── lib/
│   └── api.js                 # Fetch wrapper — all API calls
└── pages/
    ├── Landing.jsx            # Hero, stats, feature cards, curriculum grid, tour trigger
    ├── Learn.jsx              # Wizard-style lesson viewer with category nav + progress bar
    ├── Simulator.jsx          # Step-through network scenarios (ReAct pattern)
    ├── Patterns.jsx           # 10 network patterns with trade-off analysis
    ├── Quiz.jsx               # 15 questions, per-category scoring, Recharts bar chart
    ├── Papers.jsx             # 17 RFCs with key_takeaways, related_topics, doc info
    └── Settings.jsx           # System status, progress reset
```

### Lesson Content Types

Each lesson in `LESSONS` can include multiple content types:

| Field | Type | Rendered As |
|-------|------|------------|
| `description` | string | Intro paragraph |
| `keyPoints` | string[] | Bullet list with ▸ markers |
| `comparison` | object (headers + rows) | Styled table |
| `diagram` | object (nodes + edges) | Flow/architecture diagram |
| `sequenceDiagram` | object (actors + steps) | Protocol sequence diagram |
| `code` / `codeLanguage` | string | Syntax-highlighted code block |


## C4 · Code Level

### Lesson Data Structure

```python
{
    "id": 11,
    "category": "TCP/IP",
    "title": "TCP Three-Way Handshake",
    "type": "concept",
    "description": "Before data transfer, TCP establishes...",
    "keyPoints": ["SYN: Client initiates...", ...],
    "sequenceDiagram": {
        "title": "TCP Three-Way Handshake",
        "actors": ["Client", "Server"],
        "steps": [
            {"from": "Client", "to": "Server", "label": "SYN (seq=100)", "type": "request", "protocol": "TCP"},
            ...
        ]
    }
}
```

### Quiz Flow

```
Browser                    Backend
  │                          │
  │ GET /quiz/questions ────▶│  Returns questions WITHOUT correct answers
  │◀──── [{id, question,     │
  │        options}]          │
  │                          │
  │ POST /quiz/check ───────▶│  Server-side scoring
  │  {question_id, selected} │
  │◀──── {correct, index,    │
  │        explanation}       │
```

### Simulator ReAct Pattern

Each simulator scenario uses the ReAct (Reason + Act) execution model:
- **thought** — Decision/reasoning step (purple)
- **action** — Network operation performed (cyan)
- **observation** — Result observed (amber)
- **answer** — Final conclusion (green)

Steps are revealed progressively via `POST /simulator/step` with `step_index`.

### Progress Tracking

```javascript
// Browser localStorage
"net-academy-completed" → [1, 2, 3, ...]  // Completed lesson IDs
"networking-academy-tour" → "true"         // Tour dismissed
```

No server-side persistence. Progress is per-browser.
