---
name: "Domain Model"
project: "DataGuardian"
project_slug: "data-guardian"
project_url: "https://data-guardian.satszone.link"
github: "https://github.com/satsCloud01/data-guardian"
category: "data-analytics"
type: "domain-model"
icon: "🛡️"
tags: [SQLAlchemy, Pydantic]
---

# Domain Model

## Bounded Contexts

DataGuardian is organised into six bounded contexts:

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Schema Context  │  │ Lineage Context  │  │ Governance Ctx  │
│                 │  │                 │  │                 │
│ DataModel       │◄─┤ LineageLink     │  │ ChangeRequest   │
│ ModelVersion    │  │ (src→tgt edges) │  │ ChangeReview    │
│ DataField       │  └─────────────────┘  └─────────────────┘
└────────┬────────┘
         │
         ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Glossary Context │  │Compliance Context│  │ Quality Context  │
│                 │  │                 │  │                 │
│ GlossaryTerm    │  │GovernanceStandard│  │ QualityMetric   │
│ FieldGlossaryLnk│  │ComplianceMapping │  │ DataIssue       │
└─────────────────┘  │ComplianceControl │  └─────────────────┘
                     └─────────────────┘
```

---

## Entity Relationship Diagram

```
data_models
  ├── id (PK)
  ├── name (UNIQUE)
  ├── display_name
  ├── model_type        [Relational | Document | Event]
  ├── domain
  ├── owner / owner_email / department / steward
  ├── sensitivity_tier  [Restricted | Confidential | Internal]
  ├── status            [Draft | Review | Approved | Active | Deprecated | Archived]
  ├── quality_score (0–100, computed)
  ├── field_count / documented_fields / pii_field_count (computed)
  ├── record_count_estimate / refresh_cadence / source_system
  └── created_at / updated_at

model_versions (1:N from data_models)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── version_number     e.g. "v1.0", "v2.0"
  ├── changelog
  ├── created_by
  ├── is_current         [only one TRUE per model]
  └── created_at

data_fields (1:N from model_versions)
  ├── id (PK)
  ├── version_id (FK → model_versions)
  ├── name / data_type / nullable
  ├── is_pk / is_fk / fk_reference
  ├── description / example_value
  ├── is_pii / pii_type   [Name | Email | Phone | SSN | DOB | Address | Financial | ...]
  └── glossary_term_id (FK → glossary_terms, nullable)

lineage_links (M:N between data_models)
  ├── id (PK)
  ├── source_model_id (FK → data_models)
  ├── target_model_id (FK → data_models)
  ├── relationship_type  [feeds_into | derives_from | replicates | aggregates | enriches | filters | joins | archives]
  ├── source_field_name (optional)
  └── target_field_name (optional)

change_requests (1:N from data_models)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── title / description / proposed_by
  ├── change_type        [Additive | Breaking | Documentation | Structural]
  ├── breaking_changes (JSON text)
  ├── status             [Open | In Review | Approved | Rejected | Implemented]
  └── created_at / resolved_at

change_reviews (1:N from change_requests)
  ├── id (PK)
  ├── change_request_id (FK → change_requests)
  ├── reviewer / role / decision / comment
  └── created_at

glossary_terms
  ├── id (PK)
  ├── term / definition / domain
  ├── synonyms (JSON)
  ├── examples (JSON)
  ├── created_by
  └── created_at

field_glossary_links (M:N between data_fields and glossary_terms)
  ├── id (PK)
  ├── field_id (FK → data_fields)
  └── term_id (FK → glossary_terms)

governance_standards
  ├── id (PK)
  ├── code (UNIQUE)   e.g. "BCBS-239", "GDPR"
  ├── name / authority / jurisdiction / description
  └── applies_to (JSON array of model types)

compliance_mappings (M:N between data_models and governance_standards)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── standard_id (FK → governance_standards)
  ├── status             [Not Mapped | Compliant | Partial | Gap]
  ├── controls_total / controls_passed (computed)
  └── last_assessed

compliance_controls (1:N from compliance_mappings)
  ├── id (PK)
  ├── mapping_id (FK → compliance_mappings)
  ├── control_name
  ├── status             [Pending | Passed | Failed | Not Applicable]
  └── evidence (text)

quality_metrics (time-series, 1:N from data_models)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── measured_at
  ├── completeness_score / validity_score / freshness_score (0–100)
  └── overall_score (weighted average)

data_issues (1:N from data_models)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── title / description
  ├── severity           [Critical | High | Medium | Low]
  ├── category           [Completeness | Validity | Accuracy | Timeliness | Consistency | Uniqueness]
  ├── status             [Open | In Progress | Resolved | Closed]
  ├── reported_by / assigned_to
  └── created_at / resolved_at / resolution_notes

audit_logs (append-only, 1:N from data_models)
  ├── id (PK)
  ├── model_id (FK → data_models)
  ├── actor / action / detail
  └── created_at
```

---

## State Machines

### DataModel.status

```
Draft ──► Review ──► Approved ──► Active
  ▲          │                      │
  └──────────┘                      ▼
                                Deprecated ──► Archived
```

### ChangeRequest.status

```
Open ──► In Review ──► Approved ──► Implemented
             │
             ▼
          Rejected
```

### DataIssue.status

```
Open ──► In Progress ──► Resolved ──► Closed
                              │
                              ▼
                           Reopened (back to Open)
```

### ComplianceMapping.status

Auto-computed from controls:
```
Not Mapped ──► (after adding controls)
                    ├── all passed  → Compliant
                    ├── some passed → Partial
                    └── none passed → Gap
```
