---
name: "C4 Code-Level Detail"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "c4-code"
icon: "🧊"
tags: [FastAPI, Python, aiosqlite, Pydantic, SQLite]
---

# Iceberg Academy -- C4: Code-Level Detail

## Router Pattern (all 9 routers)

Every router follows this structure:

```python
router = APIRouter(prefix="/api/<domain>", tags=["<domain>"])

@router.get("/<path>", response_model=SomeModel)
async def handler():
    db = await get_db()
    try:
        # SQL query via aiosqlite
        rows = await db.execute_fetchall("SELECT ...")
        return [dict(r) for r in rows]
    finally:
        await db.close()
```

## Key Functions

| Function | Location | Signature | Purpose |
|----------|----------|-----------|---------|
| `get_db()` | `database.py` | `async def get_db() -> aiosqlite.Connection` | Creates aiosqlite connection with WAL + FK pragmas |
| `init_db()` | `database.py` | `async def init_db()` | Reads schema.sql, executes DDL + seed data |
| `health()` | `main.py` | `async def health()` | Returns counts of modules, lessons, quizzes, glossary terms, labs, papers (17), patterns (10) |
| `check_answer()` | `quizzes.py` | `async def check_answer(req: CheckAnswerRequest)` | Validates answer, records quiz_attempt, returns explanation |
| `complete_lesson()` | `progress.py` | `async def complete_lesson(req: CompleteRequest)` | Upserts user_progress with ON CONFLICT handling |
| `get_diagrams()` | `architecture.py` | `async def get_diagrams()` | Returns 3 hardcoded ArchitectureDiagram objects (Modern Lakehouse, Metadata Hierarchy, Flight Data Flow) |
| `list_papers()` | `papers.py` | `async def list_papers()` | Returns 17 curated papers from in-memory PAPERS list |
| `list_patterns()` | `patterns.py` | `async def list_patterns()` | Returns 10 architecture patterns from in-memory PATTERNS list |
| `get_examples()` | `playground.py` | `async def get_examples(technology: str)` | Returns preloaded code examples (13 total across 5 technologies) |
