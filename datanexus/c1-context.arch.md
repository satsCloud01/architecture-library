---
name: "C1 – System Context"
project: "DataNexus"
project_slug: "datanexus"
project_url: "https://satshub.satszone.link"
github: "https://github.com/satsCloud01/satshub"
category: "data-analytics"
type: "c4-context"
icon: "🗂️"
tags: [Streamlit, FastAPI, Python, Pydantic]
---

# C1 — System Context Diagram

DataNexus is a data discovery and governance platform inspired by LinkedIn's DataHub. It provides a unified catalog for datasets, dashboards, charts, and data pipelines with URN-based entity addressing, full-text search, business glossary, and ownership tracking.

```
┌──────────────────────────────────────────────────────────────────┐
│                       User Environment                           │
│                                                                  │
│   [Data Engineer]   [Data Steward]   [Analytics Lead]            │
│        │                  │                │                     │
│        └──────────────────┴────────────────┘                     │
│                           │                                      │
│                      HTTPS (Browser)                             │
│                           │                                      │
│                           ▼                                      │
│            ┌─────────────────────────┐                           │
│            │       DataNexus          │                           │
│            │   Data Discovery &       │                           │
│            │   Governance Platform    │                           │
│            │   (Streamlit + FastAPI)  │                           │
│            └────────────┬────────────┘                           │
│                         │                                        │
│              ┌──────────┼──────────┐                             │
│              ▼          ▼          ▼                              │
│       [In-Memory DB]  [Search   [Seed Data]                     │
│       (no external    Engine]   (pre-loaded                     │
│        database)                 catalog)                        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Actors

| Actor | Role |
|---|---|
| **Data Engineer** | Browse datasets, trace data pipelines, view schema fields |
| **Data Steward** | Manage glossary terms, assign ownership, tag entities |
| **Analytics Lead** | Discover dashboards and charts, explore domain hierarchy |

---

## Key Boundaries

- DataNexus uses an **in-memory database** — all entity data lives in RAM per process, no external DB required
- Entities are addressed by **URN-based IDs** (e.g. `urn:li:dataset:customers`)
- The platform supports **6 entity types**: Dataset, Dashboard, Chart, DataFlow, DataJob, SchemaField
- **Governance layer**: GlossaryTerm, GlossaryNode, Tag, Domain, User, Group
- Full-text search via a custom SearchEngine with configurable filters
- Pre-seeded with demo catalog data on startup via `seed_data.py`

---

## Container Overview

| Container | Technology | Port | Purpose |
|-----------|-----------|------|---------|
| **Streamlit UI** | Streamlit >= 1.40 | 8501 | Interactive data catalog browser with search, entity pages |
| **FastAPI Backend** | FastAPI + Pydantic | 8000 | REST API for CRUD operations on all entity types |
| **In-Memory Store** | Python dict | — | Singleton `InMemoryDatabase` holding all entities |
