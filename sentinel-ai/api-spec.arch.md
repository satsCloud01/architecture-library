---
name: "API Specification"
project: "Sentinel AI"
project_slug: "sentinel-ai"
project_url: "https://sentinel-ai.satszone.link"
github: "https://github.com/satsCloud01/sentinel-ai"
category: "data-analytics"
type: "api-spec"
icon: "🛡️"
tags: [FastAPI, OpenAPI]
---

# Sentinel AI — API Specification

**Data & AI Observability Platform**

Base URL: `http://localhost:8007/api`

All endpoints return JSON. AI-powered endpoints accept an optional `X-Api-Key` header for the Anthropic Claude API key. Without it, mock/fallback responses are returned.

---

## 1. Dashboard — `/api/dashboard`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/dashboard/summary` | Aggregate KPIs: total assets, monitors, open alerts, active incidents, average quality score |
| `GET` | `/api/dashboard/recent-alerts` | Last 10 alerts with severity and status |
| `GET` | `/api/dashboard/trends` | Time-series data for asset counts, alert volume, quality scores over 30 days |
| `GET` | `/api/dashboard/health` | System health: agent status counts, datasource connectivity, monitor pass rates |

### Response Schemas

**GET /api/dashboard/summary**
```json
{
  "total_assets": 142,
  "total_monitors": 87,
  "open_alerts": 12,
  "active_incidents": 3,
  "avg_quality_score": 0.87,
  "total_datasources": 8,
  "active_agents": 5
}
```

**GET /api/dashboard/trends**
```json
{
  "dates": ["2026-03-01", "2026-03-02", "..."],
  "asset_counts": [138, 140, "..."],
  "alert_counts": [5, 3, "..."],
  "quality_scores": [0.85, 0.86, "..."]
}
```

---

## 2. Assets — `/api/assets`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/assets` | List all assets. Query params: `datasource_id`, `type`, `status`, `search`, `skip`, `limit` |
| `GET` | `/api/assets/{id}` | Get asset detail including fields |
| `POST` | `/api/assets` | Create a new asset |
| `PUT` | `/api/assets/{id}` | Update an asset |
| `DELETE` | `/api/assets/{id}` | Delete an asset |
| `GET` | `/api/assets/{id}/fields` | List fields for an asset |
| `POST` | `/api/assets/{id}/fields` | Add a field to an asset |
| `GET` | `/api/assets/{id}/monitors` | List monitors attached to an asset |
| `GET` | `/api/assets/{id}/lineage` | Get upstream and downstream lineage for an asset |
| `GET` | `/api/assets/stats` | Asset statistics by type, status, and datasource |

### Request/Response Schemas

**POST /api/assets — Request**
```json
{
  "datasource_id": "uuid",
  "name": "customer_transactions",
  "type": "table",
  "schema_name": "public",
  "description": "Daily customer transaction records",
  "tags": ["finance", "pii"]
}
```

**GET /api/assets — Response**
```json
{
  "items": [
    {
      "id": "uuid",
      "datasource_id": "uuid",
      "name": "customer_transactions",
      "type": "table",
      "schema_name": "public",
      "description": "Daily customer transaction records",
      "row_count": 1250000,
      "freshness_hours": 2.5,
      "quality_score": 0.92,
      "tags": ["finance", "pii"],
      "status": "healthy",
      "last_profiled": "2026-03-10T08:00:00Z",
      "created_at": "2026-01-15T10:30:00Z",
      "updated_at": "2026-03-10T08:00:00Z"
    }
  ],
  "total": 142,
  "skip": 0,
  "limit": 20
}
```

---

## 3. Monitors — `/api/monitors`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/monitors` | List monitors. Query params: `asset_id`, `type`, `status`, `severity`, `skip`, `limit` |
| `GET` | `/api/monitors/{id}` | Get monitor detail with recent runs |
| `POST` | `/api/monitors` | Create a monitor |
| `PUT` | `/api/monitors/{id}` | Update a monitor |
| `DELETE` | `/api/monitors/{id}` | Delete a monitor |
| `POST` | `/api/monitors/{id}/run` | Trigger a manual monitor run |
| `GET` | `/api/monitors/{id}/runs` | Get run history. Query params: `skip`, `limit` |
| `PUT` | `/api/monitors/{id}/toggle` | Enable or disable a monitor |

### Request/Response Schemas

**POST /api/monitors — Request**
```json
{
  "asset_id": "uuid",
  "name": "Transaction Freshness Check",
  "type": "freshness",
  "config": {
    "max_hours": 6,
    "warn_hours": 4
  },
  "severity": "high",
  "schedule": "0 */2 * * *",
  "is_active": true
}
```

**GET /api/monitors/{id}/runs — Response**
```json
{
  "items": [
    {
      "id": "uuid",
      "monitor_id": "uuid",
      "status": "passed",
      "value": 2.5,
      "threshold": 6.0,
      "message": "Data is 2.5 hours fresh (threshold: 6h)",
      "duration_ms": 145,
      "executed_at": "2026-03-10T08:00:00Z"
    }
  ],
  "total": 50
}
```

---

## 4. Alerts — `/api/alerts`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/alerts` | List alerts. Query params: `severity`, `status`, `asset_id`, `monitor_id`, `skip`, `limit` |
| `GET` | `/api/alerts/{id}` | Get alert detail |
| `PUT` | `/api/alerts/{id}/acknowledge` | Acknowledge an alert |
| `PUT` | `/api/alerts/{id}/resolve` | Resolve an alert |
| `PUT` | `/api/alerts/{id}/suppress` | Suppress an alert |
| `GET` | `/api/alerts/stats` | Alert statistics: counts by severity, status, and time |
| `DELETE` | `/api/alerts/{id}` | Delete an alert |

### Request/Response Schemas

**PUT /api/alerts/{id}/acknowledge — Request**
```json
{
  "acknowledged_by": "jane.doe"
}
```

**GET /api/alerts — Response**
```json
{
  "items": [
    {
      "id": "uuid",
      "monitor_id": "uuid",
      "asset_id": "uuid",
      "severity": "high",
      "title": "Freshness SLA Breach",
      "message": "customer_transactions has not been updated in 8 hours (threshold: 6h)",
      "status": "open",
      "acknowledged_by": null,
      "resolved_at": null,
      "created_at": "2026-03-10T06:00:00Z"
    }
  ],
  "total": 12
}
```

---

## 5. Incidents — `/api/incidents`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/incidents` | List incidents. Query params: `severity`, `status`, `skip`, `limit` |
| `GET` | `/api/incidents/{id}` | Get incident detail with linked alerts |
| `POST` | `/api/incidents` | Create an incident |
| `PUT` | `/api/incidents/{id}` | Update incident (status, assignee, root cause) |
| `DELETE` | `/api/incidents/{id}` | Delete an incident |
| `POST` | `/api/incidents/{id}/analyze` | AI-powered root cause analysis (requires X-Api-Key) |
| `GET` | `/api/incidents/timeline` | Incident timeline: recent incidents with status transitions |

### Request/Response Schemas

**POST /api/incidents — Request**
```json
{
  "title": "Payment Pipeline Data Loss",
  "description": "Multiple freshness and volume alerts on payment tables",
  "severity": "critical",
  "alert_ids": ["uuid1", "uuid2"],
  "assigned_to": "platform-team"
}
```

**POST /api/incidents/{id}/analyze — Response**
```json
{
  "root_cause": "Upstream ETL job failed at 02:00 UTC due to Snowflake warehouse suspension",
  "impact": "3 downstream dashboards showing stale data, 2 data products below SLA",
  "recommendation": "Resume Snowflake warehouse and re-trigger ETL backfill for affected tables",
  "confidence": 0.85,
  "ai_generated": true
}
```

---

## 6. Lineage — `/api/lineage`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/lineage/graph` | Full lineage graph (nodes + edges). Query params: `domain_id` |
| `GET` | `/api/lineage/nodes` | List lineage nodes. Query params: `type`, `search` |
| `GET` | `/api/lineage/nodes/{id}` | Get node detail |
| `POST` | `/api/lineage/nodes` | Create a lineage node |
| `PUT` | `/api/lineage/nodes/{id}` | Update a lineage node |
| `DELETE` | `/api/lineage/nodes/{id}` | Delete a lineage node |
| `GET` | `/api/lineage/edges` | List lineage edges |
| `POST` | `/api/lineage/edges` | Create a lineage edge |
| `DELETE` | `/api/lineage/edges/{id}` | Delete a lineage edge |
| `GET` | `/api/lineage/impact/{node_id}` | Downstream impact analysis from a given node |
| `GET` | `/api/lineage/upstream/{node_id}` | Upstream trace from a given node |

### Request/Response Schemas

**GET /api/lineage/graph — Response**
```json
{
  "nodes": [
    {
      "id": "uuid",
      "asset_id": "uuid",
      "name": "raw_transactions",
      "type": "source",
      "metadata": {"tool": "snowflake"},
      "position_x": 100,
      "position_y": 200
    }
  ],
  "edges": [
    {
      "id": "uuid",
      "source_id": "node-uuid-1",
      "target_id": "node-uuid-2",
      "type": "data_flow",
      "label": "dbt transform"
    }
  ]
}
```

---

## 7. AI Observability — `/api/ai-observability`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/ai-observability/traces` | List AI/LLM traces. Query params: `provider`, `model`, `status`, `skip`, `limit` |
| `GET` | `/api/ai-observability/traces/{id}` | Get trace detail |
| `POST` | `/api/ai-observability/traces` | Record a new trace |
| `GET` | `/api/ai-observability/stats` | Aggregate stats: total calls, tokens, cost, avg latency by model |
| `GET` | `/api/ai-observability/trends` | Time-series: daily token usage, cost, latency, error rates |
| `GET` | `/api/ai-observability/models` | List distinct models with usage summaries |

### Request/Response Schemas

**POST /api/ai-observability/traces — Request**
```json
{
  "model": "claude-3-haiku-20240307",
  "provider": "anthropic",
  "operation": "completion",
  "input_tokens": 1500,
  "output_tokens": 350,
  "latency_ms": 820,
  "cost_usd": 0.0012,
  "status": "success",
  "metadata": {"use_case": "data_quality_insight"}
}
```

**GET /api/ai-observability/stats — Response**
```json
{
  "total_traces": 4580,
  "total_tokens": 12500000,
  "total_cost_usd": 45.20,
  "avg_latency_ms": 650,
  "error_rate": 0.02,
  "by_model": [
    {
      "model": "claude-3-haiku-20240307",
      "traces": 3200,
      "tokens": 8500000,
      "cost_usd": 25.50,
      "avg_latency_ms": 580
    }
  ]
}
```

---

## 8. Data Products — `/api/data-products`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/data-products` | List data products. Query params: `domain`, `status`, `search`, `skip`, `limit` |
| `GET` | `/api/data-products/{id}` | Get data product detail with constituent assets and SLA status |
| `POST` | `/api/data-products` | Create a data product |
| `PUT` | `/api/data-products/{id}` | Update a data product |
| `DELETE` | `/api/data-products/{id}` | Delete a data product |
| `GET` | `/api/data-products/{id}/health` | SLA compliance: freshness and quality checks for all constituent assets |

### Request/Response Schemas

**POST /api/data-products — Request**
```json
{
  "name": "Customer 360",
  "description": "Unified customer view combining transactions, profiles, and interactions",
  "owner": "data-products-team",
  "domain": "Customer Analytics",
  "sla_freshness_hours": 4,
  "sla_quality_score": 0.90,
  "asset_ids": ["uuid1", "uuid2", "uuid3"],
  "consumers": ["marketing", "risk-analytics"],
  "tags": ["tier-1", "customer"]
}
```

**GET /api/data-products/{id}/health — Response**
```json
{
  "product_id": "uuid",
  "overall_status": "warning",
  "freshness_compliant": true,
  "quality_compliant": false,
  "assets": [
    {
      "asset_id": "uuid",
      "name": "customer_profiles",
      "freshness_hours": 2.1,
      "quality_score": 0.88,
      "freshness_ok": true,
      "quality_ok": false
    }
  ]
}
```

---

## 9. Insights — `/api/insights`

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/insights/generate` | Generate AI insights for a given scope (requires X-Api-Key) |
| `GET` | `/api/insights` | List previously generated insights. Query params: `type`, `skip`, `limit` |
| `GET` | `/api/insights/{id}` | Get insight detail |
| `DELETE` | `/api/insights/{id}` | Delete an insight |
| `POST` | `/api/insights/anomaly-detect` | Run anomaly detection on recent monitor data (requires X-Api-Key) |
| `POST` | `/api/insights/recommendations` | Get AI recommendations for improving data quality (requires X-Api-Key) |

### Request/Response Schemas

**POST /api/insights/generate — Request**
```json
{
  "scope": "asset",
  "scope_id": "uuid",
  "context": "Recent quality score drop from 0.95 to 0.78"
}
```

**POST /api/insights/generate — Response**
```json
{
  "id": "uuid",
  "type": "quality_analysis",
  "title": "Quality Degradation in customer_transactions",
  "summary": "Null rate in payment_method column increased from 2% to 18% after March 8 deployment",
  "details": "...",
  "recommendations": [
    "Add NOT NULL constraint validation in upstream ETL",
    "Configure null_rate monitor with 5% threshold"
  ],
  "ai_generated": true,
  "created_at": "2026-03-10T09:00:00Z"
}
```

---

## 10. Circuit Breakers — `/api/circuit-breakers`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/circuit-breakers` | List circuit breakers. Query params: `status`, `datasource_id`, `skip`, `limit` |
| `GET` | `/api/circuit-breakers/{id}` | Get circuit breaker detail |
| `POST` | `/api/circuit-breakers` | Create a circuit breaker |
| `PUT` | `/api/circuit-breakers/{id}` | Update a circuit breaker |
| `DELETE` | `/api/circuit-breakers/{id}` | Delete a circuit breaker |
| `POST` | `/api/circuit-breakers/{id}/trip` | Manually trip a circuit breaker |
| `POST` | `/api/circuit-breakers/{id}/reset` | Reset a circuit breaker to closed state |
| `POST` | `/api/circuit-breakers/{id}/half-open` | Set circuit breaker to half-open (testing) |

### Request/Response Schemas

**POST /api/circuit-breakers — Request**
```json
{
  "name": "Snowflake Payment Pipeline Breaker",
  "datasource_id": "uuid",
  "asset_id": "uuid",
  "trip_condition": {
    "monitor_failures": 3,
    "severity_min": "high"
  },
  "failure_threshold": 3,
  "cooldown_minutes": 30
}
```

**GET /api/circuit-breakers/{id} — Response**
```json
{
  "id": "uuid",
  "name": "Snowflake Payment Pipeline Breaker",
  "datasource_id": "uuid",
  "asset_id": "uuid",
  "status": "closed",
  "trip_condition": {"monitor_failures": 3, "severity_min": "high"},
  "failure_count": 1,
  "failure_threshold": 3,
  "last_tripped": null,
  "cooldown_minutes": 30,
  "created_at": "2026-02-20T14:00:00Z"
}
```

---

## 11. Integrations — `/api/integrations`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/integrations` | List all integrations |
| `GET` | `/api/integrations/{id}` | Get integration detail |
| `POST` | `/api/integrations` | Create an integration |
| `PUT` | `/api/integrations/{id}` | Update an integration |
| `DELETE` | `/api/integrations/{id}` | Delete an integration |
| `POST` | `/api/integrations/{id}/test` | Test connectivity for an integration |
| `PUT` | `/api/integrations/{id}/toggle` | Enable or disable an integration |

### Request/Response Schemas

**POST /api/integrations — Request**
```json
{
  "name": "Slack Alerts Channel",
  "type": "slack",
  "config": {
    "webhook_url": "https://hooks.slack.com/services/...",
    "channel": "#data-alerts",
    "notify_severity": ["critical", "high"]
  },
  "is_active": true
}
```

**POST /api/integrations/{id}/test — Response**
```json
{
  "status": "success",
  "message": "Successfully connected to Slack webhook",
  "latency_ms": 230
}
```

---

## 12. Settings — `/api/settings`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/settings` | List all settings. Query params: `category` |
| `GET` | `/api/settings/{key}` | Get a specific setting by key |
| `PUT` | `/api/settings/{key}` | Update a setting value |
| `POST` | `/api/settings` | Create a new setting |
| `DELETE` | `/api/settings/{key}` | Delete a setting |
| `GET` | `/api/settings/categories` | List distinct setting categories |

### Request/Response Schemas

**PUT /api/settings/{key} — Request**
```json
{
  "value": "60",
  "description": "Default monitor check interval in minutes"
}
```

**GET /api/settings — Response**
```json
{
  "items": [
    {
      "id": "uuid",
      "key": "default_monitor_interval",
      "value": "60",
      "category": "monitors",
      "description": "Default monitor check interval in minutes",
      "updated_at": "2026-03-10T09:00:00Z"
    }
  ]
}
```

---

## 13. Agents — `/api/agents`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/agents` | List registered agents. Query params: `agent_type`, `status`, `skip`, `limit` |
| `GET` | `/api/agents/{id}` | Get agent detail |
| `POST` | `/api/agents/register` | Register a new agent |
| `PUT` | `/api/agents/{id}` | Update agent info |
| `DELETE` | `/api/agents/{id}` | Deregister an agent |
| `POST` | `/api/agents/{id}/heartbeat` | Record agent heartbeat |
| `POST` | `/api/agents/generate` | Generate an agent deployment script for a given type |
| `GET` | `/api/agents/types` | List all 10 supported agent types with descriptions |

### Request/Response Schemas

**POST /api/agents/register — Request**
```json
{
  "name": "prod-snowflake-agent-01",
  "agent_type": "snowflake",
  "datasource_id": "uuid",
  "hostname": "etl-server-01.internal",
  "version": "1.0.0",
  "config": {
    "account": "xy12345.us-east-1",
    "warehouse": "COMPUTE_WH",
    "poll_interval_seconds": 300
  }
}
```

**POST /api/agents/generate — Request**
```json
{
  "agent_type": "snowflake",
  "config": {
    "sentinel_url": "https://sentinel-ai.example.com",
    "agent_name": "prod-snowflake-agent-01",
    "account": "xy12345.us-east-1",
    "warehouse": "COMPUTE_WH",
    "poll_interval": 300
  }
}
```

**POST /api/agents/generate — Response**
```json
{
  "agent_type": "snowflake",
  "filename": "sentinel_snowflake_agent.py",
  "script": "#!/usr/bin/env python3\n# Sentinel AI - Snowflake Agent\n# Generated: 2026-03-10\n...",
  "instructions": "1. Save this script to your ETL server\n2. Install dependencies: pip install httpx snowflake-connector-python\n3. Run: python sentinel_snowflake_agent.py"
}
```

**GET /api/agents/types — Response**
```json
[
  {"type": "database", "description": "Monitor PostgreSQL, MySQL, SQL Server, Oracle databases"},
  {"type": "file_storage", "description": "Monitor S3, GCS, Azure Blob, local filesystem"},
  {"type": "ai_llm", "description": "Trace LLM API calls (OpenAI, Anthropic, Cohere, etc.)"},
  {"type": "dbt", "description": "Monitor dbt Core/Cloud projects, models, and tests"},
  {"type": "airflow", "description": "Monitor Apache Airflow DAGs, tasks, and SLAs"},
  {"type": "databricks", "description": "Monitor Databricks workspaces, jobs, and clusters"},
  {"type": "snowflake", "description": "Monitor Snowflake accounts, warehouses, and query history"},
  {"type": "athena", "description": "Monitor AWS Athena queries and Glue catalog"},
  {"type": "aws_datalake", "description": "Monitor S3 data lakes with Glue and Lake Formation"},
  {"type": "custom", "description": "User-defined agent with custom metric collectors"}
]
```

---

## Common Patterns

### Pagination

All list endpoints support pagination:

```
GET /api/{resource}?skip=0&limit=20
```

Response includes `total` count for client-side pagination.

### Error Responses

```json
{
  "detail": "Asset not found",
  "status_code": 404
}
```

Standard HTTP status codes: 200 (OK), 201 (Created), 400 (Bad Request), 404 (Not Found), 422 (Validation Error), 500 (Internal Server Error).

### AI-Powered Endpoints

Endpoints that use Claude API accept the `X-Api-Key` header:

```
POST /api/insights/generate
X-Api-Key: sk-ant-...
```

Without the header, these endpoints return deterministic mock/fallback responses.
