---
name: "C2 – Container Diagram"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "c4-container"
icon: "🏗️"
tags: [FastAPI, React, SQLAlchemy, SQLite]
---

# C2 — Container Diagram

## Overview

ArchSmith consists of two runtime containers: a React SPA served by Nginx, and a FastAPI backend. In production both run on a single EC2 t2.micro instance separated by Nginx routing.

```
┌────────────────────────────────────────────────────────────────────────┐
│                        ArchSmith (EC2 t2.micro)                        │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                      Nginx (port 443/80)                         │  │
│  │   /          → serves /var/www/archsmith/frontend/dist/          │  │
│  │   /api/      → proxy_pass http://127.0.0.1:8002                  │  │
│  │   /health    → proxy_pass http://127.0.0.1:8002                  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│          │ static files               │ API proxy                      │
│          ▼                            ▼                                │
│  ┌──────────────────┐      ┌─────────────────────────────────────────┐ │
│  │   React SPA      │      │   FastAPI (uvicorn :8002)               │ │
│  │   (dist/)        │      │                                         │ │
│  │                  │      │   POST /api/architecture/create         │ │
│  │  8 pages         │      │   POST /api/architecture/{id}/generate  │ │
│  │  React Router v6 │      │   POST /api/architecture/{id}/impact    │ │
│  │  Tailwind CSS    │      │   POST /api/architecture/{id}/approve   │ │
│  │  Recharts        │      │   POST /api/ai/suggest                  │ │
│  │  lucide-react    │      │   GET  /api/examples                    │ │
│  │                  │      │                                         │ │
│  │  localStorage:   │      │   PYTHONPATH=src                        │ │
│  │  - API key       │      │   module: archsmith.main                │ │
│  │  - tenant/role   │      └─────────────────────────────────────────┘ │
│  └──────────────────┘                │                                 │
│                                      ▼                                 │
│                          ┌────────────────────────┐                   │
│                          │   SQLite               │                   │
│                          │   archsmith.db         │                   │
│                          │                        │                   │
│                          │  architecture_packages │                   │
│                          │  requirements          │                   │
│                          │  domains               │                   │
│                          │  bounded_contexts      │                   │
│                          │  services              │                   │
│                          │  apis                  │                   │
│                          │  events                │                   │
│                          │  data_entities         │                   │
│                          │  deployment_nodes      │                   │
│                          │  infra_components      │                   │
│                          │  adrs                  │                   │
│                          │  artifacts             │                   │
│                          │  standard_references   │                   │
│                          │  audit_logs            │                   │
│                          └────────────────────────┘                   │
└────────────────────────────────────────────────────────────────────────┘
            │ HTTPS (443)
            │
     [Browser / User]
            │
            ▼
   ┌──────────────────┐
   │  Anthropic API   │
   │  Claude Haiku    │
   │  (per-request    │
   │   API key from   │
   │   localStorage)  │
   └──────────────────┘
```

## Container Responsibilities

### React SPA
- 8 pages with React Router v6
- Studio wizard: 6-step architecture creation with AI Suggest and Load Example
- ArtifactViewer: tabbed code viewer with copy button
- ApiKeyBanner: prompts for Anthropic key, stores in localStorage
- ExamplesDrawer: slide-in panel with 4 pre-built templates
- SettingsContext: tenant, role, actor persisted in localStorage

### FastAPI Backend
- Handles all business logic, artifact generation, impact analysis, governance
- RBAC via `X-Role` header (`ARCHITECT` / `REVIEWER` / `ADMIN`)
- Tenant isolation via `X-Tenant-Id` header
- AI endpoint: receives `{description, api_key}`, calls Claude Haiku, returns structured JSON
- Examples endpoint: serves 4 inline template payloads

### SQLite
- Single file: `backend/archsmith.db`
- Auto-created via `Base.metadata.create_all()` on first startup
- 14 tables, no migrations required

## Data Flow: Studio → Generate

```
Browser                 FastAPI              SQLite            Anthropic
   │                      │                    │                   │
   │ POST /api/ai/suggest  │                    │                   │
   │ {description,key} ──►│                    │                   │
   │                      │ messages.create ──►│                   │
   │                      │◄── JSON suggestion │                   │
   │◄── AISuggestResponse │                    │                   │
   │                      │                    │                   │
   │ POST /architecture/  │                    │                   │
   │   create {payload} ─►│                    │                   │
   │                      │ INSERT package ───►│                   │
   │◄── {id, name, v1}    │                    │                   │
   │                      │                    │                   │
   │ POST /{id}/generate ►│                    │                   │
   │                      │ generate_artifacts()                   │
   │                      │ INSERT artifacts ─►│                   │
   │◄── ArtifactsResponse │                    │                   │
```
