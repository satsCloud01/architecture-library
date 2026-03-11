---
name: "C2 – Container Diagram"
project: "DataGuardian"
project_slug: "data-guardian"
project_url: "https://data-guardian.satszone.link"
github: "https://github.com/satsCloud01/data-guardian"
category: "data-analytics"
type: "c4-container"
icon: "🛡️"
tags: [FastAPI, React, SQLAlchemy, SQLite]
---

# C2 — Container Diagram

DataGuardian consists of four containers communicating over HTTP/JSON.

```
┌──────────────────────────────────────────────────────────────────────┐
│                         data-guardian (monorepo)                     │
│                                                                      │
│  ┌─────────────────────────┐        ┌────────────────────────────┐  │
│  │      React SPA          │        │    FastAPI Governance API  │  │
│  │  Vite dev :5173         │◄──────►│    uvicorn :8001           │  │
│  │  Prod: Nginx static     │ HTTP   │    PYTHONPATH=src          │  │
│  │                         │ /api   │                            │  │
│  │  React 18.3             │        │  FastAPI 0.115             │  │
│  │  TypeScript 5.x         │        │  Python 3.12               │  │
│  │  Vite 5.x               │        │  SQLAlchemy 2.0 async      │  │
│  │  Tailwind CSS 3.x       │        │  Pydantic v2               │  │
│  │  React Flow (lineage)   │        │  aiosqlite                 │  │
│  │  Recharts (quality)     │        │                            │  │
│  │  Lucide React (icons)   │        │  8 routers (see C3)        │  │
│  │  React Router v6        │        │  Auto-docs: /docs          │  │
│  └─────────────────────────┘        └────────────┬───────────────┘  │
│                                                   │                  │
│                                                   │ SQLAlchemy async │
│                                                   ▼                  │
│                                    ┌──────────────────────────────┐  │
│                                    │   Governance Database        │  │
│                                    │   SQLite (dev)               │  │
│                                    │   dataguardian.db            │  │
│                                    │                              │  │
│                                    │   13 tables                  │  │
│                                    │   15 seeded models           │  │
│                                    │   201 fields                 │  │
│                                    │   6 mo. quality history      │  │
│                                    └──────────────────────────────┘  │
│                                                   ▲                  │
│                                                   │ seed on startup  │
│                                    ┌──────────────────────────────┐  │
│                                    │   Demo Data Seeder           │  │
│                                    │   seed.py (idempotent)       │  │
│                                    │   Faker-generated realistic  │  │
│                                    │   banking data               │  │
│                                    └──────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
                                           │ outbound (optional)
                                           ▼
                             ┌─────────────────────────┐
                             │  Anthropic Claude API   │
                             │  claude-haiku-4-5-20251001│
                             │  Key: X-API-Key header  │
                             └─────────────────────────┘
```

---

## Container Details

### React SPA

| Attribute | Value |
|-----------|-------|
| **Technology** | React 18.3 · TypeScript · Vite 5 · Tailwind CSS 3 |
| **Port (dev)** | 5173 (Vite) |
| **Port (prod)** | 443 (Nginx serves `/dist`) |
| **API Communication** | `fetch()` to `/api/*` — proxied by Vite dev server (dev) or Nginx (prod) |
| **State** | Local component state + React Context (`ApiKeyContext` for AI key) |
| **Routing** | React Router v6 with 10 pages |
| **Key Dependencies** | `reactflow` (lineage DAG), `recharts` (quality charts), `lucide-react` (icons) |

**10 Pages:** Landing, Dashboard, ModelRegistry, ModelDetail, LineageExplorer, ChangeRequests, Glossary, Compliance, DataQuality, Settings

### FastAPI Governance API

| Attribute | Value |
|-----------|-------|
| **Technology** | FastAPI 0.115 · Python 3.12 · SQLAlchemy 2.0 async · Pydantic v2 |
| **Port** | 8001 |
| **ORM** | SQLAlchemy 2.0 with `asyncio` + `aiosqlite` driver |
| **Startup** | Async lifespan: `init_db()` → `seed()` |
| **CORS** | `localhost:5173`, `localhost:3000`, production origin |
| **Docs** | `/docs` (Swagger), `/redoc` (ReDoc) |
| **Routers** | 8 (see C3 Component Diagram) |

### Governance Database

| Attribute | Value |
|-----------|-------|
| **Dev Engine** | SQLite 3 via `aiosqlite` |
| **Prod Engine** | PostgreSQL 15+ (change `DATABASE_URL` in `.env`) |
| **Tables** | 13 (see Domain Model) |
| **Location** | `backend/dataguardian.db` (auto-created on first run) |
| **Migrations** | Schema auto-created via `Base.metadata.create_all` |

### Demo Data Seeder

| Attribute | Value |
|-----------|-------|
| **File** | `backend/src/dataguardian/seed.py` |
| **Strategy** | Idempotent — checks for existing data before inserting |
| **Data Volume** | 15 models, 201 fields, 8 standards, 15 lineage links, 5 changes, 12 glossary terms |
| **Trigger** | Runs automatically on app startup via lifespan |

---

## Communication Flow

```
Browser ──HTTPS──► Nginx ──static──► /dist/index.html (React bundle)
                         ──proxy──► FastAPI :8001 (all /api/* requests)

FastAPI ──async──► SQLite (local queries)
        ──HTTPS──► Anthropic API (AI requests, if X-API-Key header present)
```
