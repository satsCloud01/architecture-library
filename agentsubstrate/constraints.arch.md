---
name: "Constraints & Decisions"
project: "AgentSubstrate"
project_slug: "agentsubstrate"
project_url: "https://agentsubstrate.satszone.link"
github: "https://github.com/satsCloud01/agentic-data-os"
category: "ai-agents"
type: "constraints"
icon: "🤖"
tags: [FastAPI, React, Security, Architecture Decisions]
---

# AgentSubstrate — Constraints & Architecture Decisions

## Security Constraints

### C1 — Session-Only API Key
**Decision:** AI API keys are never written to disk, localStorage, cookies, or any server.
**Rationale:** Keys are sensitive credentials. Persisting them to any storage medium creates exposure risk. The React `APIKeyContext` holds the key in component state only — it is cleared on tab close.
**Implication:** Users must re-enter their key each browser session. This is a deliberate UX trade-off for security.

### C2 — No Server-Side Key Storage
**Decision:** The backend never receives or stores API keys in its database.
**Rationale:** Backend compromise would not expose API keys. Keys are passed per-request via `X-AI-Key` header and used ephemerally.

### C3 — HITL for High-Risk Actions
**Decision:** Any agent action with risk score ≥ 0.8 requires human approval before execution.
**Rationale:** Autonomous agents can make mistakes. High-risk actions (PII deletion, schema drops, mass updates) must have a human checkpoint.

## Technical Constraints

### T1 — SQLite Database
**Decision:** SQLite with aiosqlite for async access.
**Rationale:** Single-instance deployment on consolidated EC2 keeps infrastructure simple. SQLite is sufficient for demo/single-tenant use. Migrations to PostgreSQL are straightforward when needed.
**Limitation:** No concurrent write scaling; not suitable for multi-tenant production.

### T2 — Single Docker Container
**Decision:** Frontend (built React SPA) is served as static files from the FastAPI backend.
**Rationale:** Simplifies deployment to a single container on port 8024. No separate nginx needed inside the container.
**Implication:** Frontend build happens at Docker image build time (`npm run build`).

### T3 — Mock AI Fallback
**Decision:** All AI features degrade gracefully to deterministic mock responses when no API key is provided.
**Rationale:** Platform must be fully usable without an AI subscription. AI features are enhancement, not core functionality.

### T4 — No Real Agent Execution
**Decision:** This is a UI/API demonstration platform — agents don't execute real data pipelines.
**Rationale:** Platform demonstrates the substrate architecture and UX patterns. Real agent execution would require production data infrastructure.

## Design Decisions

### D1 — 17 API Routers
**Decision:** Each domain capability has its own FastAPI router with isolated models and business logic.
**Rationale:** Clear separation of concerns. Each router can evolve independently. 17 routers map 1:1 to the 16 substrate capabilities plus the dashboard aggregator.

### D2 — Agent-First Navigation
**Decision:** Every UI capability is framed as an agent interaction.
**Rationale:** Reinforces the substrate mental model — users interact with agents, not with data forms.

### D3 — InfoTooltip on Every Section
**Decision:** Every major UI section has an ⓘ tooltip explaining the what and why.
**Rationale:** Platform concepts (A2A, MCP, HITL, circuit breakers) are not self-evident. Inline education reduces cognitive load and support burden.

### D4 — Indigo Brand Palette (matching ai-guardian)
**Decision:** Uses the same design system as ai-guardian (indigo/brand, slate backgrounds, Inter font).
**Rationale:** Consistent visual language across the Sats platform portfolio. Reusing the same CSS component classes (`.card`, `.btn-primary`, `.badge`) reduces custom styling.

## Deployment Constraints

### P1 — Port 8024
**Decision:** AgentSubstrate runs on port 8024 on the consolidated EC2 instance.
**Rationale:** Follows the established port allocation pattern: 8010 (ai-guardian) through 8023 (agent-control), incrementing by 1 per app.

### P2 — Shared EC2 Instance
**Decision:** Deployed alongside 14 other apps on a consolidated `t3.medium` EC2 instance.
**Rationale:** Cost optimization for demonstration portfolio. Not suitable for production-scale workloads.

### P3 — Let's Encrypt SSL
**Decision:** HTTPS via Certbot with auto-renewal.
**Rationale:** Free, automated, and trusted certificate authority. Certbot integrates directly with nginx.
