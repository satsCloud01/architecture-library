---
name: "API Specification"
project: "Enterprise AI Guardian"
project_slug: "ai-guardian"
project_url: "https://ai-guardian.satszone.link"
github: "https://github.com/satsCloud01/ai-governance-bank"
category: "ai-agents"
type: "api-spec"
icon: "🏦"
tags: [FastAPI, OpenAPI]
---

# API Specification — Enterprise AI Guardian

> **Standard**: OpenAPI 3.1 compatible
> **Base URL**: `http://localhost:8000`
> **Interactive Docs**: `http://localhost:8000/docs` (Swagger UI) · `http://localhost:8000/redoc`
> **Auth**: Bearer token (JWT) — _to be implemented for production_

---

## Authentication

> Production deployments must implement JWT-based authentication with role claims.
> Current demo runs without auth for pitch purposes.

```yaml
securitySchemes:
  BearerAuth:
    type: http
    scheme: bearer
    bearerFormat: JWT
```

**Role claims required per endpoint group:**

| Endpoint Group | Minimum Role |
|---|---|
| `GET /api/*` (read) | `viewer` |
| `POST /api/models` | `developer` |
| `POST /api/lifecycle/*/transition` | `mrm` |
| `POST /api/lifecycle/*/approve` | `cro` |
| `POST /api/risk/classify` | `mrm` |
| `POST /api/incidents` | `developer` |
| `DELETE /api/models/*` | `admin` |

---

## 1. Dashboard

### `GET /api/dashboard/summary`
Returns aggregate governance health metrics for the executive dashboard.

**Response `200`:**
```json
{
  "total_models": 13,
  "by_type": { "LLM": 6, "ML": 6, "Statistical": 1 },
  "by_risk_tier": { "Tier 1": 8, "Tier 2": 4, "Tier 3": 1 },
  "by_stage": {
    "Development": 1, "Validation": 1, "MRM Review": 1,
    "Production": 9, "Retired": 1
  },
  "open_incidents": 1,
  "critical_incidents": 0,
  "compliance_score": 73.4,
  "fairness_breaches": 2,
  "models_with_alerts": 3,
  "pending_approvals": 2
}
```

### `GET /api/dashboard/risk-heatmap`
Returns all models with tier, score, and stage for heatmap rendering.

**Response `200`:** Array of `RiskHeatmapItem`
```json
[
  {
    "name": "CapitalCalc IRB", "type": "STATISTICAL", "category": "capital-risk",
    "tier": "Tier 1", "score": 97, "inherent_risk": "High", "stage": "Production"
  }
]
```

---

## 2. Model Registry

### `GET /api/models`
List all models with optional filters.

**Query Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `model_type` | `string` | Filter by `LLM` \| `ML` \| `STATISTICAL` |
| `stage` | `string` | Filter by lifecycle stage |
| `risk_tier` | `string` | Filter by `Tier 1` \| `Tier 2` \| `Tier 3` |

**Response `200`:** Array of `ModelSummary`

---

### `POST /api/models`
Register a new AI model.

**Request Body:**
```json
{
  "name": "string",
  "description": "string",
  "model_type": "LLM | ML | STATISTICAL",
  "category": "string",
  "owner": "string",
  "owner_email": "string",
  "department": "string",
  "provider": "string | null",
  "base_model": "string | null",
  "prompt_injection_risk": "Low | Medium | High | null",
  "hallucination_rate": "number | null",
  "output_filtering": "boolean | null",
  "rag_enabled": "boolean | null",
  "human_in_loop_required": "boolean | null",
  "monthly_token_cost_usd": "number | null",
  "regulation_codes": ["SR11-7", "ECOA"],
  "data_sources": [
    {
      "source_name": "string",
      "source_type": "Database | API | File | Stream | External",
      "sensitivity": "Public | Internal | Confidential | Restricted",
      "pii_contained": "boolean",
      "refresh_cadence": "Real-time | Daily | Monthly | Quarterly"
    }
  ]
}
```

**Response `201`:** `ModelSummary`

---

### `GET /api/models/{model_id}`
Get full model detail with all relationships loaded.

**Path Parameters:** `model_id: integer`

**Response `200`:** `ModelDetail`
```json
{
  "id": 1, "name": "Aria", "model_type": "LLM",
  "llm_details": { "prompt_injection_risk": "High", "hallucination_rate": 3.2 },
  "risk_detail": { "tier": "Tier 1", "score": 87, "complexity_score": 82.1 },
  "lifecycle": {
    "stage": "Production",
    "approvals": [{ "approver": "Dr. Patricia Hollis", "role": "MRM", "decision": "Approved" }]
  },
  "versions": [...],
  "data_lineage": [...],
  "compliance_summary": [...],
  "open_incidents": 0
}
```

**Response `404`:** `{ "detail": "Model not found" }`

---

### `PATCH /api/models/{model_id}`
Update model metadata (owner, description, LLM details).

**Request Body:** Partial `ModelUpdate` — any subset of registration fields.

**Response `200`:** `ModelSummary`

---

### `POST /api/models/{model_id}/versions`
Add a new version to an existing model.

**Request Body:**
```json
{
  "version": "v2.1",
  "changelog": "Prompt hardening, PII redaction layer added",
  "deployed_by": "James Park"
}
```

**Response `201`:** `ModelVersion`

---

### `GET /api/models/{model_id}/audit`
Get the full immutable audit trail for a model.

**Response `200`:** Array of `AuditEntry`
```json
[
  {
    "id": 1, "actor": "Sarah Chen", "action": "model_registered",
    "detail": "Model 'Aria' registered in AI Guardian registry",
    "created_at": "2025-01-15T10:30:00"
  }
]
```

---

## 3. Risk Assessment

### `GET /api/risk/assessments`
All risk assessments sorted by score descending.

**Response `200`:** Array of `RiskAssessment`

---

### `POST /api/risk/classify`
Compute and persist a risk assessment for a model.

**Request Body:**
```json
{
  "model_id": 1,
  "complexity_score": 82.0,
  "regulatory_exposure": 90.0,
  "business_impact": 95.0,
  "data_sensitivity": 88.0,
  "reviewer": "Dr. Patricia Hollis",
  "notes": "High-risk LLM facing consumer-facing use. Prompt injection exposure significant."
}
```

**Response `201`:** `RiskAssessment`

> Risk tier is auto-calculated:
> - **Tier 1** (High): avg score ≥ 70
> - **Tier 2** (Medium): avg score 40–69
> - **Tier 3** (Low): avg score < 40

---

### `GET /api/risk/matrix`
Returns risk matrix data (1-5 impact × likelihood) for scatter plot.

**Response `200`:** Array of `RiskMatrixItem`

---

## 4. Lifecycle & Workflow

### `GET /api/lifecycle/kanban`
Returns all 6 lifecycle stages with their model cards.

**Response `200`:**
```json
{
  "stages": ["Development", "Validation", "MRM Review", "Approved", "Production", "Retired"],
  "board": {
    "Production": [
      { "id": 1, "name": "Aria", "type": "LLM", "risk_tier": "Tier 1", "blockers": null }
    ],
    "MRM Review": [
      { "id": 2, "name": "LoanAdvisor AI", "blockers": "Awaiting MRM capacity slot" }
    ]
  }
}
```

---

### `POST /api/lifecycle/{model_id}/transition`
Transition a model to the next lifecycle stage.

**Request Body:**
```json
{
  "target_stage": "Validation | MRM Review | Approved | Production | Retired",
  "actor": "string",
  "notes": "string | null"
}
```

**Business rules:**
- Stage must follow the defined sequence (no skipping)
- `Production` requires at least one `MRM` and one `CRO` approval
- `Retired` can be triggered from any active stage

**Response `200`:** `{ "stage": "Validation", "model_id": 2 }`

**Response `422`:** `{ "detail": "Cannot transition from Development to Production — MRM Review required" }`

---

### `POST /api/lifecycle/{model_id}/approve`
Record an approval (or rejection) for the current lifecycle stage.

**Request Body:**
```json
{
  "approver": "Dr. Patricia Hollis",
  "role": "MRM | CRO | CISO | Developer | Auditor",
  "decision": "Approved | Rejected | Conditional",
  "notes": "string | null"
}
```

**Response `201`:** `ApprovalRecord`

---

## 5. Compliance

### `GET /api/compliance/regulations`
All regulatory frameworks with descriptions.

**Response `200`:** Array of `Regulation`

---

### `GET /api/compliance/overview`
Per-model compliance status across all mapped regulations.

**Response `200`:** Array of `ComplianceOverview`

---

### `GET /api/compliance/gap-analysis`
Models with Gap or In Progress compliance status, sorted by gap size.

**Response `200`:** Array of `GapItem`

---

### `GET /api/compliance/{model_id}/controls`
Full control checklist for a model across all applicable regulations.

**Response `200`:** Array of `ControlMapping`

---

### `PATCH /api/compliance/controls/{control_id}`
Update a compliance control status and add evidence.

**Request Body:**
```json
{
  "status": "Pass | Fail | Pending",
  "evidence": "Validation report ref: MRM-2025-047, section 4.2",
  "due_date": "2025-06-30"
}
```

**Response `200`:** `ComplianceControl`

---

## 6. Monitoring

### `GET /api/monitoring/metrics/{model_id}`
Time-series performance metrics for a model (6 months).

**Response `200`:** Array of `PerformanceMetric`

---

### `GET /api/monitoring/fairness/{model_id}`
ECOA fairness metrics per protected class for a lending model.

**Response `200`:** Array of `FairnessMetric`

---

### `GET /api/monitoring/alerts`
All active performance and fairness alerts across all production models.

**Response `200`:**
```json
{
  "performance_alerts": [...],
  "fairness_alerts": [
    {
      "model_name": "CreditScorer v3.2",
      "protected_class": "Race",
      "di_ratio": 0.762,
      "flag": "Breach"
    }
  ],
  "total": 5
}
```

---

## 7. Incidents

### `GET /api/incidents`
All incidents sorted by severity then creation date.

**Response `200`:** Array of `Incident`

---

### `POST /api/incidents`
Create a new incident.

**Request Body:**
```json
{
  "model_id": 1,
  "title": "string",
  "description": "string",
  "severity": "Critical | High | Medium | Low",
  "category": "Bias | Drift | Hallucination | Security | Performance | Compliance",
  "reported_by": "string"
}
```

**Response `201`:** `Incident`

---

### `PATCH /api/incidents/{incident_id}`
Update incident status, assign owner, or add resolution notes.

**Request Body:**
```json
{
  "status": "Open | Investigating | Resolved | Closed",
  "assigned_to": "string | null",
  "resolution_notes": "string | null"
}
```

**Response `200`:** `Incident`

---

## Shared Schema Definitions

```typescript
// Core types (mirrors src/lib/api.ts)

interface ModelSummary {
  id: number
  name: string
  description: string
  model_type: "LLM" | "ML" | "STATISTICAL"
  category: string
  owner: string
  owner_email: string
  department: string
  provider: string | null
  base_model: string | null
  created_at: string               // ISO 8601
  llm_details: LLMDetails | null  // only populated for LLM type
  risk: RiskSummary | null
  stage: string | null
}

interface LLMDetails {
  prompt_injection_risk: "Low" | "Medium" | "High" | null
  hallucination_rate: number | null       // percentage
  output_filtering: boolean | null
  rag_enabled: boolean | null
  human_in_loop_required: boolean | null
  monthly_token_cost_usd: number | null
}

interface RiskAssessment {
  risk_tier: "Tier 1" | "Tier 2" | "Tier 3"
  risk_score: number                      // 0-100
  complexity_score: number
  regulatory_exposure: number
  business_impact: number
  data_sensitivity: number
  inherent_risk: "Low" | "Medium" | "High" | "Critical"
  residual_risk: "Low" | "Medium" | "High"
  last_reviewed: string                   // ISO 8601
  next_review: string                     // ISO 8601
  reviewer: string
}

interface FairnessMetric {
  protected_class: "Race" | "Gender" | "Age" | "National Origin"
  approval_rate_majority: number          // 0-1
  approval_rate_minority: number          // 0-1
  disparate_impact_ratio: number          // <0.8 = adverse impact (ECOA 80% rule)
  adverse_action_rate: number
  ecoa_compliant: boolean
  flag: "OK" | "Warning" | "Breach"
}
```

---

## Error Responses

| Status | Meaning |
|---|---|
| `400` | Bad request — invalid input |
| `404` | Resource not found |
| `422` | Business rule violation (e.g., invalid lifecycle transition) |
| `500` | Internal server error |

```json
{ "detail": "Human-readable error description" }
```
