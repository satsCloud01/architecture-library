---
name: "C1 – System Context"
project: "AgentSubstrate"
project_slug: "agentsubstrate"
project_url: "https://agentsubstrate.satszone.link"
github: "https://github.com/satsCloud01/agentic-data-os"
category: "ai-agents"
type: "c4-context"
icon: "🤖"
tags: [FastAPI, React, Claude API, Agent Orchestration, Data Ops]
---

# AgentSubstrate — Architecture Documentation

**AI-Native Agent Substrate Platform**

AgentSubstrate is a composable, AI-native platform where autonomous agents build, govern, monitor, and optimize your entire data operation — without manual intervention.

---

## C1 — System Context

```
+---------------------------------------------------+
|                  Platform Users                   |
|     (Data Engineers, Platform Admins, FinOps,     |
|      Governance Officers, Data Stewards)          |
+----------------------------+----------------------+
                             |
                        HTTPS (Browser)
                             |
                             v
              +------------------------------+
              |        AgentSubstrate        |
              |  AI-Native Agent Substrate   |
              |         Platform             |
              +-----+----------------+-------+
                    |                |
          +---------+                +---------+
          v                                    v
+------------------+              +------------------+
| Anthropic        |              | External Alert   |
| Claude API       |              | Channels         |
| (AI features,    |              | (Slack, PagerDuty|
|  guardrail eval, |              |  Email, Teams)   |
|  semantic NLP)   |              +------------------+
+------------------+
```

### Actors

| Actor | Role |
|---|---|
| **Data Engineers** | Build and monitor data pipelines, manage circuit breakers |
| **Platform Admins** | Orchestrate agents, configure guardrails, manage MCP registry |
| **FinOps Teams** | Track agent costs, chargeback attribution, optimize spend |
| **Governance Officers** | Define policies, manage compliance, review audit trails |
| **Data Stewards** | Manage semantic registry, resolve entity conflicts, govern catalog |

### External Systems

| System | Integration |
|---|---|
| **Anthropic Claude API** | Powers AI features: semantic NLP, guardrail evaluation, anomaly detection, governance insights — BYOK, session-only |
| **Alert Channels** | Slack, PagerDuty, email, Teams for HITL escalations and incident alerts |

---

## Key Capabilities

| Capability | Description |
|---|---|
| **Agent Orchestration** | Router agent dispatches tasks to 6 specialized domain agents |
| **Pipeline Studio** | Build, run, and self-heal data pipelines with circuit breaker integration |
| **Semantic Registry** | Business glossary as source of truth with conflict resolution engine |
| **Governance** | Policy-as-code evaluation with real-time compliance scoring |
| **Observability** | Agent telemetry, pipeline health monitoring, incident management |
| **HITL Queue** | Human-in-the-loop approval workflow for high-risk agent actions |
| **A2A Hub** | Agent-to-Agent protocol for cross-agent communication |
| **MCP Registry** | Model Context Protocol tool registry for agent capabilities |
| **Agent Memory** | Episodic, semantic, and working memory for persistent agent context |
| **Guardrails** | 8 guardrail categories with configurable strictness and auto-remediation |
| **FinOps** | Cost attribution, chargeback, and optimization insights per agent |
| **Circuit Breakers** | OPEN/CLOSED/HALF_OPEN state machine protecting downstream systems |
