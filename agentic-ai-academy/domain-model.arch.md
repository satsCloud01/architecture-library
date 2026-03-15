---
name: "Domain Model"
project: "Agentic AI Academy"
project_slug: "agentic-ai-academy"
category: "learning"
type: "domain-model"
icon: "🤖"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'agentic-ai']
---

# Domain Model

## Entities

### 1. Lesson (36 instances)
Primary learning content unit. Each lesson belongs to exactly one category.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier (1-36) |
| category | string | One of 12 categories |
| title | string | Display title |
| type | string | `intro`, `concept`, `summary` |
| description | string | Main content paragraph |
| keyPoints | string[] | Bullet-point takeaways |
| comparison | object? | Table with headers + rows |
| diagram | object? | Flow/architecture diagram (nodes + edges) |
| code | string? | Code/config example |
| codeLanguage | string? | Language hint for display |

### 2. Category (12 instances)
Logical grouping of lessons. Derived at runtime, not stored separately.

| Category | Lessons | Coverage |
|----------|---------|----------|
| Agentic Foundations | 3 | Agent control loop, capabilities, protocol layering |
| Protocol Landscape | 2 | Protocol classes, selection principles |
| MCP Deep Dive | 4 | Architecture, primitives, strengths/risks, transport |
| A2A Deep Dive | 3 | Core concepts, MCP vs A2A, agent cards |
| Tool/API Protocols | 4 | Tradeoff matrix, OpenAPI, JSON-RPC, gRPC |
| Security and Trust | 3 | Trust boundaries, mandatory controls, failure modes |
| Runtime Patterns | 3 | Pattern matrix, action envelope, sync vs async |
| Observability and Evaluation | 3 | Telemetry model, SLOs, evaluation dimensions |
| Governance and Compliance | 3 | Controls, compliance mapping, data handling |
| Reference Architectures | 3 | Progression model, single agent, multi-agent |
| Anti-Patterns | 3 | Common anti-patterns, hardening checklist, failure modes |
| Protocol Selector | 2 | Selection guide, baseline blueprint |

### 3. QuizQuestion (15 instances)
Multiple-choice assessment with server-side scoring.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| category | string | Maps to lesson category |
| question | string | Question text |
| options | string[4] | Four answer choices |
| correct | int | Index of correct answer (server-only) |
| explanation | string | Why the answer is correct (server-only) |

### 4. Paper (14 instances)
Curated specifications and standards for agentic AI.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | Spec/RFC title |
| authors | string | Author names |
| year | int | Publication year |
| category | string | Context Protocols, Security, RPC, etc. |
| url | string | Link to original document |
| summary | string | One-paragraph summary |
| key_takeaways | string[] | 5-6 most important points |
| related_topics | string[] | Connected concepts/specs |

## Entity Relationships

```
Category ──1:N──▶ Lesson
Category ──1:N──▶ QuizQuestion
Category ──1:N──▶ Paper (via paper.category)
```

All entities are **read-only** at runtime. No CRUD operations. No user-generated content.

## Data Storage

All data is defined as Python lists/dicts in source code:
- `backend/src/agenticacademy/data/lessons.py` — LESSONS, QUIZ_QUESTIONS
- `backend/src/agenticacademy/routers/papers.py` — PAPERS (inline)

No database. No migrations. No ORM.

