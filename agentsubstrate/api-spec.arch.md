---
name: "API Specification"
project: "AgentSubstrate"
project_slug: "agentsubstrate"
project_url: "https://agentsubstrate.satszone.link"
github: "https://github.com/satsCloud01/agentic-data-os"
category: "ai-agents"
type: "api-spec"
icon: "🤖"
tags: [FastAPI, REST API, OpenAPI, Agent Orchestration]
---

# AgentSubstrate — API Specification

**Base URL:** `/api`
**Format:** JSON
**Auth:** Session-only AI key via `X-AI-Key` header (never persisted)

## API Routers (17 total)

| Router | Prefix | Description |
|---|---|---|
| agents | `/api/agents` | Agent CRUD, status, task history |
| pipelines | `/api/pipelines` | Pipeline management, runs, metrics |
| orchestration | `/api/orchestration` | Intent routing, task dispatch |
| data_catalog | `/api/catalog` | Asset management, lineage, search |
| semantic | `/api/semantic` | Glossary entities, conflict resolution |
| governance | `/api/governance` | Policies, violations, compliance scoring |
| observability | `/api/observability` | Metrics, traces, incidents |
| circuit_breakers | `/api/circuit-breakers` | State management, trip/reset |
| finops | `/api/finops` | Cost attribution, chargeback, optimization |
| hitl | `/api/hitl` | HITL queue, approval/rejection |
| memory | `/api/memory` | Agent memory CRUD, retrieval |
| mcp_tools | `/api/mcp` | MCP tool registry, bindings |
| a2a | `/api/a2a` | A2A message routing, protocol hub |
| guardrails | `/api/guardrails` | Guardrail evaluation, rule management |
| connectors | `/api/connectors` | Data source connections |
| dashboard | `/api/dashboard` | Aggregated metrics for dashboard |
| settings | `/api/settings` | Platform configuration |

## Key Endpoints

### Agent Orchestration
```
GET    /api/orchestration/tasks          — list all orchestration tasks
POST   /api/orchestration/dispatch       — dispatch new intent to router agent
GET    /api/orchestration/tasks/{id}     — get task detail and result
PATCH  /api/orchestration/tasks/{id}     — update task status
```

### HITL Queue
```
GET    /api/hitl/tasks                   — list pending HITL tasks
GET    /api/hitl/tasks/{id}              — get task detail
POST   /api/hitl/tasks/{id}/approve      — approve with optional comment
POST   /api/hitl/tasks/{id}/reject       — reject with reason
POST   /api/hitl/tasks/{id}/escalate     — escalate to senior reviewer
```

### Guardrails
```
GET    /api/guardrails/rules             — list all guardrail rules
POST   /api/guardrails/evaluate          — evaluate action against guardrails
GET    /api/guardrails/violations        — list recent violations
PUT    /api/guardrails/rules/{id}        — update rule config
```

### Circuit Breakers
```
GET    /api/circuit-breakers/            — list all breakers with state
POST   /api/circuit-breakers/{id}/trip   — manually trip a breaker
POST   /api/circuit-breakers/{id}/reset  — reset to CLOSED state
GET    /api/circuit-breakers/{id}/history — state transition history
```

### A2A Hub
```
GET    /api/a2a/messages                 — list recent messages
POST   /api/a2a/messages                 — send A2A message
GET    /api/a2a/agents                   — list registered agents and capabilities
POST   /api/a2a/capability-query         — query agent capabilities
```

## AI-Powered Endpoints

All AI endpoints require `X-AI-Key` header. Fall back to mock data if key absent.

```
POST   /api/semantic/ai-suggest          — NLP-based glossary suggestions
POST   /api/governance/ai-policy         — AI-generated policy from natural language
POST   /api/observability/ai-insights    — anomaly detection + recommendations
POST   /api/guardrails/ai-evaluate       — AI-assisted guardrail evaluation
POST   /api/finops/ai-optimize           — AI cost optimization recommendations
```

## Request/Response Patterns

### Dispatch Intent
```json
POST /api/orchestration/dispatch
{
  "intent": "Run quality check on customer_transactions table",
  "priority": "high",
  "context": { "table": "customer_transactions", "schema": "public" }
}

Response 201:
{
  "task_id": 42,
  "routed_to": "quality",
  "confidence": 0.94,
  "estimated_duration": "2-5 minutes",
  "hitl_required": false
}
```

### Guardrail Evaluation
```json
POST /api/guardrails/evaluate
{
  "action": "delete_table",
  "agent": "pipeline",
  "context": { "table": "customer_pii", "row_count": 1500000 }
}

Response 200:
{
  "decision": "BLOCK",
  "triggered_rules": ["pii-protection", "mass-delete-threshold"],
  "risk_score": 0.97,
  "reason": "PII deletion requires HITL approval",
  "hitl_task_id": 15
}
```
