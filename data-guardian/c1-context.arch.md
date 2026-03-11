---
name: "C1 – System Context"
project: "DataGuardian"
project_slug: "data-guardian"
project_url: "https://data-guardian.satszone.link"
github: "https://github.com/satsCloud01/data-guardian"
category: "data-analytics"
type: "c4-context"
icon: "🛡️"
tags: [FastAPI, React, Claude API, React Flow]
---

# C1 — System Context Diagram

DataGuardian sits at the centre of the enterprise data governance ecosystem, serving five distinct user personas and integrating with two external systems.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Enterprise Environment                          │
│                                                                         │
│   [Data Steward]  [Domain Owner]  [Compliance Ofc]  [CRO/CISO]        │
│        │               │                │                │              │
│        └───────────────┴────────────────┴────────────────┘              │
│                                  │                                      │
│                                  ▼                                      │
│                    ┌─────────────────────────┐                         │
│                    │      DataGuardian        │                         │
│                    │  Data Governance Platform│                         │
│                    │  (React SPA + FastAPI)   │                         │
│                    └────────────┬────────────┘                         │
│                                 │                                       │
│              ┌──────────────────┼──────────────────┐                   │
│              ▼                  ▼                   ▼                   │
│   [Anthropic Claude API]  [Core Banking System]  [Reg. Bodies]         │
│   AI-powered governance   Source of truth data   BCBS, GDPR, etc.      │
│                                                                         │
│   [Data Engineer]   ────────────────────────────────────────────────►  │
│   (read-only consumer)                                                  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## User Personas

| Persona | Role | Primary Use Cases |
|---------|------|-------------------|
| **Data Steward** | Owns data quality and documentation for a domain | Document fields, raise quality issues, manage glossary terms, approve changes |
| **Domain Owner** | Business owner of a data domain | Review data models, approve change requests, view compliance status |
| **Compliance Officer** | Ensures regulatory adherence | Map models to standards (BCBS-239, GDPR, SOX), track control status, run gap analysis |
| **CRO / CISO** | Executive risk and security oversight | Dashboard KPIs, sensitivity tier review, PII exposure summary |
| **Data Engineer** | Technical consumer of schema definitions | Browse model registry, explore lineage, read field documentation |

---

## External Systems

| System | Direction | Purpose |
|--------|-----------|---------|
| **Anthropic Claude API** (`claude-haiku-4-5-20251001`) | Outbound | AI-powered glossary recommendations, field descriptions, naming validation, documentation scoring, schema summarisation, change impact narratives |
| **Core Banking System** | Reference | Models in DataGuardian represent entities from production systems (Account, Customer, Loan, etc.) — DataGuardian stores metadata, not the data itself |
| **Regulatory Bodies** | Reference | Governance standards (BCBS-239, GDPR, DAMA-DMBOK, ISO-11179, SOX-302, PCI-DSS, DCAM-V3, ISO-8000) are pre-loaded as reference data |

---

## Key Boundaries

- DataGuardian is a **metadata registry** — it stores schema definitions and governance metadata, never production data
- All AI calls use the **user-supplied Anthropic API key** (stored in browser `localStorage`, passed via `X-API-Key` header) — gracefully degrades to mock responses if no key is set
- The platform is **multi-domain** — models span Core Banking, Lending, Risk, Compliance, Trading, Analytics, Payments, HR, and Operations domains
