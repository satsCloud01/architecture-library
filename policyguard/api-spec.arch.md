---
name: "API Specification"
project: "PolicyGuard"
project_slug: "policyguard"
project_url: "https://policyguard.satszone.link"
github: "https://github.com/satsCloud01/policyguard"
category: "architecture-devops"
type: "api-spec"
icon: "🔐"
tags: [FastAPI, OpenAPI]
---

# API Reference

> **Live interactive docs:** `http://localhost:8003/docs` (Swagger UI) or `http://localhost:8003/redoc`

The Dynamic Policy Access Engine exposes a REST JSON API. All endpoints are prefixed with `/api`.

---

## Base URL

| Environment | URL |
|---|---|
| Local development | `http://localhost:8003` |
| Production | `https://policyguard.satszone.link` |

---

## Authentication

Currently **no authentication** is required (single-tenant local deployment). All endpoints are publicly accessible from the frontend SPA.

> Phase 2: JWT bearer token authentication with role-based API access planned as a future enhancement.

---

## Access Evaluation

### `POST /api/access/evaluate`

Submit an access request to the live Policy Decision Point (PDP). Writes an audit log entry (unless `dry_run=true`).

**Request Body:**
```json
{
  "subject": {
    "role": "data_scientist",
    "region": "EU",
    "clearance_level": 3,
    "principal_type": "human",
    "department": "analytics"
  },
  "resource": {
    "datasource": "snowflake_prod",
    "tables": ["customers"],
    "action": "SELECT"
  },
  "action": "SELECT",
  "context": {
    "time": "2025-03-01T14:30:00Z",
    "ip_address": "10.0.1.42",
    "purpose": "model_training"
  },
  "dry_run": false
}
```

**Response `200`:**
```json
{
  "decision": "PERMIT",
  "matched_policy_id": 7,
  "matched_policy_name": "PII Read - EU Data Scientists",
  "obligations": {
    "masked_columns": ["ssn", "dob"],
    "row_filter": "data_region = 'EU'",
    "max_rows": 10000
  },
  "evaluated_count": 23,
  "latency_ms": 3
}
```

| Field | Type | Description |
|---|---|---|
| `decision` | `"PERMIT"` \| `"DENY"` | Final access decision |
| `matched_policy_id` | `int \| null` | ID of the winning policy |
| `obligations` | `object` | Constraints on the PERMIT (masking, row filter, max rows) |
| `evaluated_count` | `int` | Number of active policies evaluated |
| `latency_ms` | `int` | Engine evaluation time in milliseconds |

---

### `POST /api/access/simulate`

Dry-run evaluation. Identical to `evaluate` but never writes to the audit log. Returns full per-policy match breakdown.

**Request Body:** Same as `/evaluate` (ignores `dry_run` field — always dry)

**Response `200`:**
```json
{
  "decision": "PERMIT",
  "policy_results": [
    {
      "policy_id": 7,
      "name": "PII Read - EU Data Scientists",
      "effect": "permit",
      "matched": true
    },
    {
      "policy_id": 12,
      "name": "Block DELETE - All Humans",
      "effect": "deny",
      "matched": false
    }
  ],
  "evaluated_count": 23
}
```

---

### `POST /api/access/batch`

Evaluate up to **100 requests** in a single API call.

**Request Body:**
```json
{
  "requests": [
    { "subject": {...}, "resource": {...}, "action": "SELECT" },
    { "subject": {...}, "resource": {...}, "action": "DELETE" }
  ]
}
```

**Response `200`:**
```json
{
  "results": [
    { "index": 0, "decision": "PERMIT", "matched_policy_id": 7, "obligations": {...} },
    { "index": 1, "decision": "DENY",   "matched_policy_id": 12, "obligations": {} }
  ]
}
```

---

## Policies

### `GET /api/policies`

List all policies with optional filters.

**Query Parameters:**

| Param | Type | Description |
|---|---|---|
| `state` | string | Filter by state: `draft`, `review`, `active`, `deprecated`, `archived` |
| `effect` | string | Filter by effect: `permit` \| `deny` |
| `category` | string | Filter by category tag |
| `search` | string | Substring search on policy name |
| `limit` | int | Max results (default 50) |
| `offset` | int | Pagination offset (default 0) |

**Response `200`:**
```json
{
  "policies": [
    {
      "id": 7,
      "name": "PII Read - EU Data Scientists",
      "effect": "permit",
      "state": "active",
      "version": 3,
      "category": "PII",
      "created_at": "2025-01-15T10:00:00Z",
      "updated_at": "2025-02-01T14:30:00Z"
    }
  ],
  "total": 23
}
```

---

### `POST /api/policies`

Create a new policy. Initial state is always `draft`.

**Request Body:**
```json
{
  "name": "Block AI Agents - Payment Writes",
  "effect": "deny",
  "category": "Payment",
  "description": "Prevents all AI agents from writing to payment tables",
  "subject_json": { "principal_type": "agent" },
  "resource_json": { "datasource": ["postgres_core"], "tables": ["payments", "transactions"] },
  "action_json": ["INSERT", "UPDATE", "DELETE"],
  "conditions_json": {}
}
```

**Response `201`:** Full policy object with `id`, `state: "draft"`, `version: 1`

---

### `GET /api/policies/{id}`

Get a single policy with its full version history.

**Response `200`:**
```json
{
  "id": 7,
  "name": "PII Read - EU Data Scientists",
  "effect": "permit",
  "state": "active",
  "version": 3,
  "subject_json": {...},
  "resource_json": {...},
  "action_json": ["SELECT"],
  "conditions_json": {...},
  "versions": [
    { "version": 1, "snapshot_json": {...}, "created_at": "2025-01-15T10:00:00Z" },
    { "version": 2, "snapshot_json": {...}, "created_at": "2025-01-20T09:15:00Z" },
    { "version": 3, "snapshot_json": {...}, "created_at": "2025-02-01T14:30:00Z" }
  ]
}
```

---

### `PUT /api/policies/{id}`

Update a policy. Automatically increments `version` and appends a `PolicyVersion` snapshot.

**Response `200`:** Updated policy object

---

### `DELETE /api/policies/{id}`

Soft-delete: sets state to `archived`. The policy is removed from PDP evaluation.

**Response `204`:** No content

---

## Principals

### `GET /api/principals`

**Query Params:** `type` (`human|system|agent`), `search`, `is_active`, `limit`, `offset`

**Response `200`:**
```json
{
  "principals": [
    {
      "id": 1,
      "identifier": "alice@corp.com",
      "name": "Alice Chen",
      "type": "human",
      "attributes_json": { "role": "data_scientist", "region": "EU", "clearance_level": 3 },
      "is_active": true
    }
  ],
  "total": 15
}
```

### `POST /api/principals` — Create principal `201`
### `GET /api/principals/{id}` — Get with `recent_access` and `stats`
### `PATCH /api/principals/{id}/toggle` — Flip `is_active`

---

## Data Sources

### `GET /api/datasources` — List all (no `schema_cache`)
### `GET /api/datasources/{id}` — Full detail including `schema_cache_json` (passwords masked)
### `POST /api/datasources` — Create new datasource `201`
### `POST /api/datasources/{id}/sync` — Trigger schema metadata sync
### `PATCH /api/datasources/{id}/live-connection` — Toggle live vs mock mode

---

## Audit Trail

### `GET /api/audit`

**Query Params:** `decision` (`PERMIT|DENY`), `requestor_type`, `action`, `datasource`, `days` (rolling window), `limit`, `offset`

**Response `200`:**
```json
{
  "logs": [
    {
      "id": 1234,
      "request_id": "req_abc123",
      "decision": "PERMIT",
      "principal_id": 1,
      "requestor_type": "human",
      "datasource": "snowflake_prod",
      "action": "SELECT",
      "matched_policy_id": 7,
      "obligations_json": { "masked_columns": ["ssn"] },
      "latency_ms": 3,
      "created_at": "2025-03-01T14:30:00Z"
    }
  ],
  "total": 847
}
```

### `GET /api/audit/stats`

Returns aggregate permit/deny counts, rates, and breakdowns by datasource and action.

---

## Dashboard

### `GET /api/dashboard/summary` — Policy counts/rates + by_effect + by_state breakdowns
### `GET /api/dashboard/activity` — 30-day daily request array `[{date, permits, denies}]`
### `GET /api/dashboard/top-denied` — Top 5 deny reasons `[{policy_name, count}]`
### `GET /api/dashboard/datasource-breakdown` — Per-datasource `[{datasource, permits, denies}]`

---

## Settings

### `GET /api/settings` — Returns `{ [key]: value }` dict of all settings
### `PUT /api/settings/{key}` — Upsert a single setting value

**Keys:**

| Key | Example Value |
|---|---|
| `combining_algorithm` | `"deny-overrides"` |
| `anthropic_api_key` | `"sk-ant-..."` |
| `audit_retention_days` | `"90"` |

---

## AI Assist

### `POST /api/ai/suggest`

**Request:** `{ "description": "Block all agents from deleting payment records" }`

**Response:**
```json
{
  "suggestion": {
    "name": "Block AI Agents - Payment DELETE",
    "effect": "deny",
    "subject_json": { "principal_type": "agent" },
    "resource_json": { "tables": ["payments"] },
    "action_json": ["DELETE"],
    "conditions_json": {}
  },
  "message": null
}
```

### `POST /api/ai/conflicts`

**Request:** `{ "policy": {...}, "existing_policy_summaries": ["Policy A: permit analyst SELECT snowflake", ...] }`

**Response:** `{ "conflicts": [{ "policy_name": "Policy A", "reason": "Overlapping PERMIT on same datasource+action" }] }`

### `POST /api/ai/impact`

**Request:** `{ "before": {...}, "after": {...}, "datasource_stats": { "total_requests_7d": 412 } }`

**Response:** `{ "narrative": "This change will deny 12 EU data scientists..." }`

---

## Error Responses

| Status | Code | Description |
|---|---|---|
| `400` | `validation_error` | Request body fails Pydantic validation |
| `404` | `not_found` | Policy/Principal/DataSource not found |
| `409` | `conflict` | Duplicate `name` or `identifier` |
| `422` | `unprocessable` | Semantic validation failure (e.g., invalid state transition) |
| `500` | `internal_error` | Unexpected server error |

```json
{
  "detail": {
    "code": "not_found",
    "message": "Policy with id=999 not found"
  }
}
```
