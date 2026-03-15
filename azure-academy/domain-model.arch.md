---
name: "Domain Model"
project: "Azure Academy"
project_slug: "azure-academy"
category: "learning"
type: "domain-model"
icon: "🔷"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'azure']
---

# Domain Model

## Entities

### 1. Lesson (42 instances)
Primary learning content unit. Each lesson belongs to exactly one category.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier (1-42) |
| category | string | One of 14 categories |
| title | string | Display title |
| type | string | `intro`, `concept` |
| description | string | Main content paragraph |
| keyPoints | string[] | Bullet-point takeaways |
| comparison | object? | Table with headers + rows |
| diagram | object? | Flow/architecture diagram (nodes + edges) |
| code | string? | Code/config example |
| codeLanguage | string? | Language hint for display |

### 2. Category (14 instances)
Logical grouping of lessons. Derived at runtime, not stored separately.

| Category | Coverage |
|----------|----------|
| Cloud Foundations | Azure global model, shared responsibility, subscription structure |
| Global Infrastructure | Regions, availability zones, paired regions, edge |
| Identity and Access | Entra ID, RBAC, Managed Identity, Conditional Access |
| Governance and Landing Zones | Management groups, Azure Policy, CAF, landing zones |
| Azure Networking | VNets, NSGs, Azure Firewall, ExpressRoute, Front Door |
| Compute and Containers | VMs, App Service, AKS, Container Apps, Functions |
| Storage | Blob, Files, Disks, Data Lake, redundancy options |
| Data, Analytics, and AI | SQL, Cosmos DB, Synapse, Azure AI, Cognitive Services |
| Integration and Messaging | Service Bus, Event Grid, Logic Apps, API Management |
| Security | Key Vault, Defender for Cloud, Sentinel, DDoS Protection |
| Reliability and DR | Availability sets/zones, ASR, backup, geo-redundancy |
| Operations and Observability | Azure Monitor, Log Analytics, Application Insights |
| DevOps and IaC | Azure DevOps, GitHub Actions, Bicep, ARM templates |
| Advanced Architectures | Multi-region, hybrid with Arc, microservices patterns |

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

### 4. Paper (16 instances)
Curated Azure documentation references and guides.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | Document/guide title |
| authors | string | Author names |
| year | int | Publication year |
| category | string | Architecture, Security, Operations, etc. |
| url | string | Link to original document |
| summary | string | One-paragraph summary |
| key_takeaways | string[] | 5-6 most important points |
| related_topics | string[] | Connected concepts |

## Entity Relationships

```
Category ──1:N──▶ Lesson
Category ──1:N──▶ QuizQuestion
Category ──1:N──▶ Paper (via paper.category)
```

All entities are **read-only** at runtime. No CRUD operations. No user-generated content.

## Data Storage

All data is defined as Python lists/dicts in source code:
- `backend/src/azureacademy/data/lessons.py` — LESSONS, QUIZ_QUESTIONS
- `backend/src/azureacademy/routers/papers.py` — PAPERS (inline)

No database. No migrations. No ORM.

