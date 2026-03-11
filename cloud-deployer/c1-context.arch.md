---
name: "C1 – System Context"
project: "Universal Cloud Deployer"
project_slug: "cloud-deployer"
project_url: "https://cloud-deployer.satszone.link"
github: "https://github.com/satsCloud01/cloud-deployer"
category: "architecture-devops"
type: "c4-context"
icon: "☁️"
tags: [FastAPI, React, AWS, GCP, Azure]
---

# Universal Cloud Deployer — Architecture Documentation

> Version 1.0 · Last updated 2026-03-06

---

## Table of Contents

1. [C1 — System Context](#c1--system-context)
2. [C2 — Container Diagram](#c2--container-diagram)
3. [C3 — Component Diagram (Backend)](#c3--component-diagram-backend)
4. [C4 — Code / Class Diagram](#c4--code--class-diagram)
5. [Domain Model](#domain-model)
6. [API Specification](#api-specification)
7. [Constraints & Non-Functional Requirements](#constraints--non-functional-requirements)

---

## C1 — System Context

The **Universal Cloud Deployer** is a web-based platform that allows engineering teams to deploy, manage, and monitor cloud applications across AWS, GCP, and Azure from a single UI, using a Bring-Your-Own-Credentials (BYOC) model.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         External Actors & Systems                           │
│                                                                             │
│   👤 DevOps Engineer          🤖 Claude AI (Anthropic)                      │
│   (provides credentials,      (analyzes NFR profiles, suggests              │
│    triggers deploys)          deployment configs via BYOK API key)          │
│                                                                             │
│   📦 GitHub (REST API)        ☁️  Cloud Providers                           │
│   (repo inspection, branch    ┌─────────────────────────────────────┐      │
│    listing, commit polling)   │  AWS  ·  Google Cloud  ·  Azure     │      │
│                               │  EC2/ECS/Lambda, Cloud Run/GCE/GCF, │      │
│                               │  ACI/AVM/Azure Functions            │      │
│                               └─────────────────────────────────────┘      │
│                                                                             │
│                    ┌──────────────────────────────┐                         │
│                    │  Universal Cloud Deployer     │                         │
│                    │  (this system)                │                         │
│                    │  localhost:5177 (UI)           │                         │
│                    │  localhost:8004 (API)          │                         │
│                    └──────────────────────────────┘                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Security Boundary

- **Cloud credentials** never cross into persistent storage. They travel from the browser (React state) to the FastAPI endpoint in the POST body, are used immediately for cloud SDK calls, and are discarded when the async handler returns.
- **AI keys** (Anthropic, GitHub PAT) are stored in browser localStorage only — never sent to the backend database.

---

## C2 — Container Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                      Universal Cloud Deployer                            │
│                                                                          │
│  ┌─────────────────────────────────────────────────────┐                │
│  │           React Frontend (Vite · TypeScript)        │                │
│  │           Port 5177                                 │                │
│  │                                                     │                │
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────────┐  │                │
│  │  │ Context  │  │  12 Pages    │  │  Components  │  │                │
│  │  │ Providers│  │  (routing)   │  │  (Sidebar,   │  │                │
│  │  │ - ApiKey │  │  Dashboard   │  │   Wizard,    │  │                │
│  │  │ - GitHub │  │  Deploy      │  │   Modals)    │  │                │
│  │  │ - Creds  │  │  FinOps etc. │  │              │  │                │
│  │  └──────────┘  └──────────────┘  └──────────────┘  │                │
│  │                                                     │                │
│  │  lib/api.ts ──── typed fetch wrapper ───────────────┼──────────────►│
│  └─────────────────────────────────────────────────────┘               │
│                              │ /api/* (proxy)                           │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────┐               │
│  │           FastAPI Backend (Python 3.12)             │               │
│  │           Port 8004                                 │               │
│  │                                                     │               │
│  │  Routers: dashboard · deployments · execute         │               │
│  │           credentials · finops · pricing            │               │
│  │           converter · scheduler · github            │               │
│  │           ai_assist                                 │               │
│  │                                                     │               │
│  │  Services: aws_executor · gcp_executor              │               │
│  │            azure_executor · cloud_dispatcher        │               │
│  │            pricing_fetcher · converter              │               │
│  │            github_client                            │               │
│  │                                                     │               │
│  │  APScheduler: pricing refresh (6h) · GitHub poll    │               │
│  │               (5min)                                │               │
│  └─────────────────────────────────────────────────────┘               │
│                              │                                           │
│  ┌───────────────────────────▼──────────────────────────┐              │
│  │  SQLite Database (clouddeployer.db)                  │              │
│  │                                                      │              │
│  │  Tables: deployed_apps · deployment_logs             │              │
│  │          schedules · pricing_cache                   │              │
│  │          nfr_profiles · deployment_templates         │              │
│  └──────────────────────────────────────────────────────┘              │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## C3 — Component Diagram (Backend)

```
clouddeployer/
│
├── main.py                      # FastAPI app + lifespan + APScheduler wiring
│   └── @asynccontextmanager lifespan()
│       ├── init_db()            # create tables
│       ├── scheduler.add_job(refresh_pricing_cache, every 6h)
│       └── scheduler.add_job(poll_github_triggers, every 5min)
│
├── database.py                  # async SQLAlchemy engine (aiosqlite)
│   ├── get_db()                 # AsyncSession dependency
│   └── init_db()                # create_all
│
├── models/core.py               # ORM models (SQLAlchemy 2.0 mapped_column)
│   ├── DeployedApp
│   ├── DeploymentLog
│   ├── Schedule
│   ├── PricingCache
│   ├── NFRProfile
│   └── DeploymentTemplate
│
├── schemas/core.py              # Pydantic v2 schemas for request/response
│
├── routers/                     # FastAPI APIRouter instances
│   ├── dashboard.py             # GET /api/dashboard/summary
│   ├── deployments.py           # CRUD /api/apps + /api/templates
│   ├── execute.py               # POST /api/execute/{deploy|start|stop|destroy}
│   ├── credentials.py           # POST /api/credentials/validate + required-permissions
│   ├── finops.py                # GET /api/finops/free-tiers + NFR CRUD + analyze
│   ├── pricing.py               # GET /api/pricing/{provider}/{type}/{region} + cache
│   ├── converter.py             # POST /api/converter/generate + GET /runtimes
│   ├── scheduler.py             # CRUD /api/schedules + toggle
│   ├── github.py                # GET repos/branches/commits (BYOK PAT)
│   └── ai_assist.py             # POST /api/ai/deploy-advice + convert-advice (BYOK)
│
├── services/
│   ├── cloud_dispatcher.py      # dispatch(action, provider, app_id, creds, db)
│   ├── aws_executor.py          # deploy/start/stop/destroy via boto3 (per-call session)
│   ├── gcp_executor.py          # deploy/start/stop/destroy via google-cloud-* SDKs
│   ├── azure_executor.py        # deploy/start/stop/destroy via azure-mgmt-* SDKs
│   ├── pricing_fetcher.py       # fetch_pricing() + fetch_free_tier_data()
│   ├── converter.py             # generate_artifacts(request) → list of files
│   └── github_client.py         # list_repos(), list_branches(), get_latest_commit()
│
└── jobs.py                      # refresh_pricing_cache(), poll_github_triggers()
```

---

## C4 — Code / Class Diagram

### ORM Models

```
DeployedApp
├── id: int (PK)
├── name: str
├── provider: str              # aws | gcp | azure
├── deployment_type: str       # vm | container | serverless
├── status: str                # pending | running | stopped | failed | destroyed
├── region: str
├── resource_ids_json: JSON    # e.g. {"instance_id": "i-xxx"} — cloud handles only
├── config_json: JSON          # instance_type, env_vars, tags, etc.
├── github_repo: str?
├── github_branch: str?
├── created_at: datetime
└── updated_at: datetime

DeploymentLog
├── id: int (PK)
├── app_id: int (FK → DeployedApp, CASCADE DELETE)
├── action: str                # deploy | start | stop | destroy
├── status: str                # pending | running | success | failure
├── message: str
└── timestamp: datetime

Schedule
├── id: int (PK)
├── app_id: int (FK → DeployedApp)
├── trigger_type: str          # cron | github
├── cron_expr: str?            # e.g. "0 3 * * *"
├── github_repo: str?
├── github_branch: str?
├── last_commit_sha: str?
├── enabled: bool
└── created_at: datetime

PricingCache
├── id: int (PK)
├── provider: str
├── service_type: str
├── region: str
├── data_json: JSON
└── fetched_at: datetime       # TTL = 24h

NFRProfile
├── id: int (PK)
├── name: str
├── requirements_json: JSON    # rps, sla, data_volume_gb, etc.
├── ai_recommendation_json: JSON?
└── created_at: datetime

DeploymentTemplate
├── id: int (PK)
├── name: str
├── provider: str
├── deployment_type: str
├── config_json: JSON
└── created_at: datetime
```

### Key Service Interactions

```
Frontend                    FastAPI                     Cloud SDK
   │                           │                            │
   │  POST /api/execute/deploy  │                            │
   │  body: {app_id, provider,  │                            │
   │          credentials}      │                            │
   │──────────────────────────►│                            │
   │                           │  dispatch("deploy", ...)   │
   │                           │──────────────────────────►│
   │                           │    boto3.Session(creds)    │
   │                           │    run_instances(...)      │
   │                           │◄──────────────────────────│
   │                           │  write DeploymentLog       │
   │                           │  (NO credentials written)  │
   │                           │  update app.status         │
   │◄──────────────────────────│                            │
   │  {status, message,        │                            │
   │   resource_ids}           │  creds → GC'd              │
```

---

## Domain Model

```
┌─────────────────┐       ┌──────────────────┐
│  DeployedApp    │ 1──* │  DeploymentLog   │
│─────────────────│       │──────────────────│
│ name            │       │ action           │
│ provider        │       │ status           │
│ deployment_type │       │ message          │
│ status          │       │ timestamp        │
│ region          │       └──────────────────┘
│ resource_ids    │
│ config          │       ┌──────────────────┐
│ github_repo     │ 1──* │  Schedule        │
└─────────────────┘       │──────────────────│
                          │ trigger_type     │
                          │ cron_expr        │
                          │ github_repo      │
                          │ last_commit_sha  │
                          │ enabled          │
                          └──────────────────┘

┌─────────────────┐       ┌──────────────────┐
│  NFRProfile     │       │  PricingCache    │
│─────────────────│       │──────────────────│
│ name            │       │ provider         │
│ requirements    │       │ service_type     │
│ ai_recommend    │       │ region           │
└─────────────────┘       │ data_json        │
                          │ fetched_at       │
┌─────────────────┐       └──────────────────┘
│  Deployment     │
│  Template       │
│─────────────────│
│ name            │
│ provider        │
│ deployment_type │
│ config          │
└─────────────────┘
```

---

## API Specification

### Base URL
- Development: `http://localhost:8004`
- Production: `https://cloud-deployer.satszone.link`

### Authentication
No server-side auth — BYOC/BYOK model. Credentials travel per-request.

### Endpoints

#### Health
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Returns `{status, version, scheduler}` |

#### Dashboard
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/dashboard/summary` | Stats: total_apps, running, costs, provider/type/status breakdown, recent_logs |

#### Deployments (App Registry)
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/apps` | List all deployed apps |
| POST | `/api/apps` | Create app record |
| GET | `/api/apps/{id}` | Get single app |
| PUT | `/api/apps/{id}` | Update app |
| DELETE | `/api/apps/{id}` | Delete app + cascade logs |
| GET | `/api/apps/{id}/logs` | Get deployment logs |
| GET | `/api/templates` | List built-in deployment templates |

#### Execute (Deploy Operations)
All operations accept `credentials` in the request body — never stored.

| Method | Path | Body | Description |
|--------|------|------|-------------|
| POST | `/api/execute/deploy` | `{app_id, provider, credentials}` | Deploy app to cloud |
| POST | `/api/execute/start` | `{app_id, provider, credentials}` | Start stopped app |
| POST | `/api/execute/stop` | `{app_id, provider, credentials}` | Stop running app |
| POST | `/api/execute/destroy` | `{app_id, provider, credentials}` | Permanently destroy |

#### Credentials
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/credentials/validate` | Validate cloud credentials (returns identity, never stored) |
| GET | `/api/credentials/required-permissions/{provider}/{type}` | List required IAM permissions |

#### FinOps
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/finops/free-tiers` | Free tier comparison: all 3 providers |
| GET | `/api/finops/nfr-profiles` | List NFR profiles |
| POST | `/api/finops/nfr-profiles` | Create NFR profile |
| GET | `/api/finops/nfr-profiles/{id}` | Get profile |
| DELETE | `/api/finops/nfr-profiles/{id}` | Delete profile |
| POST | `/api/finops/nfr-profiles/{id}/analyze` | AI analyze (BYOK: `{api_key}`) |

#### Pricing
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/pricing/{provider}/{service_type}/{region}` | Fetch pricing with 24h cache |
| GET | `/api/pricing/free-tiers/{provider}` | Free tier data for provider |
| GET | `/api/pricing/compare/all` | Side-by-side free tier comparison |

#### Converter
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/converter/runtimes` | Supported runtime metadata |
| POST | `/api/converter/generate` | Generate Dockerfile/handler/startup artifacts |

#### Scheduler
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/schedules` | List schedules |
| POST | `/api/schedules` | Create schedule (cron or github trigger) |
| GET | `/api/schedules/{id}` | Get schedule |
| DELETE | `/api/schedules/{id}` | Delete schedule |
| PATCH | `/api/schedules/{id}/toggle` | Enable/disable |

#### GitHub Integration
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/github/repos` | List user repos (BYOK PAT: `?token=`) |
| GET | `/api/github/repos/{owner}/{repo}/branches` | List branches |
| GET | `/api/github/repos/{owner}/{repo}/commits/{branch}` | Latest commit SHA |

#### AI Assist
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/ai/deploy-advice` | Claude: deployment recommendation (BYOK) |
| POST | `/api/ai/convert-advice` | Claude: conversion strategy advice (BYOK) |

---

## Constraints & Non-Functional Requirements

### Security Constraints

| ID | Constraint | Implementation |
|----|-----------|----------------|
| SEC-01 | Cloud credentials MUST NOT be persisted | Credentials accepted per-request only; no DB fields for secrets |
| SEC-02 | AI/GitHub keys MUST NOT be stored server-side | Keys passed in request body; backend never writes them |
| SEC-03 | No plaintext secrets in logs | Executors log resource IDs only; credentials never appear in DeploymentLog |
| SEC-04 | boto3/GCP/Azure sessions created per-call | No module-level SDK client with cached credentials |
| SEC-05 | Frontend cloud creds in React state only | CredentialContext never calls `localStorage.setItem` |
| SEC-06 | AI keys in localStorage with user control | Settings page allows clear; per-request modal prompt if missing |

### Availability & Performance

| ID | Constraint | Target |
|----|-----------|--------|
| PERF-01 | Pricing data latency | Served from SQLite cache (<5ms); fresh fetch only when >24h old |
| PERF-02 | Background pricing refresh | APScheduler cron every 6h — non-blocking |
| PERF-03 | GitHub poll for triggers | Every 5 minutes via APScheduler |
| PERF-04 | API response time (non-cloud) | <100ms for DB reads |
| PERF-05 | Cloud operation timeout | Up to 60s for real cloud SDK calls |

### Scalability

| ID | Constraint |
|----|-----------|
| SCALE-01 | SQLite is sufficient for single-operator use; migrate to PostgreSQL for team use |
| SCALE-02 | APScheduler runs in-process; for multi-instance deploy, replace with distributed task queue (Celery/Redis) |
| SCALE-03 | Cloud executor calls are I/O-bound; FastAPI async handles concurrent deploys |

### Compatibility

| ID | Constraint |
|----|-----------|
| COMPAT-01 | Python 3.12+ required (f-string syntax, match statements) |
| COMPAT-02 | Node.js 18+ for frontend build |
| COMPAT-03 | Browser: Chrome/Firefox/Safari latest two versions |
| COMPAT-04 | Cloud providers: AWS (us-east-1 default), GCP (us-central1), Azure (eastus) |

### Data Constraints

| ID | Constraint |
|----|-----------|
| DATA-01 | PricingCache TTL = 24 hours |
| DATA-02 | Cascade delete: deleting a DeployedApp removes all its DeploymentLogs and Schedules |
| DATA-03 | NFRProfile.ai_recommendation_json is nullable until analyzed |
| DATA-04 | Schedule.last_commit_sha tracked to prevent duplicate trigger fires |

### Operational

| ID | Constraint |
|----|-----------|
| OPS-01 | Single `./start.sh` boots both backend and frontend |
| OPS-02 | Backend auto-creates SQLite DB and tables on first run (`init_db()`) |
| OPS-03 | No migrations tooling — tables are recreated on schema changes (dev only) |
| OPS-04 | Graceful APScheduler shutdown on process exit via lifespan context manager |
