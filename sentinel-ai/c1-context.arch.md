---
name: "C1 – System Context"
project: "Sentinel AI"
project_slug: "sentinel-ai"
project_url: "https://sentinel-ai.satszone.link"
github: "https://github.com/satsCloud01/sentinel-ai"
category: "data-analytics"
type: "c4-context"
icon: "🛡️"
tags: [FastAPI, React, Claude API, React Flow]
---

# Sentinel AI — Architecture Documentation

**Data & AI Observability Platform**

This document describes the system architecture using the C4 model (Context, Container, Component, Code).

---

## C1 — System Context

```
+---------------------------------------------------+
|                  Enterprise Users                  |
|          (Data Engineers, ML Engineers,             |
|           Platform Admins, SREs)                   |
+----------------------------+-----------------------+
                             |
                        HTTPS (Browser)
                             |
                             v
              +------------------------------+
              |        Sentinel AI           |
              |  Data & AI Observability     |
              |         Platform             |
              +-----+----------------+-------+
                    |                |
                    |                |
          +---------+       +-------+---------+
          |                 |                 |
          v                 v                 v
+----------------+  +---------------+  +------------------+
| Client         |  | Anthropic     |  | External Alert   |
| Environments   |  | Claude API    |  | Channels         |
| (DBs, S3,      |  | (Insights,   |  | (Slack, PagerDuty|
|  Snowflake,    |  |  anomaly      |  |  Email, Webhook) |
|  Databricks,   |  |  detection)   |  |                  |
|  Airflow, dbt) |  +---------------+  +------------------+
+----------------+
        ^
        |
  Lightweight Agents
  (deployed in client env)
```

### Actors

| Actor | Role |
|---|---|
| **Data Engineers** | Configure monitors, manage assets, track data lineage |
| **ML Engineers** | Observe AI/LLM traces, manage data products, review circuit breakers |
| **Platform Admins** | Manage integrations, agents, settings, incident response |
| **SREs** | Monitor dashboards, respond to alerts and incidents |

### External Systems

| System | Integration |
|---|---|
| **Client Data Environments** | Lightweight agents connect to databases, file stores, Snowflake, Databricks, Airflow, dbt, Athena, AWS Data Lakes |
| **Anthropic Claude API** | Powers AI insights, anomaly detection, root cause analysis, natural-language summaries |
| **Alert Channels** | Slack, PagerDuty, email, and webhook integrations for alert delivery |

---

## C2 — Container Diagram

```
+------------------------------------------------------------------+
|                         Sentinel AI Platform                      |
|                                                                   |
|  +---------------------------+   +-----------------------------+  |
|  |     Frontend (SPA)        |   |     Backend (API)           |  |
|  |                           |   |                             |  |
|  |  React 18 + Vite          |   |  FastAPI (Python 3.12)      |  |
|  |  Tailwind CSS 3.4         |   |  Async SQLAlchemy + aiosqlite |
|  |  Recharts (charts)        |   |  13 API routers             |  |
|  |  React Flow (lineage)     |   |  16 domain models           |  |
|  |  Port 5179                |   |  Port 8007                  |  |
|  |                           |   |                             |  |
|  |  Serves: Dashboard,       |   |  Serves: REST API,          |  |
|  |  Asset Explorer, Monitors,|   |  AI insights, Agent mgmt,   |  |
|  |  Lineage Graph, AI Obs,   |   |  Seed data, WebSocket       |  |
|  |  Data Products, Incidents |   |  (future)                   |  |
|  +------------+--------------+   +-------+----------+----------+  |
|               |                          |          |             |
|          HTTP /api/*                     |          |             |
|          (proxied)                       |          |             |
|               +----------->-------------+          |             |
|                                                     |             |
|                              +----------------------+             |
|                              |                                    |
|                              v                                    |
|                   +---------------------+                         |
|                   |   SQLite Database   |                         |
|                   |   sentinel.db       |                         |
|                   |                     |                         |
|                   |   16 tables         |                         |
|                   |   Auto-seeded       |                         |
|                   +---------------------+                         |
|                                                                   |
|  +------------------------------------------------------------+  |
|  |              Agent Templates Engine                         |  |
|  |                                                             |  |
|  |  10 agent types: database, file_storage, ai_llm, dbt,      |  |
|  |  airflow, databricks, snowflake, athena, aws_datalake,      |  |
|  |  custom                                                     |  |
|  |                                                             |  |
|  |  Generates deployment-ready agent scripts with              |  |
|  |  {{PLACEHOLDER}} replacement for client configuration       |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

### Container Responsibilities

| Container | Technology | Responsibility |
|---|---|---|
| **Frontend** | React 18, Vite, Tailwind, Recharts, React Flow | Interactive UI for dashboards, asset exploration, lineage visualization, monitor management, AI observability |
| **Backend** | FastAPI, Python 3.12, SQLAlchemy (async) | REST API, business logic, AI integration, agent template generation, seed data management |
| **SQLite DB** | aiosqlite, single file | Persistent storage for all domain entities: assets, monitors, alerts, incidents, lineage, integrations |
| **Agent Templates** | Python, Jinja-style placeholders | Generate deployable agent scripts for 10 integration types; clients deploy these into their environments |

---

## C3 — Component Diagram

### Backend Components

```
+------------------------------------------------------------------+
|                     FastAPI Backend (Port 8007)                   |
|                                                                   |
|  +--------------------+  +--------------------+                   |
|  | cortex.main        |  | Middleware          |                  |
|  | (app entrypoint)   |  | - CORS (5173,5179, |                  |
|  | - lifespan mgmt    |  |   3000)             |                  |
|  | - router mounting  |  | - Error handlers    |                  |
|  +--------------------+  +--------------------+                   |
|                                                                   |
|  Routers (13)                                                     |
|  +------------------+  +------------------+  +-----------------+  |
|  | /api/dashboard   |  | /api/assets      |  | /api/monitors   |  |
|  | - summary stats  |  | - CRUD assets    |  | - CRUD monitors |  |
|  | - recent alerts  |  | - field metadata |  | - run history   |  |
|  | - trend data     |  | - asset search   |  | - trigger runs  |  |
|  +------------------+  +------------------+  +-----------------+  |
|                                                                   |
|  +------------------+  +------------------+  +-----------------+  |
|  | /api/alerts      |  | /api/incidents   |  | /api/lineage    |  |
|  | - alert list     |  | - incident mgmt  |  | - nodes & edges |  |
|  | - acknowledge    |  | - status updates |  | - impact graph  |  |
|  | - resolve        |  | - RCA tracking   |  | - upstream/down |  |
|  +------------------+  +------------------+  +-----------------+  |
|                                                                   |
|  +------------------+  +------------------+  +-----------------+  |
|  | /api/ai-         |  | /api/data-       |  | /api/insights   |  |
|  |  observability   |  |  products        |  | - AI-generated  |  |
|  | - LLM traces     |  | - product catalog|  | - anomaly detect|  |
|  | - token usage    |  | - quality scores |  | - root cause    |  |
|  | - latency stats  |  | - SLA tracking   |  | - recommendations|
|  +------------------+  +------------------+  +-----------------+  |
|                                                                   |
|  +------------------+  +------------------+  +-----------------+  |
|  | /api/circuit-    |  | /api/integrations|  | /api/settings   |  |
|  |  breakers        |  | - connections    |  | - app config    |  |
|  | - trip/reset     |  | - health checks  |  | - preferences   |  |
|  | - threshold mgmt |  | - credentials    |  | - notification  |  |
|  +------------------+  +------------------+  +-----------------+  |
|                                                                   |
|  +------------------+                                             |
|  | /api/agents      |                                             |
|  | - register agent |                                             |
|  | - generate script|                                             |
|  | - 10 agent types |                                             |
|  +------------------+                                             |
|                                                                   |
|  Models Layer (16 entities)                                       |
|  +------------------------------------------------------------+  |
|  | Domain, DataSource, Asset, AssetField, Monitor, MonitorRun, |  |
|  | Alert, Incident, LineageNode, LineageEdge, DataProduct,     |  |
|  | CircuitBreaker, AITrace, Integration, Setting,              |  |
|  | AgentRegistration                                           |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Support Layers                                                   |
|  +-------------------+  +------------------+  +----------------+  |
|  | Seed Data Layer   |  | AI Helper        |  | Database       |  |
|  | - realistic demo  |  | - Claude Haiku   |  | - async engine |  |
|  |   data on first   |  | - mock fallback  |  | - session mgmt |  |
|  |   startup         |  | - insights gen   |  | - migrations   |  |
|  +-------------------+  +------------------+  +----------------+  |
+------------------------------------------------------------------+
```

### Frontend Components

```
+------------------------------------------------------------------+
|                   React Frontend (Port 5179)                      |
|                                                                   |
|  +--------------------+  +-------------------------------------+  |
|  | App Shell           |  | State Management                   |  |
|  | - React Router      |  | - React Query (server state)       |  |
|  | - Layout + Sidebar  |  | - localStorage (API keys)          |  |
|  | - Theme (Tailwind)  |  | - Component state                  |  |
|  +--------------------+  +-------------------------------------+  |
|                                                                   |
|  Pages                                                            |
|  +----------------+ +----------------+ +------------------------+ |
|  | Landing        | | Dashboard      | | Asset Explorer         | |
|  | - hero section | | - KPI cards    | | - filterable table     | |
|  | - feature tour | | - trend charts | | - asset detail + fields| |
|  +----------------+ +----------------+ +------------------------+ |
|                                                                   |
|  +----------------+ +----------------+ +------------------------+ |
|  | Monitors       | | Alerts         | | Incidents              | |
|  | - monitor list | | - alert feed   | | - incident timeline    | |
|  | - run history  | | - ack/resolve  | | - status management    | |
|  +----------------+ +----------------+ +------------------------+ |
|                                                                   |
|  +----------------+ +----------------+ +------------------------+ |
|  | Lineage        | | AI Observability| | Data Products         | |
|  | - React Flow   | | - trace list   | | - product catalog      | |
|  | - interactive  | | - token metrics| | - quality scores       | |
|  |   graph        | | - latency      | | - SLA tracking         | |
|  +----------------+ +----------------+ +------------------------+ |
|                                                                   |
|  +----------------+ +----------------+ +------------------------+ |
|  | Circuit Breakers| | Integrations  | | Settings               | |
|  | - status panel | | - connections  | | - config management    | |
|  | - trip/reset   | | - health       | | - API key entry        | |
|  +----------------+ +----------------+ +------------------------+ |
|                                                                   |
|  +----------------+                                               |
|  | Agents         |                                               |
|  | - registration |                                               |
|  | - script gen   |                                               |
|  +----------------+                                               |
+------------------------------------------------------------------+
```

---

## C4 — Code Level

### Agent Template System

The agent template engine generates deployment-ready Python scripts for each of the 10 supported agent types. Templates use `{{PLACEHOLDER}}` syntax for client-specific configuration.

```python
# Agent Template Flow
#
# 1. User selects agent type (e.g., "snowflake")
# 2. Backend loads template for that type
# 3. Placeholders replaced with user-provided config:
#
#    {{SENTINEL_URL}}     -> https://sentinel.example.com
#    {{AGENT_TOKEN}}      -> generated auth token
#    {{CONNECTION_STRING}} -> snowflake://user:pass@account/db
#    {{POLL_INTERVAL}}    -> 300 (seconds)
#    {{AGENT_NAME}}       -> prod-snowflake-agent-01
#
# 4. Generated script returned to user for deployment

# Template structure (conceptual):
AGENT_TYPES = [
    "database",       # PostgreSQL, MySQL, SQL Server, Oracle
    "file_storage",   # S3, GCS, Azure Blob, local FS
    "ai_llm",         # LLM call tracing (OpenAI, Anthropic, etc.)
    "dbt",            # dbt Core / dbt Cloud project monitoring
    "airflow",        # Apache Airflow DAG and task monitoring
    "databricks",     # Databricks workspace, jobs, clusters
    "snowflake",      # Snowflake account, warehouse, query history
    "athena",         # AWS Athena query and catalog monitoring
    "aws_datalake",   # S3 + Glue + Lake Formation monitoring
    "custom",         # User-defined agent with custom collectors
]
```

### SentinelClient (Agent SDK)

```python
# Each generated agent script includes a SentinelClient that:
#
# class SentinelClient:
#     """Lightweight client embedded in agent scripts."""
#
#     def __init__(self, sentinel_url: str, agent_token: str):
#         self.base_url = sentinel_url
#         self.token = agent_token
#         self.session = httpx.AsyncClient(...)
#
#     async def register(self, agent_info: dict) -> str:
#         """Register this agent with the Sentinel platform."""
#         ...
#
#     async def send_heartbeat(self) -> None:
#         """Periodic heartbeat to confirm agent is alive."""
#         ...
#
#     async def report_assets(self, assets: list[dict]) -> None:
#         """Report discovered assets (tables, files, models)."""
#         ...
#
#     async def report_metrics(self, metrics: list[dict]) -> None:
#         """Report collected metrics (row counts, freshness, etc.)."""
#         ...
#
#     async def report_lineage(self, nodes: list, edges: list) -> None:
#         """Report data lineage relationships."""
#         ...
#
#     async def report_alert(self, alert: dict) -> None:
#         """Push an alert to the platform."""
#         ...
```

### AsyncSession Pattern

```python
# Database session management follows the async dependency injection pattern:
#
# database.py
# -----------
# engine = create_async_engine("sqlite+aiosqlite:///sentinel.db")
# async_session = async_sessionmaker(engine, class_=AsyncSession)
#
# async def get_db() -> AsyncGenerator[AsyncSession, None]:
#     async with async_session() as session:
#         try:
#             yield session
#             await session.commit()
#         except Exception:
#             await session.rollback()
#             raise
#
# Router usage:
# @router.get("/assets")
# async def list_assets(db: AsyncSession = Depends(get_db)):
#     result = await db.execute(select(Asset).order_by(Asset.name))
#     return result.scalars().all()
```

### AI Helper Pattern

```python
# ai_helper.py
# ------------
# The AI helper wraps Claude API calls with graceful mock fallback.
#
# async def generate_insight(
#     context: dict,
#     api_key: str | None = None
# ) -> dict:
#     if not api_key:
#         return _mock_insight(context)  # Deterministic fallback
#
#     client = anthropic.AsyncAnthropic(api_key=api_key)
#     response = await client.messages.create(
#         model="claude-haiku-...",
#         messages=[...],
#         max_tokens=1024,
#     )
#     return _parse_insight(response)
#
# API key flow:
# 1. User enters key in Settings UI -> stored in browser localStorage
# 2. Frontend sends key via X-Api-Key header on AI-powered requests
# 3. Backend extracts from header -> passes to AI helper
# 4. Never persisted server-side
```

---

## Deployment Architecture

```
+---------------------+       +-------------------------+
|   Developer Machine |       |  AWS Consolidated       |
|                     |       |  Instance (t3.medium)   |
|  Frontend :5179 ----|------>|  Docker :8007 (backend) |
|  Backend  :8007     |       |  Nginx reverse proxy    |
|  SQLite sentinel.db |       |  SSL (Let's Encrypt)    |
+---------------------+       +-------------------------+
                                        |
                                        v
                              sentinel-ai.satszone.link
```

### Local Development

```bash
# Backend
cd backend && PYTHONPATH=src .venv/bin/uvicorn sentinel.main:app --reload --port 8007

# Frontend
cd frontend && npm run dev   # Port 5179, proxies /api -> :8007
```

### Production (Docker)

```bash
# On consolidated AWS instance
cd ~/apps/sentinel-ai
git pull
sudo docker compose build sentinel-ai
sudo docker compose up -d sentinel-ai
```
