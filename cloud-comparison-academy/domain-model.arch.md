---
name: "Domain Model"
project: "Cloud Comparison Academy"
project_slug: "cloud-comparison-academy"
category: "learning"
type: "domain-model"
icon: "⚖️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'multi-cloud']
---

# Domain Model

## Entities

### 1. Lesson (28 instances)
Primary learning content unit. Each lesson belongs to exactly one category.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier (1-28) |
| category | string | One of 6 categories |
| title | string | Display title |
| type | string | `intro`, `concept`, `reference` |
| description | string | Main content paragraph |
| keyPoints | string[] | Bullet-point takeaways |
| comparison | object? | Table with headers + rows (typically AWS vs Azure vs GCP) |
| diagram | object? | Flow/architecture diagram (nodes + edges) |
| code | string? | Code/config example |
| codeLanguage | string? | Language hint for display |

### 2. Category (6 instances)
Logical grouping of lessons. Derived at runtime, not stored separately.

| Category | Coverage |
|----------|----------|
| Executive Overview | Cloud market, provider positioning, decision criteria |
| Service Mapping | Compute, storage, networking, data, AI service equivalents |
| Architecture Comparison | Design patterns, landing zones, governance models |
| Cost & FinOps | Pricing models, discount programs, FinOps tooling |
| Workload Selection | Best-fit analysis by workload type |
| Decision Framework | Selection methodology, migration paths, hybrid strategies |

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

### 4. Paper (12 instances)
Curated industry papers and cloud comparison guides.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | Paper/guide title |
| authors | string | Author names |
| year | int | Publication year |
| category | string | Analysis, Architecture, FinOps, Strategy |
| url | string | Link to original document |
| summary | string | One-paragraph summary |
| key_takeaways | string[] | 5-6 most important points |
| related_topics | string[] | Connected concepts |

### 5. Calculator Model (frontend-only)
Weighted cloud comparison tool with interactive sliders.

| Field | Type | Description |
|-------|------|-------------|
| criteria | string[8] | Decision dimensions (Service Breadth, Governance, etc.) |
| weights | number[8] | User-adjustable weights (0-10 scale) |
| ratings | object | Baseline provider ratings (0-5 per criterion per provider) |
| presets | object[4] | Pre-configured weight profiles (Balanced, Microsoft Enterprise, Data/AI First, Ops Simplicity) |

## Entity Relationships

```
Category ──1:N──▶ Lesson
Category ──1:N──▶ QuizQuestion
Category ──1:N──▶ Paper (via paper.category)
```

All entities are **read-only** at runtime. No CRUD operations. No user-generated content.

## Data Storage

All data is defined as Python lists/dicts in source code:
- `backend/src/cloudcomparison/data/lessons.py` — LESSONS, QUIZ_QUESTIONS
- `backend/src/cloudcomparison/routers/papers.py` — PAPERS (inline)
- Calculator data is defined in `frontend/src/pages/Calculator.jsx` (frontend-only)

No database. No migrations. No ORM.

