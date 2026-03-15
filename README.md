# Architecture Library

> **A centralised, version-controlled collection of architecture documentation for every application in the Sats portfolio.**

Each sub-folder maps 1-to-1 to a deployed product and contains structured `.arch.md` documents that follow the [C4 model](https://c4model.com/), domain-driven design, and Architecture Decision Record (ADR) conventions.

---

## What Is This Repository?

This repository serves as the **single source of truth** for architecture artefacts across 19 applications. Rather than scattering architecture docs inside each project repo, every C4 diagram, domain model, API specification, ADR, runbook, and constraints document lives here in a consistent, discoverable format.

All documents use the `.arch.md` extension with YAML front-matter (name, project, category, tags) so they can be indexed, searched, and rendered programmatically.

---

## Repository Structure

```
architecture-library/
├── agent-control-center/   # Agent Control Center (LangGraph orchestration)
├── agentfier/              # Agentfier (Streamlit analysis wizard)
├── agentkraft/             # AgentKraft (agent builder platform)
├── ai-guardian/            # Enterprise AI Guardian (model governance)
├── archsmith/              # ArchSmith (architecture studio)
├── cloud-deployer/         # Universal Cloud Deployer
├── data-guardian/          # DataGuardian (data model governance)
├── dataanalyzer-buddy/     # DataAnalyzerBuddy (AI data analysis)
├── datanexus/              # DataNexus (data hub)
├── duck-analytics/         # Duck Analytics
├── iceberg-academy/        # Iceberg Academy
├── messenger/              # Lightweight Messenger (WebSocket chat)
├── mylegal-doctor/         # MyLegal Doctor (legal AI assistant)
├── policyguard/            # PolicyGuard (ABAC policy engine)
├── sats-cortex/            # SatsKnowledge Cortex
├── sats-learning-academy/  # SatsLearning Academy
├── sats-recruitment/       # SatsRecruitment Intelligence
├── semantic-modeler/       # Semantic Modeler (RDF/ontology)
├── sentinel-ai/            # Sentinel AI (data observability)
└── README.md
```

---

## Document Types

| Type | File Pattern | Purpose |
|------|-------------|---------|
| **C4 Context (L1)** | `c1-context.arch.md` / `c4-context.arch.md` | System-in-environment view: actors, external systems, boundaries |
| **C4 Container (L2)** | `c2-container.arch.md` / `c4-container.arch.md` | Runtime containers: frontend, backend, database, queues |
| **C4 Component (L3)** | `c3-component.arch.md` / `c3-backend.arch.md` / `c3-frontend.arch.md` / `c4-component.arch.md` | Internal module breakdown per container |
| **C4 Code (L4)** | `c4-code.arch.md` | Class/function-level detail for critical modules |
| **Domain Model** | `domain-model.arch.md` | Entity relationships, aggregates, bounded contexts |
| **API Specification** | `api-spec.arch.md` | REST endpoints, request/response schemas, status codes |
| **Constraints** | `constraints.arch.md` | Technical and business constraints, quality attributes |
| **ADR** | `adr-*.arch.md` | Architecture Decision Records (problem, options, outcome) |
| **Runbook** | `runbook.arch.md` | Operational procedures, incident response, health checks |

Every document includes Mermaid diagrams that render natively on GitHub.

---

## Coverage Matrix

| Project | C4 L1 | C4 L2 | C4 L3 | C4 L4 | Domain | API | Constraints | ADRs | Runbook | Total |
|---------|:-----:|:-----:|:-----:|:-----:|:------:|:---:|:-----------:|:----:|:-------:|:-----:|
| agent-control-center | x | | | | x | x | x | | | 4 |
| agentfier | x | | | | x | x | x | | | 4 |
| agentkraft | x | | | | | | | | | 1 |
| ai-guardian | x | x | x | | x | x | | 3 | x | 9 |
| archsmith | x | x | | | x | x | x | | x | 6 |
| cloud-deployer | x | | | | | | | | | 1 |
| data-guardian | x | | | | x | x | x | | | 4 |
| dataanalyzer-buddy | x | | | | x | | | | | 2 |
| datanexus | x | | | | | | | | | 1 |
| duck-analytics | x | | | | x | x | x | | | 4 |
| iceberg-academy | x | x | x | x | x | x | x | | | 8 |
| messenger | x | | | | | | | | | 1 |
| mylegal-doctor | x | | | | | | | | | 1 |
| policyguard | x | x | x | | x | x | | 3 | x | 9 |
| sats-cortex | x | x | x | | x | | | | | 6 |
| sats-learning-academy | | x | x | x | x | x | x | | | 7 |
| sats-recruitment | x | | | | x | x | x | | | 4 |
| semantic-modeler | x | | | | x | | | | | 2 |
| sentinel-ai | x | | | | x | x | x | | | 4 |
| networking-academy       | architecture, domain-model, api-spec, constraints |   4 |
| **Totals** | **18** | **6** | **5** | **2** | **14** | **11** | **8** | **6** | **2** | **78** |

---

## Front-Matter Schema

Every `.arch.md` file starts with YAML front-matter:

```yaml
---
name: "C1 – System Context"
project: "Enterprise AI Guardian"
project_slug: "ai-guardian"
project_url: "https://ai-guardian.satszone.link"
github: "https://github.com/satsCloud01/ai-governance-bank"
category: "ai-agents"
type: "c4-context"
icon: "\U0001F3E6"
tags: [FastAPI, React, Recharts]
---
```

Key fields:

| Field | Description |
|-------|-------------|
| `project_slug` | Matches the folder name in this repo |
| `project_url` | Live production URL |
| `category` | Grouping (e.g., `ai-agents`, `data-platform`, `devops`) |
| `type` | Document type for filtering (`c4-context`, `domain-model`, `api-spec`, `adr`, etc.) |
| `tags` | Technology tags for cross-cutting search |

---

## How to Use

**Browse on GitHub** -- Navigate into any project folder and open a document. Mermaid diagrams render automatically.

**Search across projects** -- Use GitHub's search or clone locally and grep:

```bash
# Find all documents tagged with "FastAPI"
grep -rl "FastAPI" --include="*.arch.md" .

# Find all ADRs
find . -name "adr-*.arch.md"

# Find all domain models
find . -name "domain-model.arch.md"
```

**Reference from other repos** -- Link directly to a document:

```
https://github.com/satsCloud01/architecture-library/blob/main/ai-guardian/c1-context.arch.md
```

---

## Adding Documentation

1. Create a folder matching the project slug (lowercase, hyphenated).
2. Add `.arch.md` files with the front-matter schema above.
3. Include Mermaid diagrams for visual artefacts (C4, ER, sequence).
4. Commit with a message following the pattern: `Add <Project> architecture docs (<doc types>)`.

---

## Tech Stack Across the Portfolio

The 19 documented applications share a common technology foundation:

- **Backend**: FastAPI (Python 3.12), Streamlit, SQLite, SQLAlchemy
- **Frontend**: React 18, Vite, Tailwind CSS, Recharts, React Flow
- **AI/ML**: Anthropic Claude (Haiku/Sonnet), LangGraph, LangChain, ChromaDB
- **Infrastructure**: Docker, AWS (S3, CloudFront, EC2), Nginx reverse proxy
- **Diagramming**: Mermaid (C4, sequence, ER, flowchart)

---

## Related Links

| Resource | URL |
|----------|-----|
| Live Portfolio | [my-solution-registry.satszone.link](https://my-solution-registry.satszone.link) |
| This Repository | [github.com/satsCloud01/architecture-library](https://github.com/satsCloud01/architecture-library) |

---

## License

Internal use only. All architecture documents are proprietary to the Sats portfolio.

