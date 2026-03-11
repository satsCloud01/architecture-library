---
name: "Domain Model"
project: "SatsRecruitment Intelligence"
project_slug: "sats-recruitment"
project_url: "https://sats-recruitment.satszone.link"
github: "https://github.com/satsCloud01/sats-recruitment"
category: "productivity"
type: "domain-model"
icon: "🧠"
tags: [SQLAlchemy, Pydantic]
---

# SatsRecruitment Intelligence - Domain Model

## Entity Relationship Overview

```
Setting (standalone)

Skill (standalone, referenced by name in JSON columns)

Job ----< Candidate
 |            |
 |            +----< Interview (via candidate_id + job_id)
 |            +----< Engagement (via candidate_id)
 |            +----< ScheduledInterview (via candidate_id + job_id)
 |
 +----< ScheduledInterview (via job_id)

Employee ----< Project (via assigned_employees JSON)
    |
    +----< SuccessionPlan (as current_holder or successor)
    +----< MarketplaceListing (as posted_by or applicant)

Role ----< MobilityListing (via role_id)
  |
  +----< SuccessionPlan (via role_id)
  +---- Role (self-ref via career_paths JSON)

OutreachTemplate ----< Engagement (via template_id)
```

---

## Entities

### Setting
Key-value configuration store. Used for app-level preferences (NOT for API keys -- those are client-side only).

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| key | String(100) | Unique, not null | Setting identifier |
| value | Text | Default "" | Setting value |

---

### Skill
Taxonomy of professional skills with market demand tracking. Seeded with 50 skills across 15 categories.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| name | String(200) | Unique, not null | Skill name (e.g., "Python", "Leadership") |
| category | String(100) | Default "General" | Category (Programming, Cloud, AI/ML, Soft Skills, etc.) |
| description | Text | Default "" | Skill description |
| demand_trend | String(20) | Default "stable" | Market demand: rising, stable, declining |
| created_at | DateTime | UTC default | Creation timestamp |

**Skill Categories** (from seed data): Programming, Frontend, Backend, Data, Database, Cloud, DevOps, Infrastructure, AI/ML, API, Architecture, Security, Design, Business, Process, Soft Skills, Systems, Tools

---

### Job
A job posting/requisition that candidates can be matched against.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| title | String(300) | Not null | Job title |
| department | String(200) | Default "" | Department (Engineering, AI/ML, Data, etc.) |
| location | String(200) | Default "" | Work location or "Remote" |
| job_type | String(50) | Default "full-time" | full-time, part-time, contract |
| description | Text | Default "" | Full job description |
| requirements | Text | Default "" | Experience/qualification requirements |
| required_skills | JSON | Default [] | List of skill name strings |
| salary_min | Integer | Default 0 | Minimum salary (USD) |
| salary_max | Integer | Default 0 | Maximum salary (USD) |
| status | String(50) | Default "open" | open, closed, draft |
| posted_at | DateTime | UTC default | When posted |
| created_at | DateTime | UTC default | Creation timestamp |

**Relationships**: One-to-many with Candidate (job_id FK)

---

### Candidate
An external applicant being evaluated for a job position.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| name | String(300) | Not null | Full name |
| email | String(300) | Default "" | Email address |
| phone | String(50) | Default "" | Phone number |
| location | String(200) | Default "" | Geographic location |
| resume_text | Text | Default "" | Raw resume content for AI parsing |
| skills | JSON | Default [] | List of skill name strings |
| experience_years | Float | Default 0 | Years of experience |
| education | Text | Default "" | Education background |
| current_role | String(300) | Default "" | Current job title and company |
| match_score | Float | Default 0 | AI-computed job match score (0-100) |
| status | String(50) | Default "new" | new, screening, interview, offer, hired, rejected |
| notes | Text | Default "" | Recruiter notes |
| job_id | Integer | FK -> jobs, nullable | Associated job posting |
| anonymized | Boolean | Default False | If true, PII is masked in API responses |
| created_at | DateTime | UTC default | Creation timestamp |

**Relationships**: Many-to-one with Job

---

### Employee
An internal employee within the organization, tracked for talent management, career development, and workforce planning.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| name | String(300) | Not null | Full name |
| email | String(300) | Default "" | Corporate email |
| department | String(200) | Default "" | Department |
| current_role | String(300) | Default "" | Current role title |
| skills | JSON | Default [] | List of skill name strings |
| experience_years | Float | Default 0 | Total years of experience |
| career_goals | Text | Default "" | Career aspirations (free text) |
| performance_rating | Float | Default 3.0 | Rating on 1-5 scale |
| mentoring_interests | JSON | Default [] | Topics employee wants mentoring in |
| courses_completed | JSON | Default [] | Training courses completed |
| joined_at | DateTime | UTC default | Date joined organization |
| created_at | DateTime | UTC default | Creation timestamp |

**Referenced by**: Project (assigned_employees JSON), SuccessionPlan (current_holder_id FK, successor_ids JSON), MarketplaceListing (posted_by_id FK, applicant_ids JSON)

---

### Role
An organizational role definition used for career pathing, succession planning, and mobility.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| title | String(300) | Not null | Role title |
| department | String(200) | Default "" | Department |
| level | String(50) | Default "mid" | junior, mid, senior, lead, director, vp |
| description | Text | Default "" | Role description |
| required_skills | JSON | Default [] | Skills required for this role |
| career_paths | JSON | Default [] | List of role IDs that precede this role |
| market_demand | String(20) | Default "stable" | rising, stable, declining |
| created_at | DateTime | UTC default | Creation timestamp |

**Relationships**: One-to-many with MobilityListing, SuccessionPlan

---

### Project
An organizational project with team assignments and utilization tracking.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| name | String(300) | Not null | Project name |
| description | Text | Default "" | Project description |
| department | String(200) | Default "" | Owning department |
| required_skills | JSON | Default [] | Skills needed for the project |
| team_size | Integer | Default 1 | Target team size |
| assigned_employees | JSON | Default [] | List of employee IDs assigned |
| status | String(50) | Default "planning" | planning, active, completed |
| start_date | String(20) | Default "" | ISO date string |
| end_date | String(20) | Default "" | ISO date string |
| utilization | Float | Default 0 | Utilization percentage (0-100) |
| created_at | DateTime | UTC default | Creation timestamp |

---

### Interview
An AI-generated interview session with questions, candidate answers, and AI-evaluated scores.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| candidate_id | Integer | FK -> candidates, not null | Candidate being interviewed |
| job_id | Integer | FK -> jobs, not null | Job the interview is for |
| questions | JSON | Default [] | Array of question objects (question, type, difficulty, criteria, hints) |
| answers | JSON | Default [] | Array of candidate answer strings |
| scores | JSON | Default [] | Array of AI evaluation objects (score, strengths, improvements, feedback) |
| overall_score | Float | Default 0 | Computed average of individual scores |
| feedback | Text | Default "" | Overall interview feedback |
| status | String(50) | Default "pending" | pending, in_progress, completed |
| created_at | DateTime | UTC default | Creation timestamp |

---

### MobilityListing
An internal mobility opportunity for employee transfers, reskilling, or redeployment.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| role_id | Integer | FK -> roles, nullable | Associated role definition |
| title | String(300) | Not null | Listing title |
| department | String(200) | Default "" | Target department |
| description | Text | Default "" | Opportunity description |
| required_skills | JSON | Default [] | Required skills |
| listing_type | String(50) | Default "internal" | internal, redeployment, reskill |
| status | String(50) | Default "open" | open, closed |
| created_at | DateTime | UTC default | Creation timestamp |

---

### OutreachTemplate
Reusable email/message templates for candidate outreach.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| name | String(300) | Not null | Template name |
| subject | String(500) | Default "" | Email subject line (supports {placeholders}) |
| body | Text | Default "" | Message body (supports {candidate_name}, {job_title}, etc.) |
| template_type | String(50) | Default "initial" | initial, follow_up, rejection, offer |
| created_at | DateTime | UTC default | Creation timestamp |

---

### Engagement
A record of outreach activity to a candidate via a specific channel.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| candidate_id | Integer | FK -> candidates, not null | Target candidate |
| template_id | Integer | FK -> outreach_templates, nullable | Template used (if any) |
| channel | String(50) | Default "email" | email, linkedin, phone |
| status | String(50) | Default "sent" | draft, sent, opened, replied, bounced |
| message | Text | Default "" | Actual message sent or notes |
| sent_at | DateTime | UTC default | When the engagement was sent |
| created_at | DateTime | UTC default | Creation timestamp |

---

### ScheduledInterview
A calendar entry for an upcoming interview between a candidate and an interviewer.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| candidate_id | Integer | FK -> candidates, not null | Candidate |
| job_id | Integer | FK -> jobs, not null | Job position |
| interviewer_name | String(300) | Default "" | Name of interviewer |
| interview_type | String(50) | Default "video" | video, phone, onsite, ai |
| scheduled_date | String(20) | Default "" | ISO date (YYYY-MM-DD) |
| scheduled_time | String(10) | Default "" | Time (HH:MM) |
| duration_minutes | Integer | Default 60 | Interview duration |
| status | String(50) | Default "scheduled" | scheduled, confirmed, completed, cancelled, no_show |
| notes | Text | Default "" | Interview notes |
| meeting_link | String(500) | Default "" | Video meeting URL |
| created_at | DateTime | UTC default | Creation timestamp |

---

### SuccessionPlan
A succession plan linking a critical role to potential successor employees.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| role_id | Integer | FK -> roles, not null | The role being planned for |
| current_holder_id | Integer | FK -> employees, nullable | Current person in the role |
| successor_ids | JSON | Default [] | Array of {employee_id, readiness, timeline_months} |
| risk_level | String(20) | Default "medium" | low, medium, high, critical |
| notes | Text | Default "" | Planning notes |
| created_at | DateTime | UTC default | Creation timestamp |

**Successor readiness values**: ready, nearly_ready, developing

---

### MarketplaceListing
An internal talent marketplace listing for gigs, mentorships, projects, or learning opportunities.

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Integer | PK, auto | Primary key |
| title | String(300) | Not null | Listing title |
| listing_type | String(50) | Default "gig" | gig, mentorship, project, learning |
| department | String(200) | Default "" | Department offering the opportunity |
| description | Text | Default "" | Detailed description |
| required_skills | JSON | Default [] | Skills needed |
| duration | String(100) | Default "" | Time duration (e.g., "2 weeks", "3 months") |
| time_commitment | String(50) | Default "part-time" | full-time, part-time, flexible |
| posted_by_id | Integer | FK -> employees, nullable | Employee who posted the listing |
| applicant_ids | JSON | Default [] | List of employee IDs who applied |
| status | String(50) | Default "open" | open, closed |
| created_at | DateTime | UTC default | Creation timestamp |
