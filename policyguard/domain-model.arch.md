---
name: "Domain Model"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "domain-model"
icon: "🔐"
tags: [SQLAlchemy, Pydantic]
---

# Domain Model

> Core entities, relationships, and business rules of the Dynamic Policy Access Engine.

---

## Entity Relationship Diagram

```
┌──────────────────┐         ┌──────────────────────┐
│    Principal     │         │       Policy          │
│──────────────────│         │──────────────────────│
│ id               │         │ id                    │
│ identifier       │         │ name                  │
│ name             │         │ effect (permit|deny)  │
│ type             │◄─────── │ state                 │
│ attributes_json  │  audit  │ version               │
│ is_active        │         │ category              │
└──────────────────┘         │ subject_json          │
                             │ resource_json         │         ┌────────────────┐
                             │ action_json           │         │ PolicyVersion  │
                             │ conditions_json       │────────►│────────────────│
                             └──────────┬────────────┘  1:N   │ id             │
                                        │                      │ policy_id      │
                                        │ 1:N                  │ version        │
                                        ▼                      │ snapshot_json  │
                             ┌──────────────────────┐         │ created_at     │
                             │      AuditLog         │         └────────────────┘
                             │──────────────────────│
                             │ id                    │
                             │ request_id (UUID)     │
                             │ decision              │
                             │ principal_id          │
                             │ requestor_type        │
                             │ datasource            │
                             │ action                │
                             │ matched_policy_id     │
                             │ obligations_json      │
                             │ latency_ms            │
                             │ created_at            │
                             └──────────────────────┘

┌──────────────────┐         ┌──────────────────────┐
│   DataSource     │         │      Settings         │
│──────────────────│         │──────────────────────│
│ id               │         │ key                   │
│ name             │         │ value                 │
│ db_type          │         └──────────────────────┘
│ connection_json  │
│ schema_cache_json│
│ live_connection  │
│ last_synced_at   │
└──────────────────┘
```

---

## Entities

### Policy

The central entity. A Policy encodes a single access control rule.

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | int | PK | Auto-increment |
| `name` | string | UNIQUE, NOT NULL | Human-readable rule name |
| `effect` | enum | `permit` \| `deny` | Decision direction if this policy matches |
| `state` | enum | See state machine | Lifecycle state |
| `version` | int | ≥ 1 | Increments on every field change |
| `category` | string | nullable | Grouping tag (PII, Payment, ML, Global, etc.) |
| `description` | string | nullable | Plain-English purpose statement |
| `subject_json` | JSON | nullable | Who the policy applies to |
| `resource_json` | JSON | nullable | What data resource the policy covers |
| `action_json` | JSON array | nullable | Which operations (SELECT, DELETE, etc.) |
| `conditions_json` | JSON | nullable | Contextual conditions (time, IP, purpose) |
| `priority` | int | default 0 | Evaluation order for `first-applicable` algorithm |
| `created_at` | datetime | NOT NULL | Creation timestamp |
| `updated_at` | datetime | NOT NULL | Last modification timestamp |

**State Machine:**

```
draft ──► review ──► active ──► deprecated ──► archived
  ▲          │
  └──────────┘  (can return to draft from review)
```

| State | PDP Evaluated | Editable | Description |
|---|---|---|---|
| `draft` | No | Yes | Being authored; not yet in effect |
| `review` | No | Limited | Awaiting compliance sign-off |
| `active` | **Yes** | Yes (creates new version) | Fully enforced |
| `deprecated` | No | No | Superseded; retained for audit history |
| `archived` | No | No | Soft-deleted; hidden from UI by default |

---

### PolicyVersion

Append-only snapshot of a Policy at a point in time.

| Field | Type | Description |
|---|---|---|
| `id` | int | PK |
| `policy_id` | int | FK → Policy |
| `version` | int | Version number at time of snapshot |
| `snapshot_json` | JSON | Full policy fields at this version |
| `created_at` | datetime | When this version was created |

> Versions are created automatically on every `PUT /api/policies/{id}` that changes a field value.

---

### Principal

An identity that sends access requests to the PDP.

| Field | Type | Description |
|---|---|---|
| `id` | int | PK |
| `identifier` | string | UNIQUE — `alice@corp.com` or `etl-pipeline-001` |
| `name` | string | Display name |
| `type` | enum | `human` \| `system` \| `agent` |
| `attributes_json` | JSON | ABAC attributes: role, region, clearance_level, department, etc. |
| `is_active` | bool | Soft toggle; inactive principals still appear in audit |
| `created_at` | datetime | |

**Principal Types:**

| Type | Examples | Typical Attributes |
|---|---|---|
| `human` | Data Analyst, Data Scientist, Compliance Officer | role, region, clearance_level, department |
| `system` | ETL Pipeline, BI Dashboard, Risk Scoring Engine | role, service_type, environment |
| `agent` | Claude Analytics Agent, Fraud Detection Agent | role, agent_id, purpose, principal_type=agent |

> `principal_type` in `attributes_json` is what policies match on; `type` in the `principals` table is for display/filtering only.

---

### DataSource

A protected data platform registered with the engine.

| Field | Type | Description |
|---|---|---|
| `id` | int | PK |
| `name` | string | UNIQUE, human-readable (`Snowflake PROD DW`) |
| `identifier` | string | UNIQUE, machine key (`snowflake_prod`) |
| `db_type` | enum | `snowflake`, `postgresql`, `databricks`, `s3`, `mysql` |
| `connection_json` | JSON | Connection parameters (host, port, database, user, password) |
| `schema_cache_json` | JSON | Cached table/column metadata |
| `live_connection` | bool | False = mock metadata; True = real driver |
| `last_synced_at` | datetime | When schema was last refreshed |

> `connection_json` fields named `password`, `key`, `token`, or `secret` are **masked** (`***`) in all API responses.

---

### AuditLog

Immutable record of every access decision.

| Field | Type | Description |
|---|---|---|
| `id` | int | PK, auto-increment |
| `request_id` | UUID | Unique per evaluate call (idempotency key) |
| `decision` | enum | `PERMIT` \| `DENY` |
| `effect` | string | Effect of the matching policy (or `implicit` for no-match denies) |
| `principal_id` | int | FK → Principal (nullable for anonymous requests) |
| `requestor_type` | string | `human` \| `system` \| `agent` |
| `datasource` | string | Target datasource identifier |
| `action` | string | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| `matched_policy_id` | int | FK → Policy (nullable if no match) |
| `obligations_json` | JSON | Obligations returned with PERMIT |
| `subject_snapshot` | JSON | Subject attributes at time of request |
| `latency_ms` | int | Engine evaluation latency |
| `created_at` | datetime | Immutable timestamp |

> There are **no UPDATE or DELETE operations** on `audit_logs`. Rows are written once and never modified.

---

### Settings

Key-value configuration store.

| Key | Default | Description |
|---|---|---|
| `combining_algorithm` | `deny-overrides` | How multiple matching policies are resolved |
| `anthropic_api_key` | `""` | Claude API key for AI-assist features |
| `audit_retention_days` | `90` | Rolling audit log retention window (display only; no auto-purge in Phase 1) |

---

## Business Rules

### BR-1: Combining Algorithm
The engine evaluates all active policies for every request. The combining algorithm (from `settings`) determines the final decision:
- `deny-overrides`: Any matching DENY → final DENY; all permits, no deny → PERMIT; no match → DENY
- `permit-overrides`: Any matching PERMIT → final PERMIT; otherwise DENY
- `first-applicable`: First match (by priority) wins; no match → DENY

### BR-2: Implicit DENY
Any request with no matching active policy evaluates to DENY. There is no default-permit.

### BR-3: Obligations
Obligations are returned **only** with PERMIT decisions. A DENY response never carries obligations.

### BR-4: Policy Versioning
Every modification to a `Policy` that changes any field value creates a new `PolicyVersion` and increments `version`. The current row is always the latest version.

### BR-5: Audit Immutability
`AuditLog` rows are INSERT-only. No backend code path performs UPDATE or DELETE on this table.

### BR-6: Dry-Run Isolation
`POST /api/access/simulate` evaluates policies identically to `/evaluate` but never writes an `AuditLog` row.

### BR-7: Inactive Principals in Audit
Setting a `Principal.is_active = false` does not delete their historical audit records. Past decisions remain queryable.

### BR-8: DataSource Connection Masking
Any `connection_json` field whose key contains `password`, `key`, `token`, or `secret` (case-insensitive) is replaced with `***` in all API responses.
