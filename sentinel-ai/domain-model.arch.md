---
name: "Domain Model"
project: "Sentinel AI"
project_slug: "sentinel-ai"
project_url: "https://sentinel-ai.satszone.link"
github: "https://github.com/satsCloud01/sentinel-ai"
category: "data-analytics"
type: "domain-model"
icon: "🛡️"
tags: [SQLAlchemy, Pydantic]
---

# Sentinel AI — Domain Model

**Data & AI Observability Platform**

This document describes the 16 domain entities, their key fields, and relationships.

---

## Entity Relationship Overview

```
Domain 1───* DataSource 1───* Asset 1───* AssetField
                                 |
                    +------------+------------+
                    |            |            |
               Monitor 1──* MonitorRun   LineageNode
                    |                        |
                 Alert                  LineageEdge
                    |
                Incident

DataProduct ───* Asset (many-to-many via product_assets)

CircuitBreaker ───> DataSource

AITrace (standalone — traces LLM calls)

Integration (standalone — external connections)

Setting (standalone — key/value config)

AgentRegistration ───> DataSource (optional link)
```

---

## Entity Definitions

### 1. Domain

Top-level organizational grouping (e.g., "Risk Analytics", "Customer Data").

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `name` | str | Domain name |
| `description` | str | Purpose and scope |
| `owner` | str | Responsible team or person |
| `status` | str | `active`, `archived` |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last modification |

**Relationships:** One Domain has many DataSources.

---

### 2. DataSource

A connection to an external data system (database, data lake, API, file store).

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `domain_id` | str (FK) | Parent domain |
| `name` | str | Display name |
| `type` | str | `database`, `file_storage`, `api`, `streaming`, `data_lake` |
| `connection_info` | JSON | Connection metadata (sanitized, no secrets) |
| `status` | str | `connected`, `disconnected`, `error` |
| `last_sync` | datetime | Last successful sync |
| `created_at` | datetime | Creation timestamp |

**Relationships:** Belongs to Domain. Has many Assets. Optionally linked from AgentRegistration and CircuitBreaker.

---

### 3. Asset

A data asset discovered or registered in the platform (table, view, file, model, topic).

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `datasource_id` | str (FK) | Parent data source |
| `name` | str | Asset name (e.g., table name) |
| `type` | str | `table`, `view`, `file`, `model`, `topic`, `dataset` |
| `schema_name` | str | Schema or namespace |
| `description` | str | Human-readable description |
| `row_count` | int | Last known row count |
| `freshness_hours` | float | Hours since last update |
| `quality_score` | float | 0.0–1.0 composite quality score |
| `tags` | JSON | Freeform tags list |
| `status` | str | `healthy`, `warning`, `critical`, `unknown` |
| `last_profiled` | datetime | Last profiling timestamp |
| `created_at` | datetime | Discovery timestamp |
| `updated_at` | datetime | Last modification |

**Relationships:** Belongs to DataSource. Has many AssetFields, Monitors, and LineageNodes. Linked to DataProducts via association.

---

### 4. AssetField

Column or field-level metadata for an asset.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `asset_id` | str (FK) | Parent asset |
| `name` | str | Field/column name |
| `data_type` | str | Data type (e.g., `VARCHAR`, `INTEGER`, `TIMESTAMP`) |
| `nullable` | bool | Whether the field allows nulls |
| `is_pii` | bool | Flagged as personally identifiable information |
| `description` | str | Field description |
| `distinct_count` | int | Number of distinct values |
| `null_percentage` | float | Percentage of null values |
| `sample_values` | JSON | Sample values for preview |

**Relationships:** Belongs to Asset.

---

### 5. Monitor

A data quality or health check configured against an asset.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `asset_id` | str (FK) | Monitored asset |
| `name` | str | Monitor name |
| `type` | str | `freshness`, `volume`, `schema_change`, `custom_sql`, `null_rate`, `uniqueness`, `distribution` |
| `config` | JSON | Monitor-specific configuration (thresholds, SQL, etc.) |
| `severity` | str | `critical`, `high`, `medium`, `low` |
| `schedule` | str | Cron expression or interval |
| `is_active` | bool | Whether monitor is enabled |
| `last_run` | datetime | Last execution time |
| `last_status` | str | `passed`, `failed`, `warning`, `error` |
| `created_at` | datetime | Creation timestamp |

**Relationships:** Belongs to Asset. Has many MonitorRuns. Generates Alerts.

---

### 6. MonitorRun

A single execution record for a monitor.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `monitor_id` | str (FK) | Parent monitor |
| `status` | str | `passed`, `failed`, `warning`, `error` |
| `value` | float | Measured value (e.g., row count, null %) |
| `threshold` | float | Threshold that was evaluated against |
| `message` | str | Human-readable result message |
| `duration_ms` | int | Execution duration in milliseconds |
| `executed_at` | datetime | Execution timestamp |

**Relationships:** Belongs to Monitor.

---

### 7. Alert

A notification generated when a monitor detects an anomaly.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `monitor_id` | str (FK) | Source monitor |
| `asset_id` | str (FK) | Affected asset |
| `severity` | str | `critical`, `high`, `medium`, `low` |
| `title` | str | Alert title |
| `message` | str | Detailed alert message |
| `status` | str | `open`, `acknowledged`, `resolved`, `suppressed` |
| `acknowledged_by` | str | User who acknowledged |
| `resolved_at` | datetime | Resolution timestamp |
| `created_at` | datetime | Alert trigger timestamp |

**Relationships:** Belongs to Monitor and Asset. Can be linked to an Incident.

---

### 8. Incident

A tracked investigation triggered by one or more alerts.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `title` | str | Incident title |
| `description` | str | Detailed description |
| `severity` | str | `critical`, `high`, `medium`, `low` |
| `status` | str | `open`, `investigating`, `mitigated`, `resolved`, `closed` |
| `root_cause` | str | Root cause analysis (can be AI-generated) |
| `assigned_to` | str | Assigned responder |
| `alert_ids` | JSON | List of related alert IDs |
| `impact` | str | Business impact description |
| `started_at` | datetime | Incident start time |
| `resolved_at` | datetime | Resolution timestamp |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update |

**Relationships:** Links to multiple Alerts.

---

### 9. LineageNode

A node in the data lineage graph representing a data asset or transformation.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `asset_id` | str (FK, nullable) | Linked asset (if applicable) |
| `name` | str | Node display name |
| `type` | str | `source`, `transformation`, `destination`, `model`, `dashboard` |
| `metadata` | JSON | Additional properties (tool, owner, etc.) |
| `position_x` | float | X coordinate for graph layout |
| `position_y` | float | Y coordinate for graph layout |

**Relationships:** Optionally linked to Asset. Connected via LineageEdges.

---

### 10. LineageEdge

A directed edge in the data lineage graph.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `source_id` | str (FK) | Source LineageNode |
| `target_id` | str (FK) | Target LineageNode |
| `type` | str | `data_flow`, `transformation`, `dependency` |
| `label` | str | Edge label (e.g., transformation name) |
| `metadata` | JSON | Additional edge properties |

**Relationships:** Connects two LineageNodes (source -> target).

---

### 11. DataProduct

A curated, business-oriented data product composed of multiple assets.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `name` | str | Product name |
| `description` | str | Business description |
| `owner` | str | Product owner |
| `domain` | str | Business domain |
| `status` | str | `draft`, `active`, `deprecated` |
| `sla_freshness_hours` | float | Freshness SLA in hours |
| `sla_quality_score` | float | Minimum quality score SLA |
| `asset_ids` | JSON | List of constituent asset IDs |
| `consumers` | JSON | List of downstream consumer teams |
| `tags` | JSON | Classification tags |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last modification |

**Relationships:** Links to multiple Assets.

---

### 12. CircuitBreaker

A protection mechanism that halts data pipelines when critical failures are detected.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `name` | str | Circuit breaker name |
| `datasource_id` | str (FK) | Protected data source |
| `asset_id` | str (FK, nullable) | Specific asset (or entire source) |
| `status` | str | `closed` (healthy), `open` (tripped), `half_open` (testing) |
| `trip_condition` | JSON | Conditions that trip the breaker |
| `failure_count` | int | Current consecutive failure count |
| `failure_threshold` | int | Failures required to trip |
| `last_tripped` | datetime | Last trip timestamp |
| `cooldown_minutes` | int | Minutes before auto half-open |
| `created_at` | datetime | Creation timestamp |

**Relationships:** Belongs to DataSource. Optionally linked to Asset.

---

### 13. AITrace

A recorded trace of an AI/LLM API call for observability.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `model` | str | Model name (e.g., `claude-3-haiku`) |
| `provider` | str | Provider (e.g., `anthropic`, `openai`) |
| `operation` | str | Operation type (e.g., `completion`, `embedding`) |
| `input_tokens` | int | Input token count |
| `output_tokens` | int | Output token count |
| `total_tokens` | int | Total token count |
| `latency_ms` | int | Response latency in milliseconds |
| `cost_usd` | float | Estimated cost in USD |
| `status` | str | `success`, `error`, `timeout` |
| `error_message` | str | Error details (if failed) |
| `metadata` | JSON | Additional trace properties |
| `created_at` | datetime | Trace timestamp |

**Relationships:** Standalone entity.

---

### 14. Integration

An external service connection for alerts, notifications, or data sync.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `name` | str | Integration name |
| `type` | str | `slack`, `pagerduty`, `email`, `webhook`, `jira`, `teams` |
| `config` | JSON | Integration configuration (endpoints, channels) |
| `is_active` | bool | Whether integration is enabled |
| `last_health_check` | datetime | Last connectivity check |
| `health_status` | str | `healthy`, `degraded`, `unreachable` |
| `created_at` | datetime | Creation timestamp |

**Relationships:** Standalone entity.

---

### 15. Setting

Application-level key/value configuration.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `key` | str (unique) | Setting key |
| `value` | str | Setting value |
| `category` | str | Grouping category (e.g., `general`, `notifications`, `ai`) |
| `description` | str | Human-readable description |
| `updated_at` | datetime | Last modification |

**Relationships:** Standalone entity.

---

### 16. AgentRegistration

A record of a deployed lightweight agent reporting into the platform.

| Field | Type | Description |
|---|---|---|
| `id` | str (UUID) | Primary key |
| `name` | str | Agent display name |
| `agent_type` | str | One of 10 types: `database`, `file_storage`, `ai_llm`, `dbt`, `airflow`, `databricks`, `snowflake`, `athena`, `aws_datalake`, `custom` |
| `datasource_id` | str (FK, nullable) | Linked data source |
| `status` | str | `active`, `inactive`, `error` |
| `version` | str | Agent script version |
| `hostname` | str | Host where agent is deployed |
| `last_heartbeat` | datetime | Last heartbeat received |
| `config` | JSON | Agent configuration snapshot |
| `registered_at` | datetime | Registration timestamp |

**Relationships:** Optionally linked to DataSource.

---

## Entity Summary Table

| # | Entity | Table Name | Count of Key Fields | Primary Relationships |
|---|---|---|---|---|
| 1 | Domain | `domains` | 7 | Parent of DataSources |
| 2 | DataSource | `datasources` | 8 | Belongs to Domain; parent of Assets |
| 3 | Asset | `assets` | 14 | Belongs to DataSource; parent of Fields, Monitors |
| 4 | AssetField | `asset_fields` | 10 | Belongs to Asset |
| 5 | Monitor | `monitors` | 11 | Belongs to Asset; parent of MonitorRuns |
| 6 | MonitorRun | `monitor_runs` | 8 | Belongs to Monitor |
| 7 | Alert | `alerts` | 10 | Belongs to Monitor and Asset |
| 8 | Incident | `incidents` | 12 | Links to Alerts |
| 9 | LineageNode | `lineage_nodes` | 7 | Linked to Asset; connected via Edges |
| 10 | LineageEdge | `lineage_edges` | 6 | Connects LineageNodes |
| 11 | DataProduct | `data_products` | 12 | Links to Assets |
| 12 | CircuitBreaker | `circuit_breakers` | 11 | Belongs to DataSource |
| 13 | AITrace | `ai_traces` | 13 | Standalone |
| 14 | Integration | `integrations` | 8 | Standalone |
| 15 | Setting | `settings` | 5 | Standalone |
| 16 | AgentRegistration | `agent_registrations` | 10 | Links to DataSource |
