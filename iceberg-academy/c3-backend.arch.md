---
name: "C3 Backend Components"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "c4-component"
icon: "🧊"
tags: [FastAPI, Python, aiosqlite, Pydantic, SQLite]
---

# Iceberg Academy -- C3: Backend Components

## Routers (9)

| Router | Prefix | Endpoints | Responsibility |
|--------|--------|-----------|----------------|
| `modules` | `/api/modules` | 3 | List modules, get module detail with lessons, get individual lesson |
| `progress` | `/api/progress` | 4 | Get progress summary (by technology), mark/unmark lesson complete, list completed lessons |
| `quizzes` | `/api/quizzes` | 3 | Get quiz questions for a module, check answer (records attempt), quiz stats summary |
| `playground` | `/api/playground` | 4 | Get preloaded code examples (by technology or all), save user snippet, list saved snippets |
| `glossary` | `/api/glossary` | 2 | List glossary terms (optional technology filter), get single term |
| `labs` | `/api/labs` | 2 | List all labs (ordered by module + order_num), get single lab with instructions/starter/solution code |
| `architecture` | `/api/architecture` | 1 | Return 3 interactive architecture diagrams (nodes + edges for React Flow) |
| `papers` | `/api/papers` | 2 | List 17 curated papers/specs, get single paper with full details |
| `patterns` | `/api/patterns` | 1 | Return 10 data architecture and design patterns |

## Core Modules

| Module | File | Purpose |
|--------|------|---------|
| `main.py` | Application entry point | FastAPI app creation, CORS middleware, lifespan (init_db on startup), health endpoint, router registration |
| `database.py` | Database layer | `get_db()` returns aiosqlite connection (WAL + FK enabled); `init_db()` runs schema.sql on startup |
| `models.py` | Pydantic models | 22 request/response models covering all domain entities |
| `schema.sql` | DDL + seed data | 9 tables + INSERT OR IGNORE seed data (idempotent) |

## Data Flow

```
Request → FastAPI Router → get_db() → aiosqlite → SQLite (WAL)
                               ↓
                         Pydantic Model → JSON Response
```

All routers follow the same pattern: acquire connection, execute query, close connection in `finally` block. No connection pooling -- fresh connection per request.

## Static vs. Dynamic Content

| Content Type | Storage | Count |
|-------------|---------|-------|
| Modules, Lessons, Quizzes, Glossary, Labs | SQLite (seeded from schema.sql) | Dynamic counts via DB |
| Papers | In-memory Python list (papers.py) | 17 |
| Patterns | In-memory Python list (patterns.py) | 10 |
| Architecture Diagrams | In-memory Python objects (architecture.py) | 3 |
| Playground Examples | In-memory Python dict (playground.py) | 13 examples across 5 technologies |
| User Progress, Quiz Attempts, Snippets | SQLite (user-generated) | Variable |
