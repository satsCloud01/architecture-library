---
name: "C1 – System Context"
project: "Enterprise AI Guardian"
project_slug: "ai-guardian"
project_url: "https://ai-guardian.satszone.link"
github: "https://github.com/satsCloud01/ai-governance-bank"
category: "ai-agents"
type: "c4-context"
icon: "🏦"
tags: [FastAPI, React, Recharts]
---

# C1 — System Context Diagram

> **C4 Level 1**: Shows Enterprise AI Guardian in relation to its users and external systems.
> Scope: The entire AI Governance platform as a single black box.

---

```mermaid
C4Context
  title System Context — Enterprise AI Guardian

  Person(mrm, "Model Risk Manager", "Validates AI models against SR 11-7. Reviews and approves model risk assessments.")
  Person(developer, "Model Developer / Data Scientist", "Registers new AI/ML/LLM models, submits for validation, manages versions.")
  Person(cro, "CRO / CISO", "Final approval authority. Reviews executive dashboard and compliance posture.")
  Person(auditor, "Internal / External Auditor", "Reviews audit trails, compliance evidence, and regulatory readiness reports.")
  Person(compliance, "Compliance Officer", "Monitors regulatory coverage, fairness metrics, and gap analysis.")

  System(guardian, "Enterprise AI Guardian", "End-to-end AI governance platform. Manages model registry, risk classification, lifecycle approvals, compliance mapping, performance monitoring, fairness analysis, and audit trails.")

  System_Ext(openai, "OpenAI API", "Hosts GPT-4o and GPT-4o-mini LLMs used by Aria, FraudNarrator, ComplianceGPT.")
  System_Ext(anthropic, "Anthropic API", "Hosts Claude 3.5 Sonnet and Claude Haiku LLMs used by LoanAdvisor AI, DocuSense, TradeSentinel AI.")
  System_Ext(creditbureau, "Credit Bureau (Experian)", "Provides real-time consumer credit data consumed by CreditScorer v3.2.")
  System_Ext(corebanking, "Core Banking System", "Source of transaction history, account data, and customer profiles.")
  System_Ext(regulatory, "Regulatory Bodies", "Fed Reserve (SR 11-7), OCC, EBA, FINRA — receive compliance reports and examination evidence.")
  System_Ext(fincen, "FinCEN / OFAC", "Provides sanctions lists and SAR filing guidance consumed by AML Sentinel and FraudNarrator.")

  Rel(mrm, guardian, "Validates models, reviews risk assessments, approves lifecycle transitions", "HTTPS / Browser")
  Rel(developer, guardian, "Registers models, submits versions, monitors performance", "HTTPS / Browser")
  Rel(cro, guardian, "Reviews executive dashboard, provides final approval", "HTTPS / Browser")
  Rel(auditor, guardian, "Accesses audit trail, compliance evidence, exam packages", "HTTPS / Browser")
  Rel(compliance, guardian, "Monitors compliance gaps, fairness alerts, regulation mapping", "HTTPS / Browser")

  Rel(guardian, openai, "Routes prompts for Aria, FraudNarrator, ComplianceGPT", "HTTPS / REST")
  Rel(guardian, anthropic, "Routes prompts for LoanAdvisor AI, DocuSense, TradeSentinel AI", "HTTPS / REST")
  Rel(guardian, creditbureau, "Ingests credit data for CreditScorer v3.2", "HTTPS / REST")
  Rel(guardian, corebanking, "Reads transaction and account data for ML model inputs", "Internal API")
  Rel(guardian, regulatory, "Exports compliance packages, audit evidence", "Secure File Transfer")
  Rel(guardian, fincen, "Syncs OFAC/SDN lists, validates SAR typologies", "HTTPS / REST")

  UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

---

## System Responsibilities

| Responsibility | Description |
|---|---|
| **Model Registry** | Single inventory of all AI/ML/LLM models with full metadata, versioning, and data lineage |
| **Risk Classification** | Automated Tier 1/2/3 risk scoring per SR 11-7 methodology |
| **Lifecycle Governance** | Role-gated workflow from Development → Validation → MRM Review → Approved → Production → Retired |
| **Compliance Mapping** | Model ↔ Regulation ↔ Control checklist with gap analysis and exam-ready reporting |
| **Performance Monitoring** | Time-series tracking of AUC/PSI/Gini (ML) and hallucination/relevance/cost (LLM) |
| **Fairness & Bias** | ECOA-mandated disparate impact analysis with breach detection across protected classes |
| **Audit Trail** | Immutable, timestamped log of every model action — regulator-ready |
| **Incident Management** | Track, assign, and resolve model failures, bias events, and security findings |

## Users & Roles

| Role | Primary Actions | Access Level |
|---|---|---|
| Model Developer | Register, version, submit for validation | Read/Write own models |
| Model Risk Manager | Validate models, approve lifecycle transitions | Read all, write lifecycle |
| CRO / CISO | Final approval, dashboard review | Read all, write approvals |
| Compliance Officer | Monitor compliance, manage controls | Read all, write compliance |
| Internal Auditor | Review audit trails, generate reports | Read-only |
| External Auditor / Examiner | Access exam package exports | Scoped read-only |

## Regulatory Boundary

Enterprise AI Guardian operates within the following regulatory perimeter:

```
┌─────────────────────────────────────────────────────────────────┐
│  US Jurisdiction          │  EU Jurisdiction    │  Global       │
│  ─────────────────────    │  ─────────────────  │  ──────────   │
│  SR 11-7 (Fed Reserve)    │  EU AI Act          │  Basel III/IV │
│  OCC 2011-12              │  DORA               │               │
│  ECOA / Reg B             │                     │               │
│  FCRA                     │                     │               │
│  BSA / AML (FinCEN)       │                     │               │
│  FINRA Rules              │                     │               │
└─────────────────────────────────────────────────────────────────┘
```
