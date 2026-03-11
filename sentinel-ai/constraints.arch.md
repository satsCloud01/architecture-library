---
name: "Constraints & NFRs"
project: "Sentinel AI"
project_slug: "sentinel-ai"
project_url: "https://sentinel-ai.satszone.link"
github: "https://github.com/satsCloud01/sentinel-ai"
category: "data-analytics"
type: "constraints"
icon: "🛡️"
tags: [Performance, Security]
---

# Sentinel AI — Technical Constraints

**Data & AI Observability Platform**

This document captures the key technical constraints, decisions, and boundaries of the system.

---

## Runtime Environment

| Constraint | Detail |
|---|---|
| **Python version** | 3.12 (required) |
| **Node.js** | 18+ (for Vite build toolchain) |
| **Backend framework** | FastAPI with async/await throughout |
| **Frontend framework** | React 18 with Vite bundler |
| **greenlet dependency** | `greenlet` must be listed in `requirements.txt` — required by SQLAlchemy's async engine on this system (macOS Darwin). Without it, `create_async_engine` raises `ImportError` at startup. |

---

## Database

| Constraint | Detail |
|---|---|
| **Engine** | SQLite via `aiosqlite` (async driver) |
| **File** | Single file: `backend/sentinel.db` |
| **ORM** | SQLAlchemy 2.x with `AsyncSession` and `async_sessionmaker` |
| **Migrations** | Schema created on startup via `metadata.create_all()` — no Alembic |
| **Concurrency** | SQLite WAL mode recommended for concurrent reads; single-writer limitation accepted for demo/pitch scope |
| **Seed data** | Auto-seeded on first run when database is empty — realistic demo data across all 16 entities |
| **No connection pooling** | `aiosqlite` does not support connection pooling; `pool_size` and `max_overflow` are not configured |

---

## API Key Management

| Constraint | Detail |
|---|---|
| **No server-side persistence** | AI API keys (Anthropic Claude) are never stored in the database or on disk |
| **Client-side storage** | API keys are stored in the browser's `localStorage` |
| **Header transport** | Frontend sends the key via the `X-Api-Key` HTTP header on requests to AI-powered endpoints |
| **Mock fallback** | When no API key is provided, all AI-powered endpoints return deterministic mock/fallback responses — the app remains fully functional without an API key |
| **No `.env` requirement** | Unlike some sibling projects, Sentinel AI does not read `ANTHROPIC_API_KEY` from environment variables |

---

## Agent Template System

| Constraint | Detail |
|---|---|
| **Placeholder syntax** | Templates use `{{PLACEHOLDER}}` double-brace syntax for variable replacement |
| **Supported placeholders** | `{{SENTINEL_URL}}`, `{{AGENT_TOKEN}}`, `{{AGENT_NAME}}`, `{{CONNECTION_STRING}}`, `{{POLL_INTERVAL}}`, and type-specific placeholders |
| **10 agent types** | `database`, `file_storage`, `ai_llm`, `dbt`, `airflow`, `databricks`, `snowflake`, `athena`, `aws_datalake`, `custom` |
| **Generated output** | Self-contained Python scripts with embedded `SentinelClient` — no external SDK dependency beyond `httpx` and the connector library |
| **No agent runtime** | The platform generates scripts but does not execute or manage agents — clients deploy and run agents in their own environments |

---

## Network and CORS

| Constraint | Detail |
|---|---|
| **Backend port** | `8007` (all other ports in the portfolio are taken: 8000-8006) |
| **Frontend port** | `5179` (Vite dev server) |
| **CORS origins** | Allowed: `http://localhost:5173`, `http://localhost:5179`, `http://localhost:3000` |
| **Proxy** | Vite dev server proxies `/api/*` requests to `http://localhost:8007` |
| **No HTTPS locally** | TLS termination happens at the Nginx reverse proxy in production only |

---

## Frontend Constraints

| Constraint | Detail |
|---|---|
| **Build tool** | Vite (no CRA or Webpack) |
| **CSS** | Tailwind CSS 3.4 — utility-first, no custom CSS files |
| **Charts** | Recharts for all statistical visualizations (bar, line, area, pie) |
| **Lineage graph** | React Flow for interactive directed-graph visualization |
| **State** | React Query for server state; no Redux or Zustand |
| **Routing** | React Router v6 with lazy-loaded pages |
| **API keys in UI** | Entered via Settings page, stored in `localStorage`, never sent to non-AI endpoints |

---

## AI Integration

| Constraint | Detail |
|---|---|
| **Provider** | Anthropic Claude (Haiku model for cost efficiency) |
| **SDK** | `anthropic` Python SDK with async client |
| **Fallback** | All AI features have deterministic mock responses for demo without API key |
| **Features** | Root cause analysis, anomaly detection, insight generation, recommendations |
| **Token limits** | Responses capped at 1024 tokens per call |
| **No streaming** | AI responses are returned as complete JSON — no SSE/streaming |

---

## Deployment

| Constraint | Detail |
|---|---|
| **Local dev** | Backend: `cd backend && PYTHONPATH=src .venv/bin/uvicorn sentinel.main:app --reload --port 8007` |
| | Frontend: `cd frontend && npm run dev` (port 5179) |
| **Quick start** | `./start.sh` from project root starts both servers |
| **Docker** | Single `Dockerfile` at project root for production container |
| **Production** | Runs on consolidated AWS instance via Docker Compose (port 8007 mapped) |
| **Nginx** | Reverse proxy handles subdomain routing and SSL termination |

---

## Scope Boundaries

| Boundary | Detail |
|---|---|
| **Demo/pitch app** | Designed as a functional prototype for demonstrating Data & AI observability concepts — not production-hardened |
| **No authentication** | No user login, JWT, or RBAC — single-user assumed |
| **No real agent execution** | Agents are generated as scripts but not deployed or managed by the platform |
| **No real-time streaming** | No WebSocket or SSE for live updates — polling-based refresh |
| **No multi-tenancy** | Single SQLite database, no tenant isolation |
| **In-process seed data** | Demo data generated in Python at startup, not loaded from fixtures |

---

## Dependency Highlights

### Backend (`requirements.txt`)

| Package | Purpose |
|---|---|
| `fastapi` | Web framework |
| `uvicorn[standard]` | ASGI server |
| `sqlalchemy[asyncio]` | Async ORM |
| `aiosqlite` | SQLite async driver |
| `greenlet` | Required by SQLAlchemy async on macOS |
| `anthropic` | Claude API SDK |
| `pydantic` | Request/response validation |
| `httpx` | Async HTTP client (used in agent templates) |

### Frontend (`package.json`)

| Package | Purpose |
|---|---|
| `react`, `react-dom` | UI framework |
| `vite` | Build tool |
| `tailwindcss` | Utility CSS |
| `recharts` | Chart components |
| `reactflow` | Lineage graph visualization |
| `@tanstack/react-query` | Server state management |
| `react-router-dom` | Client-side routing |
| `lucide-react` | Icon library |
