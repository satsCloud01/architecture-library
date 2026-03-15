---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: Container Diagram
type: c4-container
tags: [C4, container, fastapi, react]
---

# C2 — Container Diagram

```
+------------------------------------------------------------------+
|                    SatsLearning Academy Platform                   |
|                                                                   |
|  +---------------------------+   +-----------------------------+  |
|  |     Frontend (SPA)        |   |     Backend (API)           |  |
|  |                           |   |                             |  |
|  |  React 18 + Vite          |   |  FastAPI (Python 3.12)      |  |
|  |  Tailwind CSS             |   |  Static data structures     |  |
|  |  Recharts (quiz charts)   |   |  4 API routers              |  |
|  |  Port 5181                |   |  No database                |  |
|  |                           |   |  Port 8025                  |  |
|  |  Serves: Landing,         |   |                             |  |
|  |  Learn (45 lessons),      |   |  Serves: REST API for       |  |
|  |  Simulator (ReAct viz),   |   |  lessons, simulator state,  |  |
|  |  Patterns gallery,        |   |  quiz scoring, paper        |  |
|  |  Quiz (scored + charted), |   |  catalog                    |  |
|  |  Papers (17 + reader)     |   |                             |  |
|  +------------+--------------+   +-------+---------------------+  |
|               |                          |                        |
|          HTTP /api/*                     |                        |
|          (proxied)                       |                        |
|               +----------->-------------+                        |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |              No Database / No External Services              |  |
|  |                                                              |  |
|  |  All lesson content, simulator scenarios, quiz questions,    |  |
|  |  and paper metadata are defined as Python dicts/lists in     |  |
|  |  the backend source code. Zero runtime dependencies.         |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

## Container Responsibilities

| Container | Technology | Responsibility |
|---|---|---|
| **Frontend** | React 18, Vite, Tailwind, Recharts | Interactive UI: 45-lesson wizard, ReAct simulator visualizer, pattern gallery, scored quiz with charts, paper reader with embedded iframe |
| **Backend** | FastAPI, Python 3.12 | REST API serving static content: lesson catalog, simulator step engine, quiz question bank and scoring, paper metadata |
