---
name: "Domain Model"
project: "AgentSubstrate"
project_slug: "agentsubstrate"
project_url: "https://agentsubstrate.satszone.link"
github: "https://github.com/satsCloud01/agentic-data-os"
category: "ai-agents"
type: "domain-model"
icon: "🤖"
tags: [FastAPI, React, Claude API, Agent Orchestration, Data Ops]
---

# AgentSubstrate — Domain Model

## Core Entities

### Agent
Represents an autonomous agent in the substrate. Each agent has a type, status, and set of capabilities.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Agent display name |
| type | str | router, pipeline, quality, governance, catalog, semantic, finops |
| status | str | active, idle, error, paused |
| description | str | Agent purpose and capabilities |
| tasks_completed | int | Lifetime task count |
| uptime | float | Availability percentage |
| last_active | datetime | Last task execution |

### Pipeline
A data pipeline managed by the Pipeline Agent with circuit breaker integration.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Pipeline name |
| status | str | running, completed, failed, paused |
| source | str | Source system/table |
| destination | str | Target system/table |
| schedule | str | Cron expression |
| records_processed | int | Total records processed |
| success_rate | float | Historical success rate |

### OrchestrationTask
A task routed by the Router Agent to a domain agent.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| intent | str | Natural language task description |
| routed_to | str | Target agent type |
| status | str | pending, running, completed, failed, hitl_required |
| confidence | float | Routing confidence score |
| result | JSON | Task execution result |

### CircuitBreaker
Protects downstream systems using OPEN/CLOSED/HALF_OPEN state machine.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Breaker name |
| state | str | CLOSED, OPEN, HALF_OPEN |
| failure_threshold | int | Failures before opening |
| failure_count | int | Current failure count |
| pipeline_id | int | FK → Pipeline |

### GovernancePolicy
A policy-as-code rule evaluated in real-time against agent actions.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Policy name |
| category | str | data_quality, access_control, retention, compliance, lineage |
| severity | str | critical, high, medium, low |
| status | str | active, draft, deprecated |
| rule_logic | str | Policy expression |
| compliance_score | float | Current compliance % |

### SemanticEntity
An entry in the business glossary — source of truth for semantic meaning.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Entity name |
| type | str | metric, dimension, attribute, concept, kpi |
| domain | str | Business domain |
| definition | str | Authoritative definition |
| status | str | approved, pending, conflict, deprecated |
| owner | str | Responsible team/person |

### HITLTask
A human-in-the-loop approval task escalated from an agent action.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| title | str | Task description |
| agent | str | Requesting agent |
| priority | str | critical, high, medium, low |
| status | str | pending, approved, rejected, escalated |
| risk_score | float | Automated risk assessment |
| action_required | str | What the human must decide |

### AgentMemory
Persistent memory entry for an agent, supporting episodic/semantic/working types.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| agent | str | Owning agent |
| type | str | episodic, semantic, working, procedural |
| content | str | Memory content |
| importance | float | Relevance score |
| ttl | int | Time-to-live in seconds (null = permanent) |

### MCPTool
A tool registered in the Model Context Protocol registry.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| name | str | Tool name |
| category | str | data_access, transformation, governance, external_api |
| status | str | active, inactive, deprecated |
| schema | JSON | Input/output JSON schema |
| agent_bindings | list[str] | Agents authorized to use this tool |

### A2AMessage
An Agent-to-Agent protocol message for cross-agent communication.

| Field | Type | Description |
|---|---|---|
| id | int | Primary key |
| from_agent | str | Sender agent |
| to_agent | str | Recipient agent |
| message_type | str | task_delegation, status_update, data_transfer, capability_query |
| payload | JSON | Message data |
| status | str | sent, delivered, processed, failed |

## Relationships

```
Agent ──→ OrchestrationTask (routes)
Agent ──→ HITLTask (escalates)
Agent ──→ AgentMemory (stores)
Agent ──→ MCPTool (uses, via bindings)
Agent ──→ A2AMessage (sends/receives)
Pipeline ──→ CircuitBreaker (protected by)
Pipeline ──→ OrchestrationTask (triggered by)
GovernancePolicy ──→ ComplianceViolation (generates)
SemanticEntity ──→ ConflictRecord (may have)
```
