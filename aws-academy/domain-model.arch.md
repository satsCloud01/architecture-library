---
name: "Domain Model"
project: "AWS Academy"
project_slug: "aws-academy"
category: "learning"
type: "domain-model"
icon: "☁️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'aws']
---

# Domain Model

## Entities

### 1. Lesson (42 instances)
Primary learning content unit. Each lesson belongs to exactly one category.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier (1-42) |
| category | string | One of 14 categories |
| title | string | Display title |
| type | string | `intro`, `concept` |
| description | string | Main content paragraph |
| keyPoints | string[] | Bullet-point takeaways |
| comparison | object? | Table with headers + rows |
| diagram | object? | Flow/architecture diagram (nodes + edges) |
| code | string? | Code/config example |
| codeLanguage | string? | Language hint for display |

### 2. Category (14 instances)
Logical grouping of lessons. Derived at runtime, not stored separately.

| Category | Coverage |
|----------|----------|
| Cloud Foundations | AWS global model, shared responsibility, account structure |
| Global Infrastructure | Regions, AZs, edge locations, Local Zones |
| IAM and Identity | IAM policies, roles, federation, SSO |
| Governance and Org Design | Organizations, SCPs, Control Tower |
| Networking | VPC, subnets, Transit Gateway, Route 53 |
| Compute and Containers | EC2, Lambda, ECS, EKS, Fargate |
| Storage | S3, EBS, EFS, Glacier, Storage Gateway |
| Data, Analytics, and AI | RDS, DynamoDB, Redshift, SageMaker |
| Integration and Eventing | SQS, SNS, EventBridge, Step Functions |
| Security | KMS, Secrets Manager, GuardDuty, Security Hub |
| Reliability and DR | Multi-AZ, backup, pilot light, warm standby |
| Operations and Observability | CloudWatch, X-Ray, CloudTrail, Config |
| DevOps and IaC | CloudFormation, CDK, CodePipeline, CI/CD |
| Advanced Architectures | Multi-account, hybrid, serverless patterns |

### 3. QuizQuestion (15 instances)
Multiple-choice assessment with server-side scoring.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| category | string | Maps to lesson category |
| question | string | Question text |
| options | string[4] | Four answer choices |
| correct | int | Index of correct answer (server-only) |
| explanation | string | Why the answer is correct (server-only) |

### 4. Paper (14 instances)
Curated AWS whitepapers and reference documents.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | Whitepaper/guide title |
| authors | string | Author names |
| year | int | Publication year |
| category | string | Architecture, Security, Operations, etc. |
| url | string | Link to original document |
| summary | string | One-paragraph summary |
| key_takeaways | string[] | 5-6 most important points |
| related_topics | string[] | Connected concepts |

## Entity Relationships

```
Category ──1:N──▶ Lesson
Category ──1:N──▶ QuizQuestion
Category ──1:N──▶ Paper (via paper.category)
```

All entities are **read-only** at runtime. No CRUD operations. No user-generated content.

## Data Storage

All data is defined as Python lists/dicts in source code:
- `backend/src/awsacademy/data/lessons.py` — LESSONS, QUIZ_QUESTIONS
- `backend/src/awsacademy/routers/papers.py` — PAPERS (inline)

No database. No migrations. No ORM.

