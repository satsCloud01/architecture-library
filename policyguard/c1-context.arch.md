---
name: "C1 – System Context"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "c4-context"
icon: "🔐"
tags: [FastAPI, React, ABAC]
---

# C1 — System Context

> Level 1 of the C4 model: who uses the system and what external systems does it interact with?

---

## Context Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ENTERPRISE DATA PLATFORM                            │
│                                                                             │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐                              │
│   │  Human   │   │  System  │   │ AI Agent │   ← Requestors               │
│   │ Analyst  │   │  (ETL)   │   │ (Claude) │                              │
│   └────┬─────┘   └────┬─────┘   └────┬─────┘                              │
│        │              │              │                                      │
│        └──────────────┴──────────────┘                                      │
│                           │  Access Request                                 │
│                           ▼                                                 │
│          ┌────────────────────────────────┐                                 │
│          │   Dynamic Policy Access Engine  │  ← This system                │
│          │                                 │                                │
│          │  • ABAC Runtime Evaluation      │                                │
│          │  • Policy Administration        │                                │
│          │  • Audit & Compliance           │                                │
│          │  • AI-Assisted Authoring        │                                │
│          └───────────────┬────────────────┘                                 │
│                          │                                                  │
│          ┌───────────────┼────────────────┐                                 │
│          ▼               ▼                ▼                                 │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐                          │
│   │ Snowflake  │  │ PostgreSQL │  │ Databricks │  ← Protected Data         │
│   │   PROD DW  │  │ Core OLTP  │  │ ML Platform│    Sources               │
│   └────────────┘  └────────────┘  └────────────┘                          │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐                          │
│   │  S3 Data   │  │   MySQL    │  │  Anthropic │  ← External Services     │
│   │    Lake    │  │    CRM     │  │   Claude   │                          │
│   └────────────┘  └────────────┘  └────────────┘                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## External Actors

### Requestors (who sends access requests)

| Actor | Type | Description | Auth Method |
|---|---|---|---|
| Data Analyst | Human | Queries Snowflake reporting schemas for BI | Username + role attributes |
| Data Scientist | Human | Reads PII data for model training (EU region) | Username + role + MFA |
| Compliance Officer | Human | Full audit read across all platforms | Username + role + MFA |
| DBA / Platform Team | Human | Infrastructure maintenance access | Username + role (maintenance windows only) |
| Customer Service | Human | Account lookup with masked card data | Username + role + purpose |
| ETL Pipeline | System | Bulk extract from OLTP to Data Lake | Service identifier + principal_type=system |
| BI Dashboard | System | Reporting schema reads for dashboards | Service identifier + principal_type=system |
| Risk Scoring Engine | System | Real-time credit data enrichment | Service identifier |
| ML Training Platform | System | Automated feature store access | Service identifier |
| Claude Analytics Agent | AI Agent | Curated data reads for insight generation | Agent identifier + principal_type=agent |
| Fraud Detection Agent | AI Agent | Feature store reads for real-time scoring | Agent identifier |
| Compliance Monitor Agent | AI Agent | Automated regulatory monitoring | Agent identifier |

### External Services

| Service | Role |
|---|---|
| Anthropic Claude Haiku | AI policy suggestion, conflict detection, impact analysis |
| Snowflake PROD DW | Primary analytics warehouse (PII, Transactions, Reporting, Risk) |
| PostgreSQL Core Banking | Operational OLTP (Accounts, Payments, KYC) |
| Databricks ML Platform | Feature store, model training, experiment tracking |
| S3 Data Lake | Raw / Processed / Curated zones |
| MySQL CRM | Customer relationships, leads, support tickets |

---

## Key Interactions

### 1. Access Evaluation (runtime PDP)
```
Requestor → POST /api/access/evaluate
  → ABAC Engine evaluates all active policies
  → Returns PERMIT/DENY + obligations (masked columns, row filter)
  → Writes immutable audit log entry
```

### 2. Policy Administration (PAP)
```
Compliance Officer → CRUD /api/policies
  → Create/update/activate/deprecate policies
  → Each change snapshots a PolicyVersion
  → AI Assist: POST /api/ai/suggest → Claude Haiku
```

### 3. Dry-Run Simulation
```
Any Actor → POST /api/simulate
  → ABAC Engine evaluates (no audit log written)
  → Returns full policy match breakdown for transparency
```

### 4. Audit & Compliance
```
Compliance Officer → GET /api/audit
  → Filterable immutable log (decision, requestor, datasource, action)
  → Stats: permit rate, deny rate, avg latency
```

---

## Security Boundary

The Dynamic Policy Access Engine sits **between requestors and data platforms**. It is the Policy Decision Point (PDP) and Policy Administration Point (PAP). The actual data retrieval happens at the data platform layer — the engine returns a decision and obligations; enforcement is the responsibility of the consuming application (Policy Enforcement Point, PEP).
