---
name: "Constraints & ADRs"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "constraints"
icon: "🧊"
tags: [FastAPI, SQLite, constraints, ADR, Iceberg, Arrow]
---

# Iceberg Academy -- Constraints & Design Decisions

## Key ADRs

1. **No External AI APIs at Runtime** -- All content is pre-authored and seeded. Chat/tutor models are defined in Pydantic but no AI router is wired. The application works fully offline with no API keys required.

2. **SQLite with aiosqlite** -- Single-file embedded database. WAL journal mode for concurrent reads. Foreign keys enabled via PRAGMA. No connection pooling -- fresh connection per request, closed in `finally`.

3. **Static Content + DB Hybrid** -- Curriculum content (modules, lessons, quizzes, glossary, labs) lives in SQLite via schema.sql seed data. Reference content (papers, patterns, architecture diagrams, playground examples) lives as in-memory Python data structures in router files. This avoids complex seeding for read-only reference data.

4. **Idempotent Seeding** -- `CREATE TABLE IF NOT EXISTS` + `INSERT OR IGNORE` ensures schema.sql can run on every startup without duplicate data or errors.

5. **No Authentication** -- Single-user educational tool. No login, sessions, or multi-tenancy. user_progress tracks one learner's state. Suitable for local development or personal deployment.

6. **Pre-Authored Content Only** -- All educational content (25+ lessons, 30+ quiz questions, 40+ glossary terms, 17 papers, 10 patterns, 13 playground examples, 3 architecture diagrams) is bundled with the application. No content management or authoring UI.

7. **Connection-Per-Request Pattern** -- Every endpoint calls `get_db()` for a fresh aiosqlite connection and closes it in a `finally` block. Simple and correct for low-concurrency educational use, though not optimal for high-throughput scenarios.

8. **Quiz Answer Exclusion** -- The `GET /api/quizzes/{slug}` endpoint returns questions with options but deliberately excludes `correct_answer`. Answers are only revealed via the `POST /api/quizzes/check` endpoint, which also records the attempt.

9. **React Flow Architecture Diagrams** -- Architecture diagrams are defined as node/edge objects with x/y coordinates, designed for React Flow rendering. Three diagrams cover the full Iceberg/Arrow/Flight stack.

10. **Five Technology Taxonomy** -- All content is organized under exactly 5 technologies (iceberg, rest_catalog, arrow, flight, flightsql), enforced by a CHECK constraint on the modules table and a Pydantic enum.

## Constraints

| Area | Constraint |
|------|-----------|
| Auth | None -- single-user educational tool |
| AI | None at runtime -- all content pre-authored |
| DB | SQLite single file, WAL mode, one writer at a time |
| Pagination | None -- full result sets returned |
| Python | 3.12+ required |
| Content | Read-only curriculum; user can only track progress and save snippets |
| Technologies | Exactly 5: iceberg, rest_catalog, arrow, flight, flightsql |
| Papers | 17 curated, hardcoded -- not user-extensible |
| Patterns | 10 hardcoded -- not user-extensible |
| Architecture Diagrams | 3 hardcoded -- not user-extensible |
| Playground Examples | 13 preloaded across 5 technologies -- not user-extensible |

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Backend | FastAPI | 0.100+ |
| Async DB | aiosqlite | Latest |
| Validation | Pydantic v2 | 2.x |
| Database | SQLite 3 | System |
| Frontend | React | 18 |
| Build | Vite | 5.x |
| CSS | Tailwind CSS | 3.x |
| Routing | React Router | v6 |

## What the App Does NOT Do

- Does not execute user code (playground is view-only with save/load)
- Does not call any external APIs at runtime
- Does not support multiple users or user accounts
- Does not provide real Iceberg/Arrow/Flight environments
- Does not paginate any list endpoints
- Does not have WebSocket or real-time features
- Does not use any ORM -- raw SQL via aiosqlite throughout
