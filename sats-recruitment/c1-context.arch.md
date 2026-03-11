---
name: "C1 – System Context"
project: "SatsRecruitment Intelligence"
project_slug: "sats-recruitment"
project_url: "https://sats-recruitment.satszone.link"
github: "https://github.com/satsCloud01/sats-recruitment"
category: "productivity"
type: "c4-context"
icon: "🧠"
tags: [FastAPI, React, Claude API, Recharts]
---

# SatsRecruitment Intelligence - Architecture Documentation

## C1: System Context

```
                         +---------------------------+
                         |    Recruiters / HR Staff   |
                         |       (Browser Users)      |
                         +------------+--------------+
                                      |
                                      | HTTPS
                                      v
                         +---------------------------+
                         |   SatsRecruitment          |
                         |   Intelligence             |
                         |                            |
                         |  AI-powered recruitment,   |
                         |  talent management, and    |
                         |  workforce planning        |
                         |  platform                  |
                         +-----+---------------+-----+
                               |               |
                               | HTTPS         | SQLite
                               v               v
                    +----------------+   +------------+
                    | Anthropic      |   | Local File |
                    | Claude API     |   | System     |
                    | (claude-haiku- |   | (DB file)  |
                    |  4-5)          |   +------------+
                    +----------------+
```

**Users**: Recruiters, HR managers, hiring managers, and talent management professionals who use the web interface to manage the full recruitment lifecycle -- from job postings and candidate tracking through AI-assisted interviews and workforce planning.

**SatsRecruitment Intelligence**: A full-stack web application providing AI-enhanced recruitment, talent management, internal mobility, and workforce analytics.

**Anthropic Claude API**: External AI service (Claude Haiku 4.5) used for resume parsing, candidate-job matching, career path suggestions, job description generation, interview question generation, answer evaluation, skill gap analysis, workforce trend forecasting, and outreach message generation. All AI features gracefully degrade to mock/heuristic fallbacks when no API key is configured.

---

## C2: Container Diagram

```
+-------------------------------------------------------------------+
|                     SatsRecruitment Intelligence                    |
|                                                                     |
|  +---------------------+        +-----------------------------+     |
|  |  Frontend            |        |  Backend                    |     |
|  |  (React 19 + Vite 7) |  /api  |  (FastAPI + Python 3.12)   |     |
|  |                       +------->                             |     |
|  |  - React Router v7   |        |  - 15 API routers          |     |
|  |  - Tailwind CSS v4   |        |  - SQLAlchemy 2.0 async    |     |
|  |  - Recharts 3        |        |  - Pydantic v2 schemas     |     |
|  |  - Lucide icons      |        |  - AI service module       |     |
|  |  - 15 pages + Landing |        |  - Auto-seed on startup    |     |
|  |                       |        |                             |     |
|  |  Port 5173 (dev)     |        |  Port 8000 (uvicorn)       |     |
|  +---------------------+        +----------+--+---------------+     |
|                                             |  |                     |
|                                             |  |                     |
+---------------------------------------------|--|---------------------+
                                              |  |
                              HTTPS (optional) |  | File I/O
                                              v  v
                                  +-----------+  +-----------------+
                                  | Anthropic |  | SQLite Database |
                                  | Claude    |  | recruitment.db  |
                                  | Haiku 4.5 |  +-----------------+
                                  +-----------+
```

### Frontend Container
- **Technology**: React 19.2, Vite 7.3, Tailwind CSS 4.2, Recharts 3.8, React Router 7.13, Lucide React
- **Responsibilities**: Renders all UI pages, manages client-side state, stores Anthropic API key in localStorage, proxies API calls to backend via Vite dev server
- **Pages** (15 + Landing): Dashboard, Jobs, Candidates, CandidateDetail, CandidateCRM, Scheduling, TalentMarketplace, TalentManagement, TalentDesign, ResourceManagement, WorkforceExchange, AIInterviewer, Analytics, GapAnalysis, Settings
- **Components**: Layout (sidebar navigation with 14 nav items, collapsible, mobile-responsive), GuidedTour (onboarding walkthrough)

### Backend Container
- **Technology**: FastAPI 0.115, Python 3.12, SQLAlchemy 2.0 (async), aiosqlite, Pydantic 2.10, Anthropic SDK 0.42
- **Responsibilities**: REST API serving, database operations, AI service orchestration, auto-seeding on first run
- **Startup lifecycle**: `init_db()` (creates tables) then `seed_database()` (populates if empty)
- **CORS**: Allows all origins (`*`)

### Database
- **Technology**: SQLite via aiosqlite (async driver)
- **File**: `backend/recruitment.db` (auto-created on first run)
- **Tables**: 14 tables corresponding to 14 SQLAlchemy models
- **Auto-seed**: 50 skills, 10 jobs, 20 candidates, 15 employees, 13 roles, 8 projects, 5 mobility listings, 4 outreach templates, 8 engagements, 5 scheduled interviews, 4 succession plans, 6 marketplace listings

### Anthropic Claude API
- **Model**: `claude-haiku-4-5-20251001`
- **Integration**: Synchronous calls via `anthropic.Anthropic` client
- **Auth**: API key passed via `X-API-Key` HTTP header from frontend; never stored server-side
- **Fallback**: All 8 AI functions have corresponding mock implementations

---

## C3: Component Diagram

### Backend Components

```
+------------------------------------------------------------------+
|                        FastAPI Application                         |
|                        (recruitment.main)                          |
|                                                                    |
|  +------------------------------------------------------------+  |
|  |                     API Routers (15)                         |  |
|  |                                                              |  |
|  |  jobs       candidates   employees   roles     skills       |  |
|  |  projects   interviews   mobility    analytics  settings    |  |
|  |  ai         crm          scheduling  succession marketplace |  |
|  +------+-----+-----------+------+-----+-----------+-----------+  |
|         |     |           |      |     |           |              |
|         v     v           v      v     v           v              |
|  +-------------+  +-------------+  +---------------------------+  |
|  | AI Service  |  | Database    |  | SQLAlchemy Models (14)    |  |
|  | Module      |  | Module      |  |                           |  |
|  |             |  |             |  | Setting, Skill, Job,      |  |
|  | 8 AI funcs  |  | engine      |  | Candidate, Employee,     |  |
|  | 3 mock funcs|  | async_session|  | Role, Project, Interview,|  |
|  | _call_claude|  | get_db()    |  | MobilityListing,         |  |
|  |             |  | init_db()   |  | OutreachTemplate,        |  |
|  +------+------+  +-------------+  | Engagement,              |  |
|         |                           | ScheduledInterview,      |  |
|         v                           | SuccessionPlan,          |  |
|  +-------------+                    | MarketplaceListing       |  |
|  | Anthropic   |                    +---------------------------+  |
|  | SDK         |                                                   |
|  +-------------+                    +---------------------------+  |
|                                     | Seed Module              |  |
|                                     | (recruitment.seed)       |  |
|                                     | Auto-populates DB        |  |
|                                     +---------------------------+  |
+--------------------------------------------------------------------+
```

#### Router Responsibilities

| Router | Prefix | Purpose |
|--------|--------|---------|
| jobs | `/api/jobs` | CRUD for job postings, includes candidate counts |
| candidates | `/api/candidates` | CRUD for candidates, anonymization toggle |
| employees | `/api/employees` | CRUD for internal employees |
| roles | `/api/roles` | CRUD for organizational roles with career paths |
| skills | `/api/skills` | CRUD for skills taxonomy, category listing |
| projects | `/api/projects` | CRUD for projects with employee assignments |
| interviews | `/api/interviews` | AI interview sessions (questions, answers, scores) |
| mobility | `/api/mobility` | Internal mobility/reskill/redeployment listings |
| analytics | `/api/analytics` | Dashboard stats, heatmaps, funnels, utilization, diversity, hiring metrics |
| settings | `/api/settings` | Key-value settings, API key status check, connection test |
| ai | `/api/ai` | All AI-powered operations (8 endpoints) |
| crm | `/api/crm` | Outreach templates, engagement tracking, CRM stats |
| scheduling | `/api/scheduling` | Interview scheduling, calendar management |
| succession | `/api/succession` | Succession plans with risk levels and successor readiness |
| marketplace | `/api/marketplace` | Internal talent marketplace (gigs, mentorship, projects, learning) |

#### AI Service Functions

| Function | Purpose | Mock Fallback |
|----------|---------|---------------|
| `parse_resume` | Extract structured data from resume text | `mock_parse_resume` |
| `match_candidate_to_job` | Score candidate-job fit (0-100) | `mock_match_candidate` |
| `suggest_career_paths` | Recommend career trajectories | `mock_career_paths` |
| `generate_job_description` | Create inclusive JD from parameters | Inline mock |
| `generate_interview_questions` | Tailored 8-question interview set | `mock_interview_questions` |
| `conduct_ai_interview` | Evaluate interview answer (1-10) | Inline mock |
| `analyze_skill_gaps` | Gap analysis with development plan | Inline mock |
| `forecast_workforce_trends` | Department workforce forecasting | Inline mock |

### Frontend Components

```
+-------------------------------------------------------------------+
|                        React Application                           |
|                        (App.jsx)                                   |
|                                                                    |
|  +-------------------+  +--------------------------------------+  |
|  | Landing Page      |  | Layout (authenticated pages)         |  |
|  | (standalone)      |  |                                      |  |
|  +-------------------+  |  +--------+  +-------------------+  |  |
|                          |  |Sidebar |  | Page Content      |  |  |
|  +-------------------+  |  |Nav     |  | (14 pages)        |  |  |
|  | GuidedTour        |  |  |14 items|  |                   |  |  |
|  | (overlay)         |  |  |        |  | Dashboard         |  |  |
|  +-------------------+  |  |API key |  | Jobs              |  |  |
|                          |  |status  |  | Candidates        |  |  |
|  +-------------------+  |  |indicator| | CandidateDetail   |  |  |
|  | API Client        |  |  |        |  | CandidateCRM      |  |  |
|  | (api.js)          |  |  |Tour    |  | Scheduling        |  |  |
|  |                   |  |  |button  |  | TalentMarketplace |  |  |
|  | request()         |  |  |        |  | TalentManagement  |  |  |
|  | aiRequest()       |  |  |Collapse|  | TalentDesign      |  |  |
|  | 40+ API methods   |  |  |toggle  |  | ResourceManagement|  |  |
|  +-------------------+  |  +--------+  | WorkforceExchange |  |  |
|                          |              | AIInterviewer     |  |  |
|                          |              | Analytics         |  |  |
|                          |              | GapAnalysis       |  |  |
|                          |              | Settings          |  |  |
|                          |              +-------------------+  |  |
|                          +--------------------------------------+  |
+-------------------------------------------------------------------+
```

#### API Client (`api.js`)
- `request(path, options)` - Base fetch wrapper with JSON handling and error extraction
- `aiRequest(path, options)` - Extends `request()` by injecting `X-API-Key` header from `localStorage.getItem('anthropic_api_key')`
- 40+ named API methods organized by domain (jobs, candidates, employees, roles, skills, projects, interviews, mobility, analytics, settings, ai, crm, scheduling, succession, marketplace)

---

## C4: Code-Level Detail

### ai_service.py

```
Module: recruitment.ai_service

Private Helpers:
  _get_client(api_key: str) -> anthropic.Anthropic
      Creates Anthropic SDK client instance.

  _call_claude(api_key: str, system: str, prompt: str, max_tokens: int = 2000) -> str
      Sends a message to Claude Haiku 4.5 with system prompt.
      Model: "claude-haiku-4-5-20251001"
      Returns raw text content from first message block.

AI Functions (all follow same pattern):
  1. Build system prompt (role-specific persona)
  2. Build user prompt with structured data
  3. Call _call_claude()
  4. Parse JSON response (with fallback: find first { to last })
  5. Return dict or error dict

Functions:
  parse_resume(api_key, resume_text) -> dict
      Extracts: name, email, phone, location, skills, experience_years,
      education, current_role, summary

  match_candidate_to_job(api_key, candidate_skills, candidate_resume,
                         job_title, job_requirements, job_skills) -> dict
      Returns: overall_score (0-100), skill/experience/culture scores,
      matched/missing skills, recommendation enum

  suggest_career_paths(api_key, current_role, skills,
                       experience_years, career_goals) -> dict
      Returns: recommended_paths, skills_to_develop, recommended_courses,
      mentorship_areas, summary

  generate_job_description(api_key, title, department,
                           required_skills, level) -> dict
      Returns: title, description, requirements, responsibilities,
      qualifications, salary_range, benefits

  generate_interview_questions(api_key, job_title, job_requirements,
                               candidate_skills, candidate_resume) -> dict
      Returns: 8 questions (3 technical, 3 behavioral, 2 situational)
      Each has: question, type, difficulty, evaluation_criteria, ideal_answer_hints

  conduct_ai_interview(api_key, job_title, question, answer,
                       evaluation_criteria) -> dict
      Returns: score (1-10), strengths, areas_for_improvement,
      follow_up_question, feedback

  analyze_skill_gaps(api_key, employee_skills, target_role,
                     target_skills) -> dict
      Returns: gap_score (0-100), matched/missing/transferable skills,
      development_plan, readiness_assessment

  forecast_workforce_trends(api_key, current_skills, department,
                            team_size) -> dict
      Returns: emerging/declining skills, hiring_recommendations,
      reskilling_opportunities, market_insights, risk_areas

Mock Functions:
  mock_parse_resume(resume_text) -> dict
  mock_match_candidate(candidate_skills, job_skills) -> dict
  mock_career_paths(current_role, skills) -> dict
  mock_interview_questions(job_title) -> dict
```

### models.py

```
Module: recruitment.models
Base: SQLAlchemy DeclarativeBase (from recruitment.database)

All models use Integer primary key, DateTime created_at (UTC default).
JSON columns store lists/dicts as native JSON (SQLite JSON1).

14 Models:
  Setting        - key/value config store (key: String(100) unique, value: Text)
  Skill          - name (unique), category, description, demand_trend
  Job            - title, department, location, job_type, description,
                   requirements, required_skills (JSON), salary range, status
                   Relationship: candidates (one-to-many -> Candidate)
  Candidate      - name, email, phone, location, resume_text, skills (JSON),
                   experience_years, education, current_role, match_score,
                   status, notes, anonymized (bool), job_id (FK -> jobs)
  Employee       - name, email, department, current_role, skills (JSON),
                   experience_years, career_goals, performance_rating (1-5),
                   mentoring_interests (JSON), courses_completed (JSON)
  Role           - title, department, level, description, required_skills (JSON),
                   career_paths (JSON: list of role IDs), market_demand
  Project        - name, description, department, required_skills (JSON),
                   team_size, assigned_employees (JSON: list of emp IDs),
                   status, start/end_date, utilization (0-100%)
  Interview      - candidate_id (FK), job_id (FK), questions (JSON),
                   answers (JSON), scores (JSON), overall_score, feedback, status
  MobilityListing - role_id (FK), title, department, description,
                    required_skills (JSON), listing_type, status
  OutreachTemplate - name, subject, body, template_type
  Engagement     - candidate_id (FK), template_id (FK), channel, status,
                   message, sent_at
  ScheduledInterview - candidate_id (FK), job_id (FK), interviewer_name,
                       interview_type, scheduled_date/time, duration_minutes,
                       status, notes, meeting_link
  SuccessionPlan - role_id (FK), current_holder_id (FK -> employees),
                   successor_ids (JSON: list of {employee_id, readiness,
                   timeline_months}), risk_level, notes
  MarketplaceListing - title, listing_type, department, description,
                       required_skills (JSON), duration, time_commitment,
                       posted_by_id (FK -> employees), applicant_ids (JSON)
```

### database.py

```
Module: recruitment.database

DATABASE_URL: "sqlite+aiosqlite:///<path>/recruitment.db"
engine: AsyncEngine (echo=False)
async_session: async_sessionmaker[AsyncSession]
Base: DeclarativeBase

Functions:
  get_db() -> AsyncGenerator[AsyncSession]   (FastAPI dependency)
  init_db() -> None                          (creates all tables)
```
