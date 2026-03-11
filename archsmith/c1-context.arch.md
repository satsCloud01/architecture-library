---
name: "C1 – System Context"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "c4-context"
icon: "🏗️"
tags: [FastAPI, React, Claude API]
---

# C1 — System Context Diagram

## Overview

ArchSmith sits between enterprise architects and the architectural artifacts they need. It receives structured inputs (or an AI-assisted description), calls Claude Haiku for intelligent suggestions, persists everything in SQLite, and returns production-ready artifacts.

```
┌──────────────────────────────────────────────────────────────────────┐
│                         ArchSmith System                             │
│                  [Architecture Intelligence Platform]                │
└──────────────────────────────────────────────────────────────────────┘
         ▲                    ▲                    ▲
         │                    │                    │
┌────────┴──────┐   ┌─────────┴───────┐   ┌───────┴────────┐
│  Enterprise   │   │  Principal /    │   │  Security &    │
│  Architect    │   │  Staff Engineer │   │  Risk Teams    │
│               │   │                 │   │                │
│ Creates arch  │   │ Reviews impact  │   │ Audits         │
│ packages via  │   │ analysis and    │   │ governance     │
│ Studio wizard │   │ ADR outputs     │   │ approvals      │
└───────────────┘   └─────────────────┘   └────────────────┘

         ▼                    ▼
┌────────┴──────┐   ┌─────────┴───────┐
│  Anthropic    │   │  Browser        │
│  Claude API   │   │  localStorage   │
│               │   │                 │
│ claude-haiku  │   │ API key, tenant │
│ AI suggest    │   │ role, actor     │
└───────────────┘   └─────────────────┘
```

## Users

| User | Role | Interaction |
|------|------|-------------|
| Enterprise Architect | `ARCHITECT` | Creates packages, generates artifacts, runs impact analysis |
| Principal / Staff Engineer | `ARCHITECT` | Reviews and refines generated artifacts |
| Security & Risk Teams | `REVIEWER` | Reviews and approves packages via governance workflow |
| CTO / CIO Office | `ADMIN` | Full access; can approve any package |

## External Systems

| System | Purpose |
|--------|---------|
| Anthropic Claude API | AI architecture suggestion — `claude-haiku-4-5-20251001` |
| Browser localStorage | Stores API key, tenant ID, role, actor (never server-side) |
| AWS EC2 (t2.micro) | Hosts FastAPI backend + Nginx serving React dist |
| Let's Encrypt | Provides SSL certificate for `archsmith.satszone.link` |

## Key Boundaries

- ArchSmith does **not** call external standards databases or knowledge stores at runtime — standards are embedded in the artifact generators.
- The Anthropic API key is always provided **by the user via browser** — it is passed per-request and never stored server-side.
- All data persists in **SQLite** on the EC2 instance — suitable for demo/enterprise pilot scale.
