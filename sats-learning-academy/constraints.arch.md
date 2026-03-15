---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: Constraints
type: constraints
tags: [constraints, decisions, scope]
---

# SatsLearning Academy — Technical Constraints

**Agentic AI Educational Platform**

This document captures the key technical constraints, decisions, and boundaries of the system.

---

## Runtime Environment

| Constraint | Detail |
|---|---|
| **Python version** | 3.12 (required) |
| **Node.js** | 18+ (for Vite build toolchain) |
| **Backend framework** | FastAPI with sync handlers (no async DB needed) |
| **Frontend framework** | React 18 with Vite bundler |
| **Package name** | `academy` at `backend/src/academy/` |

---

## No Database

| Constraint | Detail |
|---|---|
| **Storage** | All content is defined as static Python data structures (dicts and lists) in source code |
| **No SQLite** | Unlike sibling projects (Sentinel AI, DataGuardian, etc.), this app has zero database dependencies |
| **No ORM** | No SQLAlchemy, no aiosqlite, no greenlet dependency |
| **No migrations** | Content changes are code changes — edit Python source, restart server |
| **No seed data** | There is no seeding step; data is always available from import time |
| **Implications** | No persistence between restarts (but since data is static, this is irrelevant). No write operations. No connection pooling concerns. |

---

## No AI API Keys

| Constraint | Detail |
|---|---|
| **No Anthropic API key** | The platform does not call Claude or any other LLM API |
| **No `.env` file** | No environment variables required for any functionality |
| **No BYOK pattern** | Unlike sibling projects, there is no API key input in the UI |
| **No mock fallback** | There is no AI feature to mock — all content is pre-authored by humans |
| **100% offline capable** | The app functions identically without internet access (except for paper URLs and the embedded PDF reader iframe) |

---

## Static Content

| Constraint | Detail |
|---|---|
| **45 lessons** | Across 7 categories: Foundations (6), Protocols (13), Design Patterns (7), Architectures (5), Memory & State (4), Frameworks (5), Advanced (5) |
| **4 simulator scenarios** | Pre-scripted ReAct execution traces with ~10 steps each |
| **10 quiz questions** | Multiple-choice with explanations; scoring computed server-side |
| **17 research papers** | Metadata only (title, authors, org, URL, abstract, tags); actual PDFs hosted externally |
| **Content authoring** | Developers edit Python source files directly; no CMS, no admin panel |

---

## Network and CORS

| Constraint | Detail |
|---|---|
| **Backend port** | `8025` (all ports 8000-8023 taken by sibling projects) |
| **Frontend port** | `5181` (Vite dev server) |
| **CORS origins** | Allowed: `http://localhost:5181` and other common dev ports |
| **Proxy** | Vite dev server proxies `/api/*` requests to `http://localhost:8025` |
| **No HTTPS locally** | TLS termination would happen at reverse proxy in production only |

---

## Frontend Constraints

| Constraint | Detail |
|---|---|
| **Build tool** | Vite (no CRA or Webpack) |
| **CSS** | Tailwind CSS — utility-first, no custom CSS files |
| **Charts** | Recharts for quiz score visualization (bar and pie charts) |
| **No React Flow** | Unlike Sentinel AI or DataGuardian, no graph visualization library needed |
| **State** | Plain React state (`useState`, `useEffect`) — no React Query, no Redux, no Zustand |
| **Routing** | React Router v6 |
| **6 custom visual components** | SequenceDiagram, TransportDiagram, McpEcosystem, StateMachine, AgentCard, MessageFormats — all render within lessons |
| **Embedded reader** | Papers page uses an iframe to embed external PDF viewers |

---

## Scope Boundaries

| Boundary | Detail |
|---|---|
| **Educational/demo app** | Designed to teach agentic AI concepts — not a production LMS |
| **No authentication** | No user login, JWT, or RBAC — anonymous single-user assumed |
| **No progress persistence** | Lesson progress and quiz scores are not saved between sessions |
| **No user-generated content** | All content is developer-authored; users cannot create lessons or quizzes |
| **No real agent execution** | The simulator visualizes pre-scripted traces; it does not run actual agents |
| **No search** | Content volume (45 lessons, 17 papers) is small enough for category-based browsing |
| **No multi-tenancy** | Single-user, stateless application |
| **No analytics/telemetry** | No usage tracking, no event logging |

---

## Dependency Highlights

### Backend (`requirements.txt`)

| Package | Purpose |
|---|---|
| `fastapi` | Web framework |
| `uvicorn[standard]` | ASGI server |
| `pydantic` | Request/response validation |

### Frontend (`package.json`)

| Package | Purpose |
|---|---|
| `react`, `react-dom` | UI framework |
| `vite` | Build tool |
| `tailwindcss` | Utility CSS |
| `recharts` | Quiz score charts |
| `react-router-dom` | Client-side routing |
| `lucide-react` | Icon library |

---

## Comparison with Sibling Projects

| Feature | SatsLearning Academy | Sentinel AI | DataGuardian |
|---|---|---|---|
| Database | None | SQLite (16 tables) | SQLite (15 models) |
| AI API | None | Claude Haiku | Claude Haiku |
| API keys | None | localStorage + header | .env + header |
| Write endpoints | None | Full CRUD | Full CRUD |
| External services | None | Alert channels | None |
| Backend port | 8025 | 8007 | 8001 |
| Frontend port | 5181 | 5179 | 5174 |
