---
name: "Constraints & NFRs"
project: "SatsRecruitment Intelligence"
project_slug: "sats-recruitment"
project_url: "https://sats-recruitment.satszone.link"
github: "https://github.com/satsCloud01/sats-recruitment"
category: "productivity"
type: "constraints"
icon: "🧠"
tags: [Performance, Security]
---

# SatsRecruitment Intelligence - Constraints

## Technology Stack Constraints

### Backend
- **Python 3.12** required
- **FastAPI 0.115** with async request handlers
- **SQLAlchemy 2.0** with async engine -- requires `aiosqlite` and `greenlet` packages
- **SQLite** as the sole database -- no external database server required
- **Anthropic SDK 0.42** for Claude API integration
- **Pydantic v2** for all request/response validation
- All database operations are async via `AsyncSession`

### Frontend
- **React 19** with functional components and hooks only (no class components)
- **Vite 7** as build tool and dev server
- **Tailwind CSS v4** for styling (plugin-based Vite integration, no PostCSS config)
- **React Router v7** for client-side routing
- **Recharts 3** for data visualization
- **Lucide React** for iconography
- No TypeScript -- plain JSX throughout
- No state management library (React state + prop drilling)

### Shared
- Frontend proxies `/api` requests to backend via Vite dev server config
- No authentication/authorization system (open access to all endpoints)
- No WebSocket or real-time features

---

## Security Constraints

### API Key Management
- Anthropic API key is stored **exclusively in browser localStorage** (`anthropic_api_key` key)
- Frontend sends the key via `X-API-Key` HTTP header on AI-related requests only
- Backend **never** stores the API key in the database -- the settings router explicitly rejects `anthropic_api_key` writes
- Backend validates key format: must start with `sk-ant-` and be longer than 10 characters
- Key is visible in browser dev tools (localStorage and network headers)

### CORS
- CORS is configured with `allow_origins=["*"]` -- any origin can make requests
- `allow_credentials=True` and all methods/headers are permitted
- This is a demo/pitch application configuration, not production-hardened

### Data Privacy
- Candidate anonymization feature: toggling `anonymized` replaces name with "Candidate #N" and masks email/phone with "***" in API responses
- Resume text is sent to Anthropic Claude API when AI features are used (with user-provided API key)
- No data encryption at rest (SQLite file is unencrypted)
- No audit logging of data access

### No Authentication
- All API endpoints are publicly accessible (no login, no session management, no RBAC)
- The only "auth" is the optional X-API-Key header for AI features, which authenticates with Anthropic, not with the application itself

---

## Performance Constraints

### Database
- SQLite is single-writer -- concurrent write operations are serialized
- No connection pooling configuration (default aiosqlite behavior)
- JSON columns (skills, assigned_employees, successor_ids, etc.) are not indexed -- queries filtering by JSON content require full table scans
- Analytics endpoints perform multiple sequential queries (e.g., dashboard makes 8+ queries)
- Seed data is moderate: ~130 total records across all tables

### AI Service
- All Claude API calls are **synchronous** (blocking the async event loop via the Anthropic SDK's synchronous client)
- No request caching -- repeated identical AI requests make fresh API calls
- No rate limiting on AI endpoints
- No streaming -- full response must complete before returning
- Resume text is truncated to 1000 characters for matching, 500 for interview generation
- `max_tokens` defaults to 2000 per AI call

### Frontend
- No pagination on any list endpoint -- all records are returned in a single response
- No client-side caching or state persistence (except localStorage for API key)
- No lazy loading of page components
- No service worker or offline support

---

## Deployment Constraints

### Local Development
- Backend requires Python 3.12 virtual environment at `backend/.venv/`
- Run backend: `cd backend && PYTHONPATH=src .venv/bin/uvicorn recruitment.main:app --reload --port 8000`
- Run frontend: `cd frontend && npm run dev` (port 5173, proxies /api to :8000)
- Node.js must be available (tested with Node 25.6)
- No `.env` file required -- the only external dependency (Anthropic API key) is provided via the browser UI

### Database
- SQLite database file is created at `backend/recruitment.db` (two levels up from the models.py source)
- Database is auto-created and auto-seeded on first application startup
- No migration system -- schema changes require deleting the database file and restarting
- Seed data checks for existing records (idempotent: only seeds if `skills` table is empty)

### Docker / Production
- No Dockerfile or docker-compose.yml included in the project
- No production build configuration documented
- Frontend production build: `npm run build` (outputs to `dist/`)
- Backend has no production ASGI server configuration beyond uvicorn

---

## Business Constraints

### Scope
- This is a **pitch/demo application** -- not production software
- All seed data represents a fictional tech company's recruitment operations
- Financial figures (salaries, cost-per-hire) are realistic mock values, not connected to real payroll systems
- Hiring metrics (time-to-hire, cost-per-hire) are hardcoded mock values, not computed from real data
- No integration with external HR systems (ATS, HRIS, payroll)
- No email delivery -- CRM engagements are records only, not actual sent messages
- No calendar integration -- scheduled interviews are records only

### AI Features
- All 9 AI endpoints gracefully degrade to mock/heuristic responses when no API key is provided
- Mock responses are deterministic and less useful than real AI responses but allow full UI demonstration
- AI model is fixed to Claude Haiku 4.5 (`claude-haiku-4-5-20251001`) -- not configurable
- No prompt versioning or A/B testing infrastructure
- JSON parsing from AI responses uses a fallback strategy: try full response, then extract first `{` to last `}`
