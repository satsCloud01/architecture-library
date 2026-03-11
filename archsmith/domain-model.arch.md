---
name: "Domain Model"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "domain-model"
icon: "🏗️"
tags: [SQLAlchemy, Pydantic]
---

# Domain Model

## Bounded Contexts

```
┌─────────────────────────────────────────────────────────────────────┐
│                     ArchSmith Domain                                │
│                                                                     │
│  ┌─────────────────────────┐   ┌────────────────────────────────┐  │
│  │  Package Context        │   │  Artifact Context              │  │
│  │                         │   │                                │  │
│  │  ArchitecturePackage    │──►│  Artifact (8 types)            │  │
│  │  Requirement (FR/NFR)   │   │  - domain_model                │  │
│  │  Domain                 │   │  - c4                          │  │
│  │  BoundedContext         │   │  - openapi                     │  │
│  │  Service                │   │  - asyncapi                    │  │
│  │  API                    │   │  - terraform                   │  │
│  │  Event                  │   │  - adr                         │  │
│  │  DataEntity             │   │  - impact_baseline             │  │
│  │  DeploymentNode         │   │  - arc42                       │  │
│  │  InfraComponent         │   │                                │  │
│  │  StandardReference      │   └────────────────────────────────┘  │
│  └─────────────────────────┘                                        │
│                                                                     │
│  ┌──────────────────────┐   ┌────────────────────────────────────┐  │
│  │  Governance Context  │   │  AI Suggestion Context             │  │
│  │                      │   │                                    │  │
│  │  ADR                 │   │  (stateless — request/response     │  │
│  │  AuditLog            │   │   only; not persisted)             │  │
│  │  Approval            │   │                                    │  │
│  │  Role (RBAC)         │   │  Input:  description + api_key     │  │
│  │  Tenant              │   │  Output: full architecture payload │  │
│  └──────────────────────┘   └────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

## Entity Relationships

```
ArchitecturePackage (1)
  ├── Requirement (*)        category: FR | NFR
  ├── Domain (*)
  │     └── BoundedContext (*)
  │           └── Service (*)
  │                 ├── API (*)
  │                 └── Event (*)
  ├── DataEntity (*)
  ├── DeploymentNode (*)
  ├── InfraComponent (*)
  ├── StandardReference (*)
  ├── ADR (*)               status: proposed | accepted
  ├── Artifact (*)          8 types per generation run
  └── AuditLog (*)          every action recorded
```

## Package State Machine

```
          [create]
              │
              ▼
          ┌───────┐
          │ draft │
          └───────┘
              │
          [generate]
              │
              ▼
         ┌──────────┐
         │ generated│
         └──────────┘
              │
          [approve]      (requires REVIEWER or ADMIN role)
              │
              ▼
         ┌──────────┐
         │ approved │  ◄── version++, ADR-APPROVAL written, audit logged
         └──────────┘
```

## Artifact Traceability Schema

Each `Artifact` record stores a `traceability` JSON field:
```json
{
  "requirements": ["FR-1", "FR-2", "NFR-1"],
  "adrs": ["ADR-001 Adopt Service-Oriented Bounded Contexts", "ADR-002 Event-Driven Integration Pattern"],
  "citations": ["ArchSmith Architecture Standards v1", "PRD FR1-FR6"]
}
```

## Version Management

- Every `generate` call increments `package.version`
- Every `approve` call also increments `package.version`
- Artifacts are versioned with the package version at generation time
- Old artifact versions are retained (full history available via `GET /artifacts`)

## Audit Log Actions

| Action | Trigger |
|--------|---------|
| `architecture.create` | POST /create |
| `architecture.generate` | POST /{id}/generate |
| `architecture.impact` | POST /{id}/impact |
| `architecture.approve` | POST /{id}/approve |
