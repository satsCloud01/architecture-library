---
name: "C3 – Component Diagram"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "c4-component"
icon: "🏗️"
tags: [FastAPI, React, Claude API]
---

# C3 — Component Diagram

## Backend Components

```
archsmith/
├── main.py               App factory + CORS + router mounts
├── database.py           Engine, SessionLocal, get_db()
├── models.py             14 ORM classes
├── schemas.py            Pydantic models (request + response)
├── security.py           RequestContext, get_request_context, require_reviewer_or_admin
├── routers/
│   ├── architecture.py   FR1–FR6 endpoints (7 routes)
│   ├── ai.py             POST /ai/suggest
│   └── examples.py       GET /examples, GET /examples/{id}
└── services/
    ├── artifact_generator.py   8 generator functions + generate_artifacts()
    ├── impact.py               run_impact_analysis()
    └── ai_suggest.py           get_ai_suggestions() → Claude Haiku
```

### ORM Models (models.py)

| Model | Table | Key Fields |
|-------|-------|-----------|
| `ArchitecturePackage` | `architecture_packages` | tenant_id, name, status, version, approved |
| `Requirement` | `requirements` | key, category (FR/NFR), text, priority |
| `Domain` | `domains` | name, description |
| `BoundedContext` | `bounded_contexts` | domain_name, name |
| `Service` | `services` | bounded_context_name, name, service_type |
| `API` | `apis` | service_name, name, method, path |
| `Event` | `events` | name, producer_service |
| `DataEntity` | `data_entities` | name, owning_service |
| `DeploymentNode` | `deployment_nodes` | name, platform |
| `InfraComponent` | `infra_components` | name, cloud |
| `ADR` | `adrs` | title, status, context, decision |
| `StandardReference` | `standard_references` | source, citation, license |
| `Artifact` | `artifacts` | artifact_type, name, version, content, traceability (JSON) |
| `AuditLog` | `audit_logs` | actor, action, payload (JSON) |

### Artifact Generator Functions

| Function | Artifact Type | Output Format |
|----------|--------------|--------------|
| `_domain_doc()` | `domain_model` | Markdown narrative |
| `_c4_container_doc()` | `c4` | Markdown + Mermaid flowchart |
| `_openapi_doc()` | `openapi` | YAML (OpenAPI 3.1.0) |
| `_asyncapi_doc()` | `asyncapi` | YAML (AsyncAPI 2.6.0) |
| `_terraform_doc()` | `terraform` | HCL (Terraform) |
| `_adr_doc()` | `adr` | Markdown (RFC-style) |
| `_impact_baseline_doc()` | `impact_baseline` | Markdown inventory |
| `_arc42_doc()` | `arc42` | Markdown (arc42 template) |

---

## Frontend Components

```
src/
├── App.tsx                  BrowserRouter + providers + AppLayout
├── lib/
│   ├── api.ts               All fetch calls (typed, tenant/role headers)
│   ├── apiKeyContext.tsx     ApiKeyProvider, useApiKey(), getStoredApiKey()
│   └── settingsContext.tsx  SettingsProvider, useSettings()
├── components/
│   ├── layout/
│   │   └── Sidebar.tsx      NavLink list, logo, settings link
│   ├── ApiKeyBanner.tsx     Top banner: key input + save + dismiss
│   ├── ArtifactViewer.tsx   Tab list + code pre + copy button
│   └── ExamplesDrawer.tsx   Overlay drawer: 4 template cards + Use button
└── pages/
    ├── Landing.tsx           Hero, lifecycle, capabilities, metrics, CTA
    ├── Dashboard.tsx         Stats grid, Recharts bar chart, package list
    ├── Studio.tsx            6-step wizard (main page)
    │   ├── Step1             Name + description + AI Suggest button
    │   ├── Step2             Dynamic requirement rows (FR/NFR)
    │   ├── Step3             Domains + services
    │   ├── Step4             APIs + events
    │   ├── Step5             Data entities + infra components
    │   └── Step6             Review summary + Create button
    ├── Packages.tsx          Filter tabs + package card list
    ├── PackageDetail.tsx     Package header + ArtifactViewer
    ├── Impact.tsx            Changeset form + result cards
    ├── Governance.tsx        Approval form + pending/approved queues
    └── SettingsPage.tsx      Identity + API key management
```

### State Management

| Context | Storage | Contents |
|---------|---------|---------|
| `ApiKeyContext` | localStorage (`archsmith_anthropic_key`) | API key string |
| `SettingsContext` | localStorage (`archsmith_tenant/role/actor`) | Tenant ID, role, actor |
| Studio local state | Component state | All 9 form sections across 6 steps |

### Security Headers

Every API call in `api.ts` sends:
```
X-Tenant-Id: {tenantId}   — scopes all queries
X-Role: {role}            — enforced by require_reviewer_or_admin()
```

AI suggest sends API key in request body (`api_key` field) — never in headers.
