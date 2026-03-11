# AgentKraft — Architecture Documentation

> Version 1.0 · Last updated 2026-03-11

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

**AgentKraft** is a web-based platform that enables business users and developers to design, build, deploy, and manage AI agents through a guided no-code/low-code interface. It translates business requirements into fully configured agent packages with MCP and A2A protocol support.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         External Actors & Systems                           │
│                                                                             │
│   👤 Business User              🤖 AI Providers (BYOK)                     │
│   (answers guided questions,     (Anthropic Claude, OpenAI GPT,             │
│    reviews blueprint, deploys)    Google Gemini, Mistral — keys in          │
│                                   browser localStorage only)               │
│   👤 Developer                                                              │
│   (uses Agent Builder chat,     📦 Docker / Kubernetes / Serverless        │
│    customizes generated code)    (deployment targets for generated agents)  │
│                                                                             │
│                    ┌──────────────────────────────┐                         │
│                    │        AgentKraft             │                         │
│                    │        (this system)          │                         │
│                    │  Frontend: localhost:5180      │                         │
│                    │  Backend:  localhost:8006      │                         │
│                    └──────────────────────────────┘                         │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Security Boundary

- **AI API keys** are stored in browser localStorage only (`ak_api_key`, `ak_provider`). They are passed to the backend via `X-Api-Key` header per request and never persisted server-side.
- **No credentials at rest** — the backend never writes API keys to the database or filesystem.

---

## C2 — Container Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                            AgentKraft                                    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────┐                │
│  │           React Frontend (Vite · JSX)               │                │
│  │           Port 5180                                  │                │
│  │                                                     │                │
│  │  ┌──────────┐  ┌──────────────┐  ┌──────────────┐  │                │
│  │  │ BYOK     │  │  14 Pages    │  │  Components  │  │                │
│  │  │ Storage  │  │  Landing     │  │  Layout      │  │                │
│  │  │ (local-  │  │  Dashboard   │  │  Sidebar     │  │                │
│  │  │  Storage)│  │  Decision    │  │  Tour        │  │                │
│  │  │          │  │  Builder etc │  │              │  │                │
│  │  └──────────┘  └──────────────┘  └──────────────┘  │                │
│  │                                                     │                │
│  │  lib/api.js ──── fetch wrapper + BYOK headers ─────┼──────────────► │
│  └─────────────────────────────────────────────────────┘               │
│                              │ /api/* (Vite proxy)                     │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────┐               │
│  │           FastAPI Backend (Python 3.12)             │               │
│  │           Port 8006                                 │               │
│  │                                                     │               │
│  │  Routers: auth · projects · decisions ·             │               │
│  │           agent_builder · artifacts · deploy ·      │               │
│  │           registry · templates · marketplace ·      │               │
│  │           settings                                  │               │
│  │                                                     │               │
│  │  Services: agent_codegen · mcp_generator ·          │               │
│  │            a2a_generator · doc_generator ·           │               │
│  │            export_service                            │               │
│  │                                                     │               │
│  │  AI Helper: multi-provider LLM (mock fallback)      │               │
│  └──────────────────────────┬──────────────────────────┘               │
│                              │                                          │
│                              ▼                                          │
│  ┌─────────────────────────────────────────────────────┐               │
│  │           SQLite (agentkraft.db)                    │               │
│  │           17 tables · async via aiosqlite           │               │
│  └─────────────────────────────────────────────────────┘               │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## C3 — Component Diagram (Backend)

```
┌──────────────────────────────────────────────────────────────────────┐
│                       FastAPI Application                            │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                        API Routers (10)                        │  │
│  │                                                                │  │
│  │  /api/auth          /api/projects        /api/decisions        │  │
│  │  Login, RBAC        CRUD, phases,        Guided questions,     │  │
│  │                     gates, advance       impact analysis,      │  │
│  │                                          blueprint derivation  │  │
│  │                                                                │  │
│  │  /api/agent-builder /api/artifacts       /api/deploy           │  │
│  │  Chat, generate,    CRUD, versioning,    Package, download,    │  │
│  │  preview, export    AI-generate          launch (3 targets)    │  │
│  │                                                                │  │
│  │  /api/registry      /api/templates       /api/marketplace      │  │
│  │  List, stats,       CRUD, use-template   List, publish, fork,  │  │
│  │  heartbeat, logs    (creates project)    rate, download        │  │
│  │                                                                │  │
│  │  /api/settings                                                 │  │
│  │  Profile, providers, key validation                            │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                              │                                        │
│                              ▼                                        │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                      Service Layer (6)                         │  │
│  │                                                                │  │
│  │  agent_codegen      mcp_generator       a2a_generator          │  │
│  │  AgentSpec →        MCP server with     A2A handler with       │  │
│  │  15+ file Python    tool registration   agent card, task       │  │
│  │  package            & manifest          lifecycle              │  │
│  │                                                                │  │
│  │  doc_generator      export_service                             │  │
│  │  README, ARCH,      ZIP creation,                              │  │
│  │  API, DEPLOY docs   Docker/K8s/Lambda                          │  │
│  │                     deploy configs                             │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                              │                                        │
│                              ▼                                        │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                    AI Helper Module                             │  │
│  │                                                                │  │
│  │  GUIDED_QUERIES (7 business questions)                         │  │
│  │  _derive_blueprint() — answers → agent spec                   │  │
│  │  _impact_for_answer() — per-question business impact           │  │
│  │  call_llm() / call_llm_json() — multi-provider + mock         │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                              │                                        │
│                              ▼                                        │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │               SQLAlchemy Models (17 tables)                    │  │
│  │  Tenant · User · Project · Phase · Artifact · ArtifactVersion │  │
│  │  GateReview · DecisionSession · DecisionQuery · ImpactAnalysis│  │
│  │  AgentBuilderSession · AgentBuilderMessage · DeployedAgent    │  │
│  │  AgentLog · AgentMetric · BlueprintListing · Template         │  │
│  │  Setting                                                      │  │
│  └────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## C4 — Code / Class Diagram

### Decision Intelligence Flow

```
DecisionStudio.jsx                  decisions.py router              ai_helper.py
─────────────────                   ──────────────────               ────────────
startSession()                      POST /sessions
  │                                   → DecisionSession(project_id)
  ▼
generateQueries()                   POST /sessions/{id}/generate-queries
  │                                   → GUIDED_QUERIES (7 questions)
  │                                   → DecisionQuery rows created
  ▼
submitAnswer(queryId, answer)       POST /sessions/{id}/queries/{qid}/answer
  │                                   → q.answer = answer            _impact_for_answer()
  │                                   → ImpactAnalysis created         category → impact dict
  ▼
analyzeAll()                        POST /sessions/{id}/analyze
  │                                   → answers = {cat: answer}      _derive_blueprint()
  │                                   → Project updated with            answers → {
  │                                     agent_type, framework,            agent_type,
  │                                     config.tools,                     framework,
  │                                     config.capabilities               tools[],
  ▼                                                                       capabilities[],
Blueprint displayed                                                       use_mcp, use_a2a
                                                                        }
```

### Agent Code Generation

```
AgentSpec (Pydantic)
  name: str
  description: str
  tools: list[str]
  framework: str          ──►  generate_agent_package(spec) ──► dict[path, content]
  use_mcp: bool                  │
  use_a2a: bool                  ├── {name}/agent.py         (ReAct or Plan-Execute loop)
                                 ├── {name}/tools.py         (tool implementations)
                                 ├── {name}/config.py        (AgentConfig dataclass)
                                 ├── {name}/prompts.py       (system prompts)
                                 ├── {name}/__init__.py
                                 ├── {name}/__main__.py
                                 ├── mcp_server.py           (if use_mcp)
                                 ├── a2a_handler.py           (if use_a2a)
                                 ├── requirements.txt
                                 ├── Dockerfile
                                 ├── README.md
                                 ├── ARCHITECTURE.md
                                 ├── API.md
                                 └── DEPLOYMENT.md
```

---

## Domain Model

| Entity | Key Attributes | Relationships |
|--------|---------------|---------------|
| **Tenant** | id, name, slug | has many Users, Projects |
| **User** | id, email, name, role | belongs to Tenant |
| **Project** | id, name, description, agent_type, framework, use_mcp, use_a2a, config (JSON) | belongs to Tenant; has many Phases, Artifacts, GateReviews |
| **Phase** | id, name, status, order | belongs to Project |
| **Artifact** | id, name, type, content | belongs to Project; has many ArtifactVersions |
| **ArtifactVersion** | id, version, content, created_at | belongs to Artifact |
| **GateReview** | id, phase_name, status, reviewer, notes | belongs to Project |
| **DecisionSession** | id, project_id, status | belongs to Project; has many DecisionQueries |
| **DecisionQuery** | id, question, category, options (JSON), answer, order | belongs to DecisionSession; has one ImpactAnalysis |
| **ImpactAnalysis** | id, risk_level, complexity, explanation, trade_offs (JSON) | belongs to DecisionQuery |
| **AgentBuilderSession** | id, project_id, status, agent_spec (JSON) | belongs to Project; has many Messages |
| **AgentBuilderMessage** | id, role, content | belongs to AgentBuilderSession |
| **DeployedAgent** | id, project_id, name, status, target, url, config (JSON) | belongs to Project |
| **AgentLog** | id, agent_id, level, message, timestamp | belongs to DeployedAgent |
| **AgentMetric** | id, agent_id, metric_name, value, timestamp | belongs to DeployedAgent |
| **BlueprintListing** | id, name, description, category, author, rating, downloads, config (JSON) | standalone (marketplace) |
| **Template** | id, name, description, category, config (JSON) | standalone |
| **Setting** | id, key, value | standalone |

### Entity Relationship Diagram

```
Tenant 1──* User
Tenant 1──* Project
Project 1──* Phase
Project 1──* Artifact 1──* ArtifactVersion
Project 1──* GateReview
Project 1──* DecisionSession 1──* DecisionQuery 1──1 ImpactAnalysis
Project 1──* AgentBuilderSession 1──* AgentBuilderMessage
Project 1──* DeployedAgent 1──* AgentLog
                            1──* AgentMetric
```

---

## API Specification

### Authentication (`/api/auth`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| POST | `/api/auth/login` | Login with email/password | 200 |
| GET | `/api/auth/me` | Get current user | 200 |
| GET | `/api/auth/tenants` | List tenants | 200 |

### Projects (`/api/projects`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/projects` | List all projects | 200 |
| POST | `/api/projects` | Create project | 201 |
| GET | `/api/projects/{id}` | Get project detail | 200 |
| PUT | `/api/projects/{id}` | Update project | 200 |
| DELETE | `/api/projects/{id}` | Delete project (cascades) | 204 |
| GET | `/api/projects/{id}/phases` | List phases | 200 |
| GET | `/api/projects/{id}/gates` | List gate reviews | 200 |
| POST | `/api/projects/{id}/advance` | Advance to next phase | 200 |

### Decision Intelligence (`/api/decisions`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| POST | `/api/decisions/sessions` | Start guided session | 200 |
| GET | `/api/decisions/sessions/{id}` | Get session | 200 |
| POST | `/api/decisions/sessions/{id}/generate-queries` | Generate 7 business questions | 200 |
| POST | `/api/decisions/sessions/{id}/queries/{qid}/answer` | Submit answer + get impact | 200 |
| POST | `/api/decisions/sessions/{id}/analyze` | Derive agent blueprint from all answers | 200 |

### Agent Builder (`/api/agent-builder`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| POST | `/api/agent-builder/sessions` | Create builder session | 200 |
| GET | `/api/agent-builder/sessions/{id}` | Get session + spec | 200 |
| POST | `/api/agent-builder/sessions/{id}/chat` | Chat message | 200 |
| GET | `/api/agent-builder/sessions/{id}/messages` | List messages | 200 |
| POST | `/api/agent-builder/sessions/{id}/generate` | Generate agent package | 200 |
| GET | `/api/agent-builder/projects/{pid}/preview` | Preview generated files | 200 |
| GET | `/api/agent-builder/sessions/{id}/export` | Download agent ZIP | 200 |

### Artifacts (`/api/artifacts`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/artifacts/projects/{pid}` | List artifacts | 200 |
| POST | `/api/artifacts/projects/{pid}` | Create artifact | 201 |
| PUT | `/api/artifacts/{id}` | Update artifact (creates version) | 200 |
| DELETE | `/api/artifacts/{id}` | Delete artifact | 204 |
| GET | `/api/artifacts/{id}/versions` | List versions | 200 |
| POST | `/api/artifacts/{id}/ai-generate` | AI-generate content | 200 |

### Deploy (`/api/deploy`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/deploy/targets` | List deploy targets | 200 |
| POST | `/api/deploy/package` | Create deploy package | 200 |
| GET | `/api/deploy/download/{pid}` | Download deploy ZIP | 200 |
| POST | `/api/deploy/launch` | Launch deployment | 200 |

### Registry (`/api/registry`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/registry/agents` | List deployed agents | 200 |
| GET | `/api/registry/stats` | Aggregate statistics | 200 |
| GET | `/api/registry/agents/{id}` | Agent detail | 200 |
| PUT | `/api/registry/agents/{id}/status` | Update status | 200 |
| POST | `/api/registry/agents/{id}/heartbeat` | Record heartbeat | 200 |
| GET | `/api/registry/agents/{id}/logs` | Get logs | 200 |
| GET | `/api/registry/agents/{id}/metrics` | Get metrics | 200 |

### Templates (`/api/templates`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/templates` | List templates | 200 |
| POST | `/api/templates` | Create template | 201 |
| GET | `/api/templates/{id}` | Get template | 200 |
| POST | `/api/templates/{id}/use` | Create project from template | 201 |
| DELETE | `/api/templates/{id}` | Delete template | 204 |

### Marketplace (`/api/marketplace`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/marketplace/blueprints` | List blueprints | 200 |
| POST | `/api/marketplace/publish` | Publish blueprint | 201 |
| GET | `/api/marketplace/blueprints/{id}/download` | Download blueprint | 200 |
| POST | `/api/marketplace/blueprints/{id}/rate` | Rate blueprint | 200 |
| POST | `/api/marketplace/blueprints/{id}/fork` | Fork blueprint as project | 201 |

### Settings (`/api/settings`)

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/settings` | List all settings | 200 |
| PUT | `/api/settings/{key}` | Update setting | 200 |
| GET | `/api/settings/providers` | List AI providers | 200 |
| POST | `/api/settings/validate-key` | Validate API key | 200 |
| GET | `/api/settings/profile` | Get user profile | 200 |

---

## Constraints & Non-Functional Requirements

| ID | Category | Constraint |
|----|----------|-----------|
| NFR-01 | Security | AI API keys stored in browser localStorage only; passed via X-Api-Key header; never persisted server-side |
| NFR-02 | Security | No server-side secrets required — app runs fully with mock AI fallback |
| NFR-03 | Privacy | All data stays local (SQLite file); no telemetry or external data sharing |
| NFR-04 | Performance | SQLite + async (aiosqlite) supports ~100 concurrent users |
| NFR-05 | Performance | Agent code generation completes in < 500ms (template-based, no AI call) |
| NFR-06 | Availability | Stateless backend — horizontal scaling via Docker replicas |
| NFR-07 | Portability | Docker multi-stage build; runs on any Linux/macOS/Windows with Docker |
| NFR-08 | Compatibility | Generated agents are standalone Python packages — no AgentKraft runtime dependency |
| NFR-09 | Protocols | Generated agents include MCP server (tool registration, manifest, request handling) |
| NFR-10 | Protocols | Generated agents include A2A handler (Google Agent-to-Agent spec: agent card, task lifecycle) |
| NFR-11 | Extensibility | Tool registry is additive — new tools added without modifying existing code |
| NFR-12 | UX | Business-first guided interview (7 questions) — no technical knowledge required |
| NFR-13 | UX | Dark theme (slate-950 bg, brand-600 indigo accent) matching AI Guardian design system |
| NFR-14 | Testing | 82+ backend tests covering all routers and services |
| NFR-15 | Deployment | Three export targets: Docker, Kubernetes (Helm), Serverless (AWS Lambda) |
| NFR-16 | AI Provider | Multi-provider: Anthropic, OpenAI, Google, Mistral — graceful mock fallback without keys |
