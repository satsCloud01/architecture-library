---
name: "C3 – Component Diagram"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "c4-component"
icon: "🔐"
tags: [FastAPI, React, Claude API]
---

# C3 — Component Diagram

> Level 3 of the C4 model: the internal components of the FastAPI application container.

---

## Component Diagram

```
┌──────────────────────────────────────────────────────────────────┐
│                     FastAPI Application (Port 8003)              │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │   /access   │  │  /policies  │  │  /principals            │  │
│  │   Router    │  │   Router    │  │  Router                 │  │
│  │             │  │             │  │                         │  │
│  │ evaluate    │  │ list        │  │ list / get              │  │
│  │ simulate    │  │ create      │  │ create / toggle         │  │
│  │ batch       │  │ get         │  │ recent_access           │  │
│  └──────┬──────┘  │ update      │  └────────────┬────────────┘  │
│         │         │ delete      │               │               │
│         │         │ versions    │               │               │
│         ▼         └──────┬──────┘               │               │
│  ┌─────────────────────┐ │                      │               │
│  │   ABAC Engine       │ │  ┌─────────────────┐ │               │
│  │   (engine.py)       │ │  │  /datasources   │ │               │
│  │                     │ │  │  Router         │ │               │
│  │  evaluate()         │ │  │                 │ │               │
│  │  simulate()         │ │  │  list / get     │ │               │
│  │  _match_subject()   │ │  │  create         │ │               │
│  │  _match_resource()  │ │  │  sync_schema    │ │               │
│  │  _match_action()    │ │  │  toggle_live    │ │               │
│  │  _match_conditions()│ │  └────────┬────────┘ │               │
│  │  _obligations()     │ │           │           │               │
│  │  _apply_combining() │ │           │           │               │
│  └──────────┬──────────┘ │           │           │               │
│             │            │           │           │               │
│  ┌──────────▼────────────▼───────────▼───────────▼────────────┐  │
│  │                 SQLAlchemy Async Session                    │  │
│  │                                                             │  │
│  │  PolicyRepository  PrincipalRepository  AuditRepository    │  │
│  │  DatasourceRepository  SettingsRepository                  │  │
│  └─────────────────────────────┬───────────────────────────────┘  │
│                                │                                  │
│  ┌──────────┐  ┌────────────┐  │  ┌──────────────────────────┐   │
│  │ /audit   │  │ /dashboard │  │  │   /ai Router             │   │
│  │ Router   │  │ Router     │  │  │                          │   │
│  │          │  │            │  │  │  suggest_policy()        │   │
│  │ list     │  │ summary    │  │  │  check_conflicts()       │   │
│  │ stats    │  │ activity   │  │  │  impact_analysis()       │   │
│  │          │  │ top_denied │  │  │                          │   │
│  │          │  │ breakdown  │  │  │  [Anthropic Claude SDK]  │   │
│  └────┬─────┘  └─────┬──────┘  │  └──────────────────────────┘   │
│       │              │         │                                  │
│       └──────────────┘         │                                  │
│                                ▼                                  │
│                    ┌────────────────────┐                        │
│                    │   SQLite DB        │                        │
│                    │   policyguard.db   │                        │
│                    └────────────────────┘                        │
└──────────────────────────────────────────────────────────────────┘
```

---

## Components

### ABAC Engine (`policyguard/engine.py`)

The heart of the system. Stateless evaluation function — receives a request dict and a list of `Policy` ORM objects.

**`evaluate(request, policies, algorithm) → EvaluationResult`**

```
Input:
  request = {
    subject:  { role, region, clearance_level, principal_type, ... }
    resource: { datasource, tables, columns, regions, ... }
    action:   SELECT | INSERT | UPDATE | DELETE | *
    context:  { time, ip_address, purpose, ... }
  }
  policies  = list[Policy]  (only active policies from DB)
  algorithm = deny-overrides | permit-overrides | first-applicable

Output:
  EvaluationResult = {
    decision:         PERMIT | DENY
    matched_policy:   Policy | None
    obligations:      { masked_columns, row_filter, max_rows } | {}
    evaluated_count:  int
    matching_permits: list[Policy]
    matching_denies:  list[Policy]
  }
```

**Matching Logic:**

| Phase | Function | Description |
|---|---|---|
| Subject | `_match_subject()` | Evaluates role (list ∋ value), region (list ∋ value), clearance_level (gte/lte/eq), principal_type (exact) |
| Resource | `_match_resource()` | Evaluates datasource (list/wildcard), tables (list/wildcard), columns.allow (whitelist), regions (list) |
| Action | `_match_action()` | Wildcard `*` matches all; otherwise exact match from list |
| Conditions | `_match_conditions()` | time_window (hour range), ip_cidr, purpose, max_rows |

**Combining Algorithms:**

| Algorithm | Behavior |
|---|---|
| `deny-overrides` | Any DENY match → final DENY; all must permit → PERMIT; no match → DENY |
| `permit-overrides` | Any PERMIT match → final PERMIT; no match or all deny → DENY |
| `first-applicable` | First matching policy (permit or deny) wins; no match → DENY |

**Obligations (`_obligations()`):**
- `masked_columns` ← `resource_json.columns.masked` (list of column names to mask)
- `row_filter` ← `resource_json.row_filter` (SQL WHERE clause string)
- `max_rows` ← `conditions_json.max_rows_per_query` (integer cap)

---

### Access Router (`routers/access.py`)

| Endpoint | Method | Description |
|---|---|---|
| `/api/access/evaluate` | POST | Real PDP: evaluate request, write audit log (unless `dry_run=true`) |
| `/api/access/simulate` | POST | Dry-run: evaluate request, skip audit log, return full policy breakdown |
| `/api/access/batch` | POST | Batch evaluate up to 100 requests in a single call |

**Evaluate flow:**
```
POST /api/access/evaluate
  1. Load active policies from DB
  2. Load combining algorithm from settings
  3. engine.evaluate(request, policies, algorithm)
  4. Write AuditLog row (if not dry_run)
  5. Return: { decision, matched_policy_id, obligations, evaluated_count, latency_ms }
```

**Simulate flow:**
```
POST /api/access/simulate
  1-3. Same as evaluate
  4. SKIP audit log
  5. Return: { decision, policy_results: [{ policy_id, name, matched, effect }] }
```

---

### Policies Router (`routers/policies.py`)

Full CRUD with state machine and versioning.

**State Machine:**
```
draft → review → active → deprecated → archived
         ↑________↓
         (can move back to draft)
```

- Every `PUT` that changes policy fields appends a `PolicyVersion` snapshot
- Soft delete sets state to `archived` (no physical delete)

---

### Principals Router (`routers/principals.py`)

Manages identities used in access requests. Each principal has `attributes_json` evaluated by ABAC policies at runtime.

| Field | Values |
|---|---|
| `type` | `human` \| `system` \| `agent` |
| `attributes_json` | `{ role, region, clearance_level, department, ... }` |
| `is_active` | Boolean; inactive principals can still appear in audit history |

---

### Data Sources Router (`routers/datasources.py`)

| Mode | Behavior |
|---|---|
| Mock (default) | Returns cached `schema_cache_json` from DB; no real DB connection |
| Live | Attempts real driver connection (psycopg2/snowflake-connector/etc.) |

`connection_json` fields named `password`, `key`, `token`, `secret` are **masked** (`***`) in all API responses.

---

### Audit Router (`routers/audit.py`)

Immutable append-only log. No UPDATE or DELETE operations exist.

**Filters:** `decision`, `requestor_type`, `action`, `datasource`, `days` (rolling window), `limit`, `offset`

**Stats endpoint** (`GET /api/audit/stats`):
```json
{
  "total_requests": 847,
  "permit_count": 612,
  "deny_count": 235,
  "permit_rate": 72.3,
  "avg_latency_ms": 4.2,
  "by_datasource": { "snowflake": 412, ... },
  "by_action": { "SELECT": 780, ... }
}
```

---

### Dashboard Router (`routers/dashboard.py`)

Aggregation-only endpoint — no new data written.

| Endpoint | Description |
|---|---|
| `GET /api/dashboard/summary` | Counts, rates, `by_effect`, `by_state` breakdowns |
| `GET /api/dashboard/activity` | Daily request counts for last 30 days |
| `GET /api/dashboard/top-denied` | Top 5 deny reasons (policy name + count) |
| `GET /api/dashboard/datasource-breakdown` | Per-datasource permit/deny counts |

---

### AI Router (`routers/ai.py`)

| Endpoint | Description |
|---|---|
| `POST /api/ai/suggest` | Send plain-English rule → Claude Haiku → returns policy JSON suggestion |
| `POST /api/ai/conflicts` | Send new policy → Claude Haiku → returns conflict analysis vs existing policies |
| `POST /api/ai/impact` | Send policy change → Claude Haiku → returns impact narrative |

Requires `anthropic_api_key` in `settings` table. Returns `{ suggestion: null, message: "API key not configured" }` if key absent.

---

### Settings Router (`routers/settings.py`)

Key-value store backed by `settings` table.

| Key | Values | Default |
|---|---|---|
| `combining_algorithm` | `deny-overrides` \| `permit-overrides` \| `first-applicable` | `deny-overrides` |
| `anthropic_api_key` | string | `""` |
| `audit_retention_days` | integer string | `"90"` |

---

## Data Flow: Live Access Decision

```
1. Client (Human/System/Agent) sends:
   POST /api/access/evaluate
   { "subject": {...}, "resource": {...}, "action": "SELECT", "context": {...} }

2. Access Router:
   a. Queries DB: SELECT * FROM policies WHERE state = 'active'
   b. Queries DB: SELECT value FROM settings WHERE key = 'combining_algorithm'

3. ABAC Engine:
   a. For each active policy: _match_subject + _match_resource + _match_action + _match_conditions
   b. _apply_combining(matching_permits, matching_denies, algorithm) → decision
   c. _obligations(first_matching_permit) → { masked_columns, row_filter, max_rows }

4. Audit Logger:
   INSERT INTO audit_logs (decision, policy_id, subject_snapshot, obligations, latency_ms)

5. Response:
   { "decision": "PERMIT", "matched_policy_id": 7, "obligations": {...}, "latency_ms": 3 }
```
