---
name: "API Specification"
project: "SatsRecruitment Intelligence"
project_slug: "sats-recruitment"
project_url: "https://sats-recruitment.satszone.link"
github: "https://github.com/satsCloud01/sats-recruitment"
category: "productivity"
type: "api-spec"
icon: "🧠"
tags: [FastAPI, OpenAPI]
---

# SatsRecruitment Intelligence - API Specification

Base URL: `/api`

All request/response bodies are JSON. Errors return `{"detail": "<message>"}` with appropriate HTTP status.

---

## Health

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Health check |

**Response**: `{"status": "healthy", "app": "SatsRecruitment Intelligence", "version": "1.0.0"}`

---

## Jobs

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/jobs` | List jobs |
| GET | `/api/jobs/{job_id}` | Get job with candidates |
| POST | `/api/jobs` | Create job |
| PUT | `/api/jobs/{job_id}` | Update job |
| DELETE | `/api/jobs/{job_id}` | Delete job |

### GET /api/jobs
**Query params**: `status` (optional), `department` (optional)
**Response**: Array of job objects with `candidate_count` included.

### GET /api/jobs/{job_id}
**Response**: Job object with nested `candidates` array (id, name, status, match_score, skills, experience_years).

### POST /api/jobs
**Request body**:
```json
{
  "title": "string (required)",
  "department": "string",
  "location": "string",
  "job_type": "full-time | part-time | contract",
  "description": "string",
  "requirements": "string",
  "required_skills": ["string"],
  "salary_min": 0,
  "salary_max": 0,
  "status": "open | closed | draft"
}
```
**Response**: `{"id": 1, "title": "...", "status": "created"}`

### PUT /api/jobs/{job_id}
**Request body**: Same fields as POST, all optional. Only provided fields are updated.
**Response**: `{"id": 1, "title": "...", "status": "updated"}`

### DELETE /api/jobs/{job_id}
**Response**: `{"status": "deleted"}`

---

## Candidates

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/candidates` | List candidates |
| GET | `/api/candidates/{id}` | Get candidate detail |
| POST | `/api/candidates` | Create candidate |
| PUT | `/api/candidates/{id}` | Update candidate |
| DELETE | `/api/candidates/{id}` | Delete candidate |
| POST | `/api/candidates/{id}/anonymize` | Toggle PII anonymization |

### GET /api/candidates
**Query params**: `status` (optional), `job_id` (optional)
**Response**: Array of candidate objects. If `anonymized=true`, name becomes "Candidate #N", email/phone become "***".

### GET /api/candidates/{id}
**Response**: Full candidate object. If associated with a job, includes `job_title` and `job_department`.

### POST /api/candidates
**Request body**:
```json
{
  "name": "string (required)",
  "email": "string",
  "phone": "string",
  "location": "string",
  "resume_text": "string",
  "skills": ["string"],
  "experience_years": 0,
  "education": "string",
  "current_role": "string",
  "status": "new | screening | interview | offer | hired | rejected",
  "job_id": null,
  "notes": "string"
}
```

### POST /api/candidates/{id}/anonymize
Toggles the `anonymized` boolean.
**Response**: `{"id": 1, "anonymized": true}`

---

## Employees

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/employees` | List employees |
| GET | `/api/employees/{id}` | Get employee detail |
| POST | `/api/employees` | Create employee |
| PUT | `/api/employees/{id}` | Update employee |
| DELETE | `/api/employees/{id}` | Delete employee |

### GET /api/employees
**Query params**: `department` (optional)

### POST /api/employees
**Request body**:
```json
{
  "name": "string (required)",
  "email": "string",
  "department": "string",
  "current_role": "string",
  "skills": ["string"],
  "experience_years": 0,
  "career_goals": "string",
  "performance_rating": 3.0,
  "mentoring_interests": ["string"],
  "courses_completed": ["string"]
}
```

---

## Roles

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/roles` | List roles |
| GET | `/api/roles/{id}` | Get role detail |
| POST | `/api/roles` | Create role |
| PUT | `/api/roles/{id}` | Update role |
| DELETE | `/api/roles/{id}` | Delete role |

### GET /api/roles
**Query params**: `department` (optional), `level` (optional: junior, mid, senior, lead, director, vp)

### POST /api/roles
**Request body**:
```json
{
  "title": "string (required)",
  "department": "string",
  "level": "junior | mid | senior | lead | director | vp",
  "description": "string",
  "required_skills": ["string"],
  "career_paths": [1, 2],
  "market_demand": "rising | stable | declining"
}
```

---

## Skills

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/skills` | List skills |
| GET | `/api/skills/categories` | List unique categories |
| POST | `/api/skills` | Create skill |
| DELETE | `/api/skills/{id}` | Delete skill |

### GET /api/skills
**Query params**: `category` (optional), `trend` (optional)
**Response**: Array of `{id, name, category, description, demand_trend}`

### GET /api/skills/categories
**Response**: Array of category strings, e.g., `["Programming", "Cloud", "AI/ML", ...]`

---

## Projects

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/projects` | List projects |
| GET | `/api/projects/{id}` | Get project with assigned team details |
| POST | `/api/projects` | Create project |
| PUT | `/api/projects/{id}` | Update project |
| DELETE | `/api/projects/{id}` | Delete project |

### GET /api/projects
**Query params**: `status` (optional), `department` (optional)

### GET /api/projects/{id}
**Response**: Includes `assigned_team` array with full employee details (id, name, role, skills).

### POST /api/projects
**Request body**:
```json
{
  "name": "string (required)",
  "description": "string",
  "department": "string",
  "required_skills": ["string"],
  "team_size": 1,
  "assigned_employees": [1, 5, 8],
  "status": "planning | active | completed",
  "start_date": "2026-01-15",
  "end_date": "2026-06-30",
  "utilization": 0
}
```

---

## Interviews

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/interviews` | List interviews |
| GET | `/api/interviews/{id}` | Get full interview (questions, answers, scores) |
| POST | `/api/interviews` | Create interview |
| PUT | `/api/interviews/{id}/answer` | Submit answer to a question |
| PUT | `/api/interviews/{id}/complete` | Mark interview as completed, compute overall score |

### GET /api/interviews
**Query params**: `status` (optional)

### PUT /api/interviews/{id}/answer
**Request body**: `{"question_index": 0, "answer": "My response..."}`
**Response**: `{"id": 1, "answers_count": 3, "status": "answer_recorded"}`

### PUT /api/interviews/{id}/complete
Calculates `overall_score` as average of all score values, sets status to "completed".

---

## Mobility

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/mobility` | List mobility listings |
| GET | `/api/mobility/{id}` | Get listing detail |
| POST | `/api/mobility` | Create listing |
| DELETE | `/api/mobility/{id}` | Delete listing |

### GET /api/mobility
**Query params**: `listing_type` (optional: internal, redeployment, reskill), `status` (optional)

---

## Analytics

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/analytics/dashboard` | Dashboard summary stats |
| GET | `/api/analytics/skills-heatmap` | Skill supply/demand heatmap |
| GET | `/api/analytics/hiring-funnel` | Candidate pipeline funnel |
| GET | `/api/analytics/department-skills` | Skills per department |
| GET | `/api/analytics/utilization` | Resource utilization |
| GET | `/api/analytics/diversity-metrics` | Experience distribution and location diversity |
| GET | `/api/analytics/hiring-metrics` | Hiring KPIs (time-to-hire, cost, quality, conversion) |

### GET /api/analytics/dashboard
**Response**:
```json
{
  "jobs_total": 10,
  "jobs_open": 10,
  "candidates_total": 20,
  "employees_total": 15,
  "projects_active": 6,
  "mobility_open": 5,
  "pipeline": {"new": 12, "screening": 3, "interview": 2, ...},
  "departments": {"Engineering": 7, "Data": 2, ...},
  "avg_performance_rating": 4.13
}
```

### GET /api/analytics/skills-heatmap
**Response**: Dict keyed by skill name, values have `category`, `demand_trend`, `employee_count`, `candidate_count`, `demand_count`, `supply`, `gap`.

### GET /api/analytics/hiring-metrics
**Response**: Includes `avg_time_to_hire_days`, `avg_cost_per_hire`, `quality_of_hire_score`, `offer_acceptance_rate`, `source_effectiveness`, `stage_conversion_rates`.

---

## Settings

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/settings` | Get all settings (excludes anthropic_api_key) |
| PUT | `/api/settings` | Create/update a setting |
| GET | `/api/settings/api-key-status` | Check if API key is present in request header |
| POST | `/api/settings/test-connection` | Test Anthropic API key validity |

### PUT /api/settings
**Request body**: `{"key": "string", "value": "string"}`
Rejects `key="anthropic_api_key"` with `{"status": "rejected", "detail": "API key must be stored client-side only."}`.

### GET /api/settings/api-key-status
**Headers**: `X-API-Key: sk-ant-...` (optional)
**Response**: `{"configured": true, "has_api_key": true}` or `{"configured": false, "has_api_key": false}`

### POST /api/settings/test-connection
**Headers**: `X-API-Key: sk-ant-...` (required)
Makes a real API call to Claude to verify the key works.
**Response**: `{"valid": true}` or `{"valid": false, "error": "..."}`

---

## AI (Requires X-API-Key header for real AI; falls back to mock without it)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/ai/parse-resume` | Parse resume text into structured data |
| POST | `/api/ai/match` | Match candidate to job (updates match_score) |
| POST | `/api/ai/career-paths` | Suggest career paths for employee |
| POST | `/api/ai/generate-jd` | Generate job description |
| POST | `/api/ai/generate-interview` | Generate interview questions (creates Interview record) |
| POST | `/api/ai/evaluate-answer` | Evaluate a single interview answer |
| POST | `/api/ai/skill-gap` | Analyze skill gaps between employee and target role |
| POST | `/api/ai/workforce-trends` | Forecast workforce trends for a department |
| POST | `/api/ai/generate-outreach` | Generate personalized outreach message |

### POST /api/ai/parse-resume
**Request**: `{"resume_text": "Full resume text..."}`
**Response**: `{name, email, phone, location, skills[], experience_years, education, current_role, summary}`

### POST /api/ai/match
**Request**: `{"candidate_id": 1, "job_id": 1}`
**Side effect**: Updates `candidate.match_score` in database.
**Response**: `{overall_score, skill_match_score, experience_match_score, culture_fit_score, matched_skills[], missing_skills[], strengths[], concerns[], recommendation, summary}`

### POST /api/ai/career-paths
**Request**: `{"employee_id": 1}`
**Response**: `{recommended_paths[], skills_to_develop[], recommended_courses[], mentorship_areas[], summary}`

### POST /api/ai/generate-jd
**Request**: `{"title": "...", "department": "...", "required_skills": [...], "level": "mid"}`
**Response**: `{title, description, requirements, responsibilities[], qualifications{required[], preferred[]}, salary_range{min, max}, benefits[]}`

### POST /api/ai/generate-interview
**Request**: `{"candidate_id": 1, "job_id": 1}`
**Side effect**: Creates Interview record in database.
**Response**: `{interview_id, questions[]}`
Each question: `{question, type, difficulty, evaluation_criteria, ideal_answer_hints}`

### POST /api/ai/evaluate-answer
**Request**: `{"interview_id": 1, "question_index": 0, "answer": "..."}`
**Side effect**: Updates Interview answers and scores arrays.
**Response**: `{score (1-10), strengths[], areas_for_improvement[], follow_up_question, feedback}`

### POST /api/ai/skill-gap
**Request**: `{"employee_id": 1, "target_role_id": 3}`
**Response**: `{gap_score (0-100), matched_skills[], missing_skills[], transferable_skills[], development_plan[], readiness_assessment, summary}`

### POST /api/ai/workforce-trends
**Request**: `{"department": "Engineering"}`
**Response**: `{emerging_skills[], declining_skills[], hiring_recommendations[], reskilling_opportunities[], market_insights[], risk_areas[], summary}`

### POST /api/ai/generate-outreach
**Request**: `{"candidate_id": 1, "job_id": 1, "template_type": "initial | follow_up"}`
**Response**: `{subject, body, candidate_name, job_title, template_type}`

---

## CRM

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/crm/templates` | List outreach templates |
| POST | `/api/crm/templates` | Create template |
| GET | `/api/crm/engagements` | List engagements |
| POST | `/api/crm/engagements` | Create engagement |
| PUT | `/api/crm/engagements/{id}` | Update engagement status |
| GET | `/api/crm/stats` | Engagement statistics |

### POST /api/crm/templates
**Request**:
```json
{
  "name": "string (required)",
  "subject": "string",
  "body": "string",
  "template_type": "initial | follow_up | rejection | offer"
}
```

### GET /api/crm/engagements
**Query params**: `candidate_id` (optional)
**Response**: Includes `candidate_name` resolved from FK.

### POST /api/crm/engagements
**Request**:
```json
{
  "candidate_id": 1,
  "template_id": null,
  "channel": "email | linkedin | phone",
  "status": "draft | sent | opened | replied | bounced",
  "message": "string"
}
```

### GET /api/crm/stats
**Response**: `{total, sent, opened, replied, bounced, open_rate, reply_rate, bounce_rate}`

---

## Scheduling

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/scheduling` | List scheduled interviews |
| POST | `/api/scheduling` | Schedule an interview |
| PUT | `/api/scheduling/{id}` | Update scheduled interview |
| DELETE | `/api/scheduling/{id}` | Cancel interview (sets status to "cancelled") |

### GET /api/scheduling
**Query params**: `candidate_id` (optional), `status` (optional)
**Response**: Includes resolved `candidate_name` and `job_title`.

### POST /api/scheduling
**Request**:
```json
{
  "candidate_id": 1,
  "job_id": 1,
  "interviewer_name": "string",
  "interview_type": "video | phone | onsite | ai",
  "scheduled_date": "2026-03-12",
  "scheduled_time": "10:00",
  "duration_minutes": 60,
  "notes": "string",
  "meeting_link": "string"
}
```

### DELETE /api/scheduling/{id}
Does NOT delete the record; sets `status` to "cancelled".
**Response**: `{"message": "Interview cancelled", "id": 1}`

---

## Succession

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/succession` | List succession plans |
| GET | `/api/succession/{id}` | Get plan with successor employee details |
| POST | `/api/succession` | Create plan |
| PUT | `/api/succession/{id}` | Update plan |

### GET /api/succession/{id}
**Response**: Includes resolved `role_title`, `current_holder` object, and `successors` array with full employee details and readiness/timeline.

### POST /api/succession
**Request**:
```json
{
  "role_id": 13,
  "current_holder_id": null,
  "successor_ids": [
    {"employee_id": 5, "readiness": "ready", "timeline_months": 6}
  ],
  "risk_level": "low | medium | high | critical",
  "notes": "string"
}
```

---

## Marketplace

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/marketplace` | List marketplace listings |
| GET | `/api/marketplace/{id}` | Get listing with applicant details |
| POST | `/api/marketplace` | Create listing |
| PUT | `/api/marketplace/{id}` | Update listing |
| POST | `/api/marketplace/{id}/apply` | Employee applies to listing |

### GET /api/marketplace
**Query params**: `listing_type` (optional: gig, mentorship, project, learning)

### GET /api/marketplace/{id}
**Response**: Includes `posted_by_name` and `applicants` array with employee details.

### POST /api/marketplace
**Request**:
```json
{
  "title": "string (required)",
  "listing_type": "gig | mentorship | project | learning",
  "department": "string",
  "description": "string",
  "required_skills": ["string"],
  "duration": "string",
  "time_commitment": "full-time | part-time | flexible",
  "posted_by_id": null
}
```

### POST /api/marketplace/{id}/apply
**Request**: `{"employee_id": 1}`
**Validation**: Returns 400 if employee already applied.
**Response**: `{"message": "Application submitted", "listing_id": 1, "employee_id": 1, "applicant_count": 3}`
