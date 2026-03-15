---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: Component Diagram
type: c4-component
tags: [C4, component, routers, pages]
---

# C3 — Component Diagram

## Backend Components

```
+------------------------------------------------------------------+
|                    FastAPI Backend (Port 8025)                     |
|                                                                   |
|  +--------------------+  +--------------------+                   |
|  | academy.main       |  | Middleware          |                  |
|  | (app entrypoint)   |  | - CORS (localhost   |                  |
|  | - router mounting  |  |   origins)          |                  |
|  | - health endpoint  |  | - Error handlers    |                  |
|  +--------------------+  +--------------------+                   |
|                                                                   |
|  Routers (4)                                                      |
|  +------------------+  +------------------+                       |
|  | /api/lessons     |  | /api/simulator   |                       |
|  | - list all       |  | - list scenarios |                       |
|  | - get by id      |  | - get scenario   |                       |
|  | - category       |  | - step through   |                       |
|  |   summary        |  |   execution      |                       |
|  +------------------+  +------------------+                       |
|                                                                   |
|  +------------------+  +------------------+                       |
|  | /api/quiz        |  | /api/papers      |                       |
|  | - get questions  |  | - list all       |                       |
|  | - check answers  |  | - get by id      |                       |
|  | - score + stats  |  | - metadata only  |                       |
|  +------------------+  +------------------+                       |
|                                                                   |
|  Static Data Layer                                                |
|  +------------------------------------------------------------+  |
|  | LESSONS: list[dict]     — 45 lessons across 7 categories    |  |
|  | SCENARIOS: list[dict]   — 4 ReAct simulator scenarios       |  |
|  | QUESTIONS: list[dict]   — 10 quiz questions                 |  |
|  | PAPERS: list[dict]      — 17 curated research papers        |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

## Frontend Components

```
+------------------------------------------------------------------+
|                   React Frontend (Port 5181)                       |
|                                                                   |
|  +--------------------+  +-------------------------------------+  |
|  | App Shell           |  | State Management                   |  |
|  | - React Router      |  | - Component state (useState)       |  |
|  | - Layout + Nav      |  | - No external state library        |  |
|  | - Theme (Tailwind)  |  | - No localStorage usage            |  |
|  +--------------------+  +-------------------------------------+  |
|                                                                   |
|  Pages (6)                                                        |
|  +------------------+  +------------------+  +------------------+ |
|  | Landing          |  | Learn            |  | Simulator        | |
|  | - hero section   |  | - 45-lesson      |  | - ReAct step-by- | |
|  | - feature cards  |  |   wizard         |  |   step visualizer| |
|  | - navigation     |  | - 7 categories   |  | - 4 scenarios    | |
|  |                  |  | - progress track |  | - animated flow  | |
|  +------------------+  +------------------+  +------------------+ |
|                                                                   |
|  +------------------+  +------------------+  +------------------+ |
|  | Patterns         |  | Quiz             |  | Papers           | |
|  | - design pattern |  | - 10 questions   |  | - 17 papers      | |
|  |   gallery        |  | - answer check   |  | - filterable list| |
|  | - visual cards   |  | - Recharts score |  | - embedded reader| |
|  |                  |  |   breakdown      |  |   (iframe)       | |
|  +------------------+  +------------------+  +------------------+ |
|                                                                   |
|  Visual Components (6 custom)                                     |
|  +------------------+  +------------------+  +------------------+ |
|  | SequenceDiagram  |  | TransportDiagram |  | McpEcosystem     | |
|  | - protocol flows |  | - transport layer|  | - MCP component  | |
|  | - message arrows |  |   visualization  |  |   relationships  | |
|  +------------------+  +------------------+  +------------------+ |
|                                                                   |
|  +------------------+  +------------------+  +------------------+ |
|  | StateMachine     |  | AgentCard        |  | MessageFormats   | |
|  | - state          |  | - agent profile  |  | - protocol       | |
|  |   transitions    |  |   display        |  |   message        | |
|  | - animated       |  | - capability     |  |   structure      | |
|  +------------------+  +------------------+  +------------------+ |
+------------------------------------------------------------------+
```
