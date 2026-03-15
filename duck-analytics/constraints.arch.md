---
name: "Constraints & ADRs"
project: "Duck Analytics"
project_slug: "duck-analytics"
project_url: null
github: "https://github.com/satsCloud01/duck-analytics"
category: "learning"
type: "constraints"
icon: "🦆"
tags: [Litestar, DuckDB, Arrow, constraints, ADR]
---

# Duck Analytics -- Constraints & Design Decisions

## Key ADRs

1. **Litestar over FastAPI** — Class-based controllers, native dataclass validation; demonstrates an alternative async Python framework
2. **DuckDB In-Process** — Embedded OLAP database, no server to manage; zero-copy Arrow integration
3. **Arrow IPC as Binary Transport** — Demonstrates columnar binary format advantages (3-5x faster/smaller than JSON)
4. **Auto-Seeded Demo Data** — 3 demo tables (6,200 rows) created on startup; idempotent seeding
5. **System Tables Prefixed `_`** — Internal tables hidden from schema listing

## Constraints

| Area | Constraint |
|------|-----------|
| Auth | None — educational/demo tool |
| AI | None — pure data engineering focus |
| DB | Single DuckDB file, one writer at a time |
| Pagination | None — full `fetchall()` |
| Ports | Backend 8009, Frontend 5180 |
| Python | 3.12+ required |
