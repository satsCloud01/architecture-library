---
name: "C3 – Component Diagram"
project: "DataGuardian"
project_slug: "data-guardian"
project_url: "https://data-guardian.satszone.link"
github: "https://github.com/satsCloud01/data-guardian"
category: "data-analytics"
type: "c4-component"
icon: "🛡️"
tags: [FastAPI, React, Claude API]
---

# C3 — Component Diagram

## Backend Components

```
dataguardian/
├── main.py ─────────────────── FastAPI app factory, CORS, lifespan, router mounts
├── database.py ─────────────── Async engine, SessionLocal, get_db() dependency
├── models/
│   └── core.py ─────────────── 13 SQLAlchemy ORM classes (see table below)
├── seed.py ─────────────────── Idempotent seeder, Faker data, 70KB+
└── routers/
    ├── dashboard.py ────────── GET /api/dashboard/summary, /quality-heatmap
    ├── models.py ───────────── Full CRUD: models, fields, versions, issues, audit
    ├── lineage.py ──────────── Graph, impact, link CRUD, relationship types
    ├── changes.py ──────────── Change requests + review workflow
    ├── glossary.py ─────────── Term CRUD + field linking
    ├── compliance.py ───────── Standards, mappings, controls, gap analysis
    ├── quality.py ──────────── Metrics, trends, issue CRUD
    └── ai_router.py ────────── 6 Claude-powered endpoints (with header key support)
```

### ORM Models (core.py)

| Class | Table | Key Relationships |
|-------|-------|-------------------|
| `DataModel` | `data_models` | has many `ModelVersion`, `DataIssue`, `LineageLink`, `ComplianceMapping`, `AuditLog` |
| `ModelVersion` | `model_versions` | belongs to `DataModel`; has many `DataField` |
| `DataField` | `data_fields` | belongs to `ModelVersion` |
| `LineageLink` | `lineage_links` | source → `DataModel`, target → `DataModel` |
| `ChangeRequest` | `change_requests` | belongs to `DataModel`; has many `ChangeReview` |
| `ChangeReview` | `change_reviews` | belongs to `ChangeRequest` |
| `GlossaryTerm` | `glossary_terms` | has many `FieldGlossaryLink` |
| `FieldGlossaryLink` | `field_glossary_links` | links `DataField` ↔ `GlossaryTerm` |
| `GovernanceStandard` | `governance_standards` | has many `ComplianceMapping` |
| `ComplianceMapping` | `compliance_mappings` | links `DataModel` ↔ `GovernanceStandard`; has many `ComplianceControl` |
| `ComplianceControl` | `compliance_controls` | belongs to `ComplianceMapping` |
| `QualityMetric` | `quality_metrics` | time-series metrics per `DataModel` |
| `DataIssue` | `data_issues` | belongs to `DataModel` |
| `AuditLog` | `audit_logs` | append-only log per `DataModel` |

### Router Endpoint Summary

| Router | Prefix | GET | POST | PUT | DELETE |
|--------|--------|-----|------|-----|--------|
| dashboard | `/api/dashboard` | summary, quality-heatmap | — | — | — |
| models | `/api/models` | list, get, fields, versions, audit, upload-template | create, addField, createVersion, createIssue | update, updateField | deprecate, deleteField |
| lineage | `/api/lineage` | graph, impact, relationship-types | createLink | — | deleteLink |
| changes | `/api/changes` | list, get | create, review | — | — |
| glossary | `/api/glossary` | list, get, field-links | create, link-field | — | — |
| compliance | `/api/compliance` | standards, overview, mappings, gap-analysis | createMapping | updateMapping, updateControl | deleteMapping |
| quality | `/api/quality` | metrics, model-metrics, issues, trends | createIssue | updateIssue | — |
| ai | `/api/ai` | — | recommend-glossary, validate-name, generate-description, score-documentation, summarize-schema, impact-narrative | — | — |

---

## Frontend Components

```
src/
├── App.tsx ──────────────────── BrowserRouter + ApiKeyProvider + AppLayout
├── lib/
│   ├── api.ts ───────────────── Typed fetch client; get/post/put/del helpers; 45+ methods
│   └── apiKeyContext.tsx ─────── localStorage context; useApiKey() hook
├── components/
│   ├── layout/
│   │   └── Sidebar.tsx ──────── Fixed nav (10 links + collapse)
│   ├── ui/
│   │   ├── Badge.tsx ────────── Variant-coloured chip (restricted/confidential/internal/…)
│   │   ├── Modal.tsx ────────── Centred modal with backdrop
│   │   └── StandardsGuide.tsx ── Standards reference tooltip
│   └── ApiKeyBanner.tsx ─────── Top banner: prompt + save Anthropic key (hides when set)
└── pages/
    ├── Landing.tsx ──────────── Hero + feature cards + CTA → /dashboard
    ├── Dashboard.tsx ────────── KPI summary, domain distribution, recent activity
    ├── ModelRegistry.tsx ─────── Card grid, search/filter, New Model modal, Upload JSON modal
    ├── ModelDetail.tsx ──────── 7-tab view: Schema · Versions · Issues · Compliance · Lineage · Audit · Overview
    ├── LineageExplorer.tsx ──── React Flow DAG, click-node impact, Add Link modal, click-edge delete
    ├── ChangeRequests.tsx ───── Accordion list + AI impact narrative + approve/reject
    ├── Glossary.tsx ─────────── Two-panel: term list + detail; AI recommend; link to fields
    ├── Compliance.tsx ───────── Standards overview, gap analysis, Map Model modal
    ├── DataQuality.tsx ──────── Quality cards, trend chart, bar chart, issue table + Raise Issue modal
    └── Settings.tsx ─────────── Naming conventions, tiers, model types, lifecycle, AI key config
```

### Data Flow: Model Registration

```
User → ModelRegistry.tsx (New Model button)
     → CreateModelModal (fill form + add fields)
     → api.models.create(payload)
     → POST /api/models
     → models.py: create_model()
          → INSERT data_models
          → INSERT model_versions (v1.0, is_current=true)
          → INSERT data_fields (for each field)
          → _recalc_model_counts()
          → INSERT audit_log (action=create)
     → Response: DataModel JSON
     → navigate(/models/{id})
```

### Data Flow: AI Assistance

```
User → ModelDetail.tsx (AI Describe button)
     → api.ai.generateDescription(field context)
          → reads localStorage key via getApiKey()
          → adds X-API-Key header if key present
     → POST /api/ai/generate-description  [X-API-Key: sk-ant-...]
     → ai_router.py: generate_description()
          → get_client(x_api_key)  ← prefers header key over env var
          → if no client: return mock response
          → else: Claude Haiku prompt → parse → return
```

### State Management

DataGuardian uses **local React state** only — no Redux or Zustand. Each page manages its own data with `useState` + `useEffect`. The only cross-page state is the Anthropic API key via `ApiKeyContext` (backed by `localStorage`).
