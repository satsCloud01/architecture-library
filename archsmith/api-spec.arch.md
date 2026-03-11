---
name: "API Specification"
project: "ArchSmith"
project_slug: "archsmith"
project_url: "https://archsmith.satszone.link"
github: "https://github.com/satsCloud01/archsmith"
category: "architecture-devops"
type: "api-spec"
icon: "🏗️"
tags: [FastAPI, OpenAPI]
---

# API Reference

Base URL (local): `http://localhost:8002`
Base URL (production): `https://archsmith.satszone.link`

All `/api/architecture/*` endpoints require headers:
```
X-Tenant-Id: {tenantId}    # defaults to "default-tenant"
X-Role: {role}              # ARCHITECT | REVIEWER | ADMIN
```

---

## System

### GET /health
Health check.

**Response:**
```json
{"status": "ok", "service": "archsmith", "version": "1.0.0"}
```

---

## Architecture (FR1–FR6)

### POST /api/architecture/create
Create a new architecture package (FR1).

**Body:**
```json
{
  "name": "Retail Lending Platform",
  "description": "Cloud-native lending operations",
  "created_by": "architect.user",
  "requirements": [
    {"key": "FR-1", "category": "FR", "text": "...", "priority": "high"}
  ],
  "domains": [{"name": "Lending", "description": "..."}],
  "bounded_contexts": [{"domain_name": "Lending", "name": "Loan Management"}],
  "services": [{"bounded_context_name": "Loan Management", "name": "Loan Service", "service_type": "application"}],
  "apis": [{"service_name": "Loan Service", "name": "Create Loan", "method": "POST", "path": "/loans"}],
  "events": [{"name": "LoanCreated", "producer_service": "Loan Service"}],
  "data_entities": [{"name": "Loan", "owning_service": "Loan Service"}],
  "deployment_nodes": [{"name": "lending-cluster", "platform": "kubernetes"}],
  "infra_components": [{"name": "eks-cluster", "cloud": "aws"}],
  "standards": [{"source": "Basel III", "citation": "Risk framework", "license": "public"}]
}
```

**Response (201):**
```json
{"id": 1, "name": "Retail Lending Platform", "status": "draft", "version": 1}
```

---

### POST /api/architecture/{id}/generate
Generate all 8 artifacts for a package (FR2/FR4/FR5).

**Body:** `{"actor": "architect.user"}`

**Response:** `ArtifactsListResponse` — list of 8 artifact objects, each with:
```json
{
  "id": 1,
  "artifact_type": "openapi",
  "name": "OpenAPI Specification",
  "version": 2,
  "content": "openapi: 3.1.0\n...",
  "traceability": {"requirements": ["FR-1"], "adrs": ["ADR-001"], "citations": [...]},
  "created_at": "2025-01-01T00:00:00"
}
```

---

### GET /api/architecture/{id}/artifacts
Retrieve all artifacts for a package (all versions).

**Response:** Same as `/generate`

---

### POST /api/architecture/{id}/impact
Run impact analysis on a proposed changeset (FR3).

**Body:**
```json
{
  "actor": "architect.user",
  "proposed_changes": {
    "removed_services": ["Risk Engine"],
    "renamed_apis": [{"from": "CreateLoan", "to": "SubmitLoan"}],
    "modified_entities": ["LoanApplication"]
  }
}
```

**Response:**
```json
{
  "package_id": 1,
  "severity": "high",
  "breaking_changes": ["Removing service 'Risk Engine' breaks dependent APIs..."],
  "mitigation_plan": ["Deploy compatibility adapters...", "..."],
  "migration_steps": ["Implement dual-write...", "..."]
}
```

Severity: `low` (0 changes) | `medium` (1–2) | `high` (3+)

---

### POST /api/architecture/{id}/approve
Approve a package (FR6). Requires `REVIEWER` or `ADMIN` role.

**Body:** `{"actor": "reviewer.user", "comment": "Approved for prod."}`

**Response:**
```json
{
  "package_id": 1,
  "approved": true,
  "approved_by": "reviewer.user",
  "approved_at": "2025-01-01T12:00:00",
  "version": 3
}
```

**Error (403):** If role is `ARCHITECT`.

---

### GET /api/architecture
List all packages for the current tenant.

**Response:** Array of `ArchitectureSummary` objects.

---

### GET /api/architecture/dashboard/summary
Dashboard KPIs.

**Response:**
```json
{
  "total_packages": 5,
  "approved_packages": 2,
  "pending_reviews": 3,
  "avg_nfr_coverage": 42.5,
  "packages": [...]
}
```

---

## AI Suggestion

### POST /api/ai/suggest
Generate architecture elements from a plain-English description.

**Body:**
```json
{
  "description": "A cloud-native banking platform for retail customers...",
  "api_key": "sk-ant-api03-..."
}
```

**Response:**
```json
{
  "requirements": [...],
  "domains": [...],
  "bounded_contexts": [...],
  "services": [...],
  "apis": [...],
  "events": [...],
  "data_entities": [...],
  "infra_components": [...]
}
```

**Errors:**
- `400` — API key missing or description too short
- `401` — Invalid Anthropic API key
- `422` — Claude response could not be parsed as JSON
- `500` — Other AI failure

---

## Examples

### GET /api/examples
List available example templates.

**Response:**
```json
[
  {"id": "retail-banking-core", "name": "Retail Banking Core", "description": "...", "tags": ["banking", "payments"]},
  {"id": "ecommerce-platform", "name": "E-Commerce Platform", ...},
  {"id": "healthcare-data-platform", "name": "Healthcare Data Platform", ...},
  {"id": "realtime-analytics", "name": "Real-Time Analytics Platform", ...}
]
```

### GET /api/examples/{id}
Get a complete example payload ready to POST to `/api/architecture/create`.

**Example IDs:** `retail-banking-core` · `ecommerce-platform` · `healthcare-data-platform` · `realtime-analytics`
