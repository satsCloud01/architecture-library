---
name: "C2 – Container Diagram"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "c4-container"
icon: "🔐"
tags: [FastAPI, React, SQLAlchemy]
---

# C2 — Container Diagram

> Level 2 of the C4 model: what are the deployable units that make up the system?

---

## Container Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                   DYNAMIC POLICY ACCESS ENGINE                      │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    React SPA (Vite)                         │   │
│   │                  Port 5176 (dev) / Nginx                    │   │
│   │                                                             │   │
│   │  Pages: Dashboard | Policy Explorer | Policy Studio |      │   │
│   │         Access Gateway | Simulator | Audit Trail |         │   │
│   │         Principals | Data Sources | Settings               │   │
│   │                                                             │   │
│   │  Tech: React 18, Vite, Tailwind CSS, Recharts, React Router│   │
│   └──────────────────────────┬──────────────────────────────────┘   │
│                              │ HTTP/JSON (REST)                     │
│                              │ /api/*                               │
│   ┌──────────────────────────▼──────────────────────────────────┐   │
│   │                   FastAPI Application                       │   │
│   │                  Port 8003 / uvicorn                        │   │
│   │                                                             │   │
│   │  ┌────────────────────────────────────────────────────┐    │   │
│   │  │             ABAC Policy Engine                     │    │   │
│   │  │  evaluate() | simulate() | build_obligations()     │    │   │
│   │  │  Combining: deny-overrides / permit-overrides /    │    │   │
│   │  │             first-applicable                       │    │   │
│   │  └────────────────────────────────────────────────────┘    │   │
│   │                                                             │   │
│   │  Routers: access, policies, principals, datasources,       │   │
│   │           audit, dashboard, settings, ai                   │   │
│   │                                                             │   │
│   │  Tech: FastAPI, SQLAlchemy (async), Pydantic v2            │   │
│   └──────────────────────────┬──────────────────────────────────┘   │
│                              │ SQLAlchemy async                     │
│   ┌──────────────────────────▼──────────────────────────────────┐   │
│   │                  SQLite Database                            │   │
│   │               policyguard.db (file)                        │   │
│   │                                                             │   │
│   │  Tables: policies, policy_versions, principals,            │   │
│   │          datasources, audit_logs, settings                 │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS / anthropic-sdk
                              ▼
                   ┌────────────────────┐
                   │  Anthropic Claude  │
                   │  (Haiku model)     │
                   │  External API      │
                   └────────────────────┘
```

---

## Containers

### 1. React SPA

| Property | Value |
|---|---|
| Technology | React 18 + Vite + TypeScript + Tailwind CSS |
| Port (dev) | 5176 |
| Port (prod) | 80/443 via Nginx |
| Responsibilities | Render UI, call REST API, display ABAC decisions, manage tour guide |
| Key Libraries | react-router-dom, recharts, lucide-react, clsx, axios |
| Build Output | `frontend/dist/` static bundle |

**Pages:**

| Route | Page | Purpose |
|---|---|---|
| `/` | Landing | Marketing overview + feature highlights |
| `/dashboard` | Dashboard | KPI cards, activity charts, top-denied reasons |
| `/policies` | Policy Explorer | List/filter/search all ABAC policies |
| `/policies/new` | Policy Studio | Author new policy; AI-assist via Claude |
| `/policies/:id` | Policy Detail | Read-only view with version history |
| `/policies/:id/edit` | Policy Studio (edit) | Edit existing policy |
| `/gateway` | Access Gateway | Live PDP request/response terminal |
| `/simulator` | Simulator | Dry-run with full policy match breakdown |
| `/audit` | Audit Trail | Filterable immutable access log |
| `/principals` | Principals | Identity management (Human/System/Agent) |
| `/datasources` | Data Sources | Schema catalog; live connection toggle |
| `/settings` | Settings | Combining algorithm, Anthropic key, retention |

---

### 2. FastAPI Application

| Property | Value |
|---|---|
| Technology | Python 3.12, FastAPI, SQLAlchemy (async), Pydantic v2 |
| Port | 8003 |
| ASGI Server | uvicorn |
| Responsibilities | Policy evaluation, CRUD, audit logging, AI suggestions |

**API Routers:**

| Router | Prefix | Description |
|---|---|---|
| access | `/api/access` | PDP evaluate, simulate, batch evaluate |
| policies | `/api/policies` | CRUD + state machine + versioning |
| principals | `/api/principals` | Identity CRUD + recent access |
| datasources | `/api/datasources` | Schema catalog + live sync |
| audit | `/api/audit` | Immutable log + stats |
| dashboard | `/api/dashboard` | Summary, activity, top-denied, breakdown |
| settings | `/api/settings` | Key-value config store |
| ai | `/api/ai` | Claude Haiku policy suggestion + conflict check |

**Core Module — ABAC Engine (`engine.py`):**

```
engine.evaluate(request, policies) → EvaluationResult
  ├── _match_subject(policy, subject)     — role, region, clearance, type checks
  ├── _match_resource(policy, resource)   — datasource, tables, columns, regions
  ├── _match_action(policy, action)       — SELECT/INSERT/UPDATE/DELETE/*
  ├── _match_conditions(policy, context)  — time-window, IP range, purpose checks
  ├── _apply_combining(matches, algorithm) — deny-overrides / permit-overrides / first-applicable
  └── _obligations(matching_permit)       — masked_columns, row_filter, max_rows
```

---

### 3. SQLite Database

| Property | Value |
|---|---|
| Technology | SQLite 3 via SQLAlchemy async (aiosqlite) |
| File | `backend/policyguard.db` |
| Schema migrations | None (SQLAlchemy `create_all` on startup) |
| Test DB | In-memory SQLite (session-scoped, isolated) |

**Tables:**

| Table | Key Columns | Notes |
|---|---|---|
| `policies` | id, name, effect, state, version, subject/resource/action/conditions _json | Soft-delete via `archived` state |
| `policy_versions` | id, policy_id, version, snapshot_json | Append-only version snapshots |
| `principals` | id, identifier, name, type, attributes_json, is_active | type ∈ {human, system, agent} |
| `datasources` | id, name, db_type, connection_json, schema_cache_json | connection_json fields masked in API |
| `audit_logs` | id, request_id, decision, effect, principal_id, datasource, action, policy_id, obligations_json, latency_ms, created_at | Immutable (no UPDATE/DELETE) |
| `settings` | key, value | Key-value store; combining_algorithm, anthropic_api_key, retention_days |

---

### 4. Anthropic Claude (External)

| Property | Value |
|---|---|
| Model | Claude Haiku (claude-haiku-4-5-20251001) |
| Usage | Policy JSON suggestion from plain English; conflict detection; impact analysis |
| Integration | `POST /api/ai/suggest` → `anthropic.Anthropic().messages.create()` |
| Key Storage | `settings` table, key = `anthropic_api_key` |
| Fallback | Returns `null` suggestion if key absent (graceful degradation) |

---

## Communication

| From | To | Protocol | Notes |
|---|---|---|---|
| Browser | React SPA | HTTPS (Nginx) | Static bundle served by Nginx |
| React SPA | FastAPI | HTTPS REST (`/api/*`) | Nginx proxy → localhost:8003 |
| FastAPI | SQLite | SQLAlchemy async | Local file or in-memory (tests) |
| FastAPI | Anthropic Claude | HTTPS anthropic-sdk | API key from settings table |

---

## Deployment Topology (Production)

```
Internet
    │
    ▼
 Nginx (443/80)
    ├── /* → /frontend/dist (static files)
    └── /api/* → localhost:8003 (FastAPI)
                       │
                   policyguard.db
```

See [Deployment Runbook](../deployment/runbook.md) for full setup instructions.
