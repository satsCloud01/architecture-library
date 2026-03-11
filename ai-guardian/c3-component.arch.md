---
name: "C3 – Component Diagram"
project: "Enterprise AI Guardian"
project_slug: "ai-guardian"
project_url: "https://ai-guardian.satszone.link"
github: "https://github.com/satsCloud01/ai-governance-bank"
category: "ai-agents"
type: "c4-component"
icon: "🏦"
tags: [FastAPI, React, Claude API]
---

# C3 — Component Diagrams

> **C4 Level 3**: Zooms into individual containers showing internal components,
> their responsibilities, and interactions.

---

## 3.1 Governance API — Internal Components

```mermaid
C4Component
  title Component Diagram — Governance API (FastAPI)

  Container_Boundary(api, "Governance API") {

    Component(main, "Application Bootstrap", "governance/main.py", "Initialises FastAPI app, registers CORS middleware, mounts all routers, and triggers DB init + seed on startup via lifespan context manager.")

    Component(db, "Database Layer", "governance/database.py", "Creates async SQLAlchemy engine, session factory, and Base declarative class. Exposes get_db() dependency for injection into all route handlers.")

    Component(models_pkg, "ORM Models Package", "governance/models/core.py", "Defines 13 SQLAlchemy Mapped Column model classes representing all governance entities. All relationships declared with back_populates for bidirectional navigation.")

    Component(seed, "Demo Data Seeder", "governance/seed.py", "Idempotent seeder: checks if data exists before inserting. Populates 13 AI models, versions, risk assessments, lifecycle stages, compliance mappings with controls, performance metrics, fairness metrics, incidents, and audit logs.")

    Component(r_dashboard, "Dashboard Router", "routers/dashboard.py", "Aggregates cross-cutting summary metrics (counts, compliance score, alert counts, pending approvals) and risk heatmap data for the executive dashboard.")

    Component(r_models, "Models Router", "routers/models.py", "Full CRUD for AI model registry. Supports filtering by type/stage/tier. Eager-loads relationships for detail view. Returns audit trail per model.")

    Component(r_risk, "Risk Router", "routers/risk.py", "Returns risk assessments sorted by score. Provides risk matrix data (impact × likelihood 1-5 scale) for scatter plot visualisation.")

    Component(r_lifecycle, "Lifecycle Router", "routers/lifecycle.py", "Returns kanban board structure: all 6 lifecycle stages with their assigned model cards including blockers and approval status.")

    Component(r_compliance, "Compliance Router", "routers/compliance.py", "Regulation list, per-model compliance overview (controls passed/total by regulation), gap analysis (Gap/In Progress mappings), and per-model control checklists.")

    Component(r_monitoring, "Monitoring Router", "routers/monitoring.py", "Time-series performance metrics per model, ECOA fairness metrics per model, and aggregated performance + fairness alerts across all production models.")

    Component(r_incidents, "Incidents Router", "routers/incidents.py", "Returns all incidents joined with model metadata, sorted by severity then date.")
  }

  Container_Ext(db_container, "Governance Database", "SQLite / PostgreSQL")

  Rel(main, db, "Calls init_db() and seed() on startup")
  Rel(main, r_dashboard, "Mounts router at /api/dashboard")
  Rel(main, r_models, "Mounts router at /api/models")
  Rel(main, r_risk, "Mounts router at /api/risk")
  Rel(main, r_lifecycle, "Mounts router at /api/lifecycle")
  Rel(main, r_compliance, "Mounts router at /api/compliance")
  Rel(main, r_monitoring, "Mounts router at /api/monitoring")
  Rel(main, r_incidents, "Mounts router at /api/incidents")

  Rel(r_dashboard, db, "AsyncSession queries via get_db()", "SQLAlchemy")
  Rel(r_models, db, "AsyncSession queries + selectinload eager loading", "SQLAlchemy")
  Rel(r_risk, db, "AsyncSession queries", "SQLAlchemy")
  Rel(r_lifecycle, db, "AsyncSession queries", "SQLAlchemy")
  Rel(r_compliance, db, "AsyncSession queries + selectinload", "SQLAlchemy")
  Rel(r_monitoring, db, "AsyncSession queries", "SQLAlchemy")
  Rel(r_incidents, db, "AsyncSession queries with join", "SQLAlchemy")

  Rel(db, db_container, "Async reads/writes", "aiosqlite / asyncpg")
  Rel(seed, db_container, "Idempotent seed insert", "SQLAlchemy Async")
  Rel(models_pkg, db_container, "DDL via create_all()", "SQLAlchemy Core")
```

---

## 3.2 React SPA — Internal Components

```mermaid
C4Component
  title Component Diagram — React SPA

  Container_Boundary(spa, "Single-Page Application") {

    Component(router, "App Router", "App.tsx + React Router", "Defines all routes. Wraps platform pages in AppLayout (sidebar + main content area). Landing page has no layout wrapper.")

    Component(layout, "AppLayout / Sidebar", "components/layout/Sidebar.tsx", "Fixed left navigation with active route highlighting. Links to all 8 platform pages. Back-link to landing page.")

    Component(api_client, "Typed API Client", "lib/api.ts", "Centralised fetch wrapper with full TypeScript interface definitions for all 20+ API response shapes. All components import from here — no inline fetch calls.")

    Component(landing, "Landing Page", "pages/Landing.tsx", "Marketing/pitch page with: hero, problem statement, 8-step governance lifecycle flowchart, LLM vs ML dual panel, 8 capability cards, regulatory coverage grid, compliance flow, governance maturity journey, 13-model preview, CTA.")

    Component(dashboard, "Dashboard", "pages/Dashboard.tsx", "Executive view: 5 stat cards, model type breakdown, lifecycle pipeline bar chart, governance health radar chart, risk heatmap sortable table.")

    Component(registry, "Model Registry", "pages/ModelRegistry.tsx", "Searchable, multi-filter model table with inline LLM-specific fields (injection risk, hallucination %, token cost). Click-through to model detail.")

    Component(detail, "Model Detail", "pages/ModelDetail.tsx", "Full model profile: info panel, risk scoring bars, lifecycle + approvals, LLM governance panel (6 fields), data lineage table, version timeline, audit log.")

    Component(risk_page, "Risk Assessment", "pages/RiskAssessment.tsx", "Tier summary cards, interactive scatter plot (impact × regulatory exposure), full assessment table with progress bars.")

    Component(lifecycle_page, "Lifecycle Kanban", "pages/Lifecycle.tsx", "6-column kanban board (Dev → Retired). Each card shows blocker badge, risk tier, owner. Stage progress indicator bar.")

    Component(compliance_page, "Compliance", "pages/Compliance.tsx", "3 tabs: Overview (expandable model rows with per-regulation compliance %), Gap Analysis table, Regulation cards. Overall compliance score bar.")

    Component(monitoring_page, "Monitoring", "pages/Monitoring.tsx", "Model selector, 3 tabs: Performance (AUC/PSI or Hallucination/Cost charts), Fairness (ECOA disparate impact table), Alerts (performance + fairness breach list).")

    Component(incidents_page, "Incidents", "pages/Incidents.tsx", "Severity-sorted incident cards with color-coded left border, status badges, resolution notes, filter tabs.")

    Component(badge, "Badge Component", "components/ui/Badge.tsx", "Auto-resolves variant from children string (e.g., 'Tier 1', 'Production', 'Critical') for consistent color coding across all pages.")

    Component(statcard, "StatCard Component", "components/ui/StatCard.tsx", "Reusable metric card with label, large value, optional sub-text and icon. Color variants: default, red, green, yellow, brand.")
  }

  Container_Ext(api_container, "Governance API", "FastAPI :8000")

  Rel(router, layout, "Wraps platform pages in AppLayout")
  Rel(layout, landing, "No layout wrap")
  Rel(router, dashboard, "Route: /dashboard")
  Rel(router, registry, "Route: /models")
  Rel(router, detail, "Route: /models/:id")
  Rel(router, risk_page, "Route: /risk")
  Rel(router, lifecycle_page, "Route: /lifecycle")
  Rel(router, compliance_page, "Route: /compliance")
  Rel(router, monitoring_page, "Route: /monitoring")
  Rel(router, incidents_page, "Route: /incidents")

  Rel(dashboard, api_client, "api.dashboard.summary(), api.dashboard.heatmap()")
  Rel(registry, api_client, "api.models.list(filters)")
  Rel(detail, api_client, "api.models.get(id), api.models.audit(id)")
  Rel(risk_page, api_client, "api.risk.assessments(), api.risk.matrix()")
  Rel(lifecycle_page, api_client, "api.lifecycle.kanban()")
  Rel(compliance_page, api_client, "api.compliance.overview/gapAnalysis/regulations()")
  Rel(monitoring_page, api_client, "api.monitoring.metrics/fairness/alerts()")
  Rel(incidents_page, api_client, "api.incidents.list()")

  Rel(api_client, api_container, "fetch() calls", "HTTP/JSON over Vite proxy")
```

---

## 3.3 Data Flow: Model Registration to Approval

```mermaid
sequenceDiagram
  participant Dev as Model Developer
  participant SPA as React SPA
  participant API as Governance API
  participant DB as Database
  participant MRM as Model Risk Manager
  participant CRO

  Dev->>SPA: Fill model registration form
  SPA->>API: POST /api/models
  API->>DB: INSERT AIModel + DataLineage
  API->>DB: INSERT LifecycleStage (stage=Development)
  API->>DB: INSERT AuditLog (model_registered)
  API-->>SPA: 201 Created { id, name }

  Dev->>SPA: Request risk classification
  SPA->>API: POST /api/risk/classify
  API->>DB: INSERT RiskAssessment (tier, scores)
  API->>DB: INSERT ComplianceMapping per regulation
  API->>DB: INSERT ComplianceControls (status=Pending)
  API->>DB: INSERT AuditLog (risk_assessment)
  API-->>SPA: RiskAssessment { tier, score }

  Dev->>SPA: Submit for Validation
  SPA->>API: POST /api/lifecycle/{id}/transition
  API->>DB: UPDATE LifecycleStage (stage=Validation)
  API->>DB: INSERT AuditLog (stage_transition)
  API-->>SPA: 200 OK

  MRM->>SPA: Review validation findings
  MRM->>SPA: Approve → MRM Review stage
  SPA->>API: POST /api/lifecycle/{id}/transition
  API->>DB: UPDATE LifecycleStage (stage=MRM Review)
  API->>DB: INSERT ApprovalRecord (role=MRM, decision=Approved)
  API->>DB: INSERT AuditLog (approval)

  CRO->>SPA: Final sign-off
  SPA->>API: POST /api/lifecycle/{id}/transition
  API->>DB: UPDATE LifecycleStage (stage=Production)
  API->>DB: INSERT ApprovalRecord (role=CRO, decision=Approved)
  API->>DB: INSERT AuditLog (approval)
  API-->>SPA: 200 OK { stage: Production }
```

---

## 3.4 Monitoring Alert Flow

```mermaid
flowchart TD
  A[Scheduled Metric Ingest] --> B{Model Type?}
  B -->|LLM| C[Collect: Hallucination Rate\nResponse Relevance\nLatency p95\nToken Cost]
  B -->|ML / Statistical| D[Collect: AUC-ROC\nPSI\nGini\nAccuracy / F1]

  C --> E{Breach Threshold?}
  D --> E

  E -->|Yes| F[Insert PerformanceMetric\nalert_triggered = True]
  E -->|No| G[Insert PerformanceMetric\nalert_triggered = False]

  F --> H[Monitoring Alerts API\nGET /api/monitoring/alerts]
  H --> I[Dashboard Alert Banner]
  H --> J[Incidents Router\nAuto-create Incident]

  K[Fairness Monitor] --> L[Compute DI Ratio\nper Protected Class]
  L --> M{DI Ratio < 0.8?}
  M -->|Yes - Breach| N[Insert FairnessMetric\nflag = Breach]
  M -->|≥0.75 - Warning| O[Insert FairnessMetric\nflag = Warning]
  M -->|≥0.8 - OK| P[Insert FairnessMetric\nflag = OK]

  N --> Q[Fairness Alert Surface\nGET /api/monitoring/alerts]
  Q --> R[Compliance Gap Created\nfor ECOA]
```
