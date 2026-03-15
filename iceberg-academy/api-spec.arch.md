---
name: "API Specification"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "api-spec"
icon: "🧊"
tags: [FastAPI, REST, SQLite, Iceberg, Arrow]
---

# Iceberg Academy -- API Specification

**Framework:** FastAPI 0.100+ | **Base URL:** `/api` | **Port:** 8000 (configurable)

## Endpoints Overview

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | System health with entity counts |
| GET | `/api/modules` | List all curriculum modules |
| GET | `/api/modules/{slug}` | Module detail with lesson list |
| GET | `/api/modules/{slug}/lessons/{lessonSlug}` | Full lesson content |
| GET | `/api/progress` | Progress summary with completion % |
| POST | `/api/progress/complete` | Mark lesson as completed |
| POST | `/api/progress/uncomplete` | Unmark lesson completion |
| GET | `/api/progress/lessons` | List completed lessons |
| GET | `/api/quizzes/{module_slug}` | Quiz questions for a module |
| POST | `/api/quizzes/check` | Check quiz answer |
| GET | `/api/quizzes/stats/summary` | Aggregate quiz statistics |
| GET | `/api/playground/examples/{technology}` | Code examples for a technology |
| GET | `/api/playground/examples` | All code examples grouped by technology |
| POST | `/api/playground/save` | Save user code snippet |
| GET | `/api/playground/snippets` | List saved snippets |
| GET | `/api/glossary` | List glossary terms |
| GET | `/api/glossary/{term_id}` | Single glossary term |
| GET | `/api/labs` | List all labs |
| GET | `/api/labs/{lab_id}` | Single lab with instructions and code |
| GET | `/api/architecture/diagrams` | Architecture diagrams (nodes + edges) |
| GET | `/api/papers/` | List 17 curated papers |
| GET | `/api/papers/{paper_id}` | Single paper with full details |
| GET | `/api/patterns/` | List 10 design patterns |

## Content Types

- `application/json` -- all responses and request bodies
- No file uploads, no binary responses

---

## Health

### `GET /api/health`

Returns system status and entity counts.

**Response:**
```json
{
  "status": "ok",
  "app": "Iceberg Academy",
  "modules": 5,
  "lessons": 25,
  "quizzes": 30,
  "glossary_terms": 40,
  "labs": 10,
  "papers": 17,
  "patterns": 10
}
```

---

## Modules (3 endpoints)

### `GET /api/modules`

List all modules with lesson and completion counts.

**Response:** `list[ModuleOut]`
```json
[
  {
    "id": 1,
    "slug": "iceberg-fundamentals",
    "title": "Apache Iceberg Fundamentals",
    "description": "Master the open table format...",
    "icon": "🧊",
    "order_num": 1,
    "technology": "iceberg",
    "lesson_count": 6,
    "completed_count": 3
  }
]
```

### `GET /api/modules/{slug}`

Module detail with ordered lesson list.

**Response:** `ModuleDetail`
```json
{
  "id": 1,
  "slug": "iceberg-fundamentals",
  "title": "Apache Iceberg Fundamentals",
  "description": "...",
  "icon": "🧊",
  "order_num": 1,
  "technology": "iceberg",
  "lessons": [
    {
      "id": 1,
      "title": "What is Apache Iceberg?",
      "slug": "what-is-iceberg",
      "order_num": 1,
      "difficulty": "beginner",
      "completed": false
    }
  ]
}
```

**Error:** `404` -- Module not found

### `GET /api/modules/{slug}/lessons/{lessonSlug}`

Full lesson content with code example.

**Response:** `LessonOut`
```json
{
  "id": 1,
  "module_id": 1,
  "title": "What is Apache Iceberg?",
  "slug": "what-is-iceberg",
  "content": "# Apache Iceberg\n\nApache Iceberg is an open table format...",
  "code_example": "from pyiceberg.catalog import load_catalog\n...",
  "order_num": 1,
  "difficulty": "beginner",
  "completed": false
}
```

**Error:** `404` -- Lesson not found

---

## Progress (4 endpoints)

### `GET /api/progress`

Aggregated learning progress.

**Response:** `ProgressSummary`
```json
{
  "total_lessons": 25,
  "completed_lessons": 8,
  "completion_pct": 32.0,
  "total_quizzes": 15,
  "correct_quizzes": 12,
  "quiz_accuracy": 80.0,
  "by_technology": {
    "iceberg": {"total": 6, "completed": 3},
    "arrow": {"total": 5, "completed": 2},
    "flight": {"total": 5, "completed": 1},
    "rest_catalog": {"total": 5, "completed": 2},
    "flightsql": {"total": 4, "completed": 0}
  }
}
```

### `POST /api/progress/complete`

Mark a lesson as completed. Upserts -- safe to call multiple times.

**Request:**
```json
{
  "lesson_id": 3,
  "notes": "Great explanation of hidden partitioning"
}
```

**Response:**
```json
{"status": "ok"}
```

### `POST /api/progress/uncomplete`

Remove lesson completion.

**Request:**
```json
{"lesson_id": 3}
```

**Response:**
```json
{"status": "ok"}
```

### `GET /api/progress/lessons`

List all completed lessons with timestamps.

**Response:** `list[CompletedLesson]`
```json
[
  {
    "lesson_id": 3,
    "lesson_title": "Hidden Partitioning",
    "module_title": "Apache Iceberg Fundamentals",
    "completed_at": "2024-06-15 10:30:00",
    "notes": "Great explanation"
  }
]
```

---

## Quizzes (3 endpoints)

### `GET /api/quizzes/{module_slug}`

Get quiz questions for a module. Correct answers are excluded.

**Response:** `list[QuizQuestionOut]`
```json
[
  {
    "id": 1,
    "module_id": 1,
    "question": "What problem does Apache Iceberg solve on object stores like S3?",
    "options": [
      "Lack of ACID transactions",
      "No support for Parquet files",
      "Slow compute performance",
      "Missing SQL syntax"
    ],
    "difficulty": "beginner"
  }
]
```

**Error:** `404` -- Module not found

### `POST /api/quizzes/check`

Submit an answer. Records the attempt and returns feedback.

**Request:**
```json
{
  "question_id": 1,
  "selected_answer": "Lack of ACID transactions"
}
```

**Response:** `CheckAnswerResponse`
```json
{
  "is_correct": true,
  "correct_answer": "Lack of ACID transactions",
  "explanation": "Iceberg brings ACID transactions to data lakes by using an optimistic concurrency model with atomic metadata swaps."
}
```

**Error:** `404` -- Question not found

### `GET /api/quizzes/stats/summary`

Aggregate quiz performance statistics.

**Response:** `QuizStats`
```json
{
  "total_attempts": 25,
  "correct_count": 20,
  "accuracy": 80.0,
  "by_module": {
    "iceberg-fundamentals": {"total": 10, "correct": 8},
    "rest-catalog": {"total": 5, "correct": 4}
  }
}
```

---

## Playground (4 endpoints)

### `GET /api/playground/examples/{technology}`

Preloaded code examples for a technology. Technologies: `iceberg`, `rest_catalog`, `arrow`, `flight`, `flightsql`.

**Response:** `list[CodeExample]`
```json
[
  {
    "title": "Create Iceberg Table (Spark)",
    "code": "from pyspark.sql import SparkSession\n...",
    "language": "python",
    "description": "Create a new Iceberg table using PySpark"
  }
]
```

### `GET /api/playground/examples`

All preloaded examples grouped by technology.

**Response:** `dict[str, list[CodeExample]]`

### `POST /api/playground/save`

Save a user code snippet to SQLite.

**Request:**
```json
{
  "title": "My Iceberg Query",
  "code": "SELECT * FROM demo.db.events",
  "language": "sql",
  "technology": "iceberg"
}
```

**Response:** `SnippetOut`
```json
{
  "id": 1,
  "title": "My Iceberg Query",
  "code": "SELECT * FROM demo.db.events",
  "language": "sql",
  "technology": "iceberg",
  "created_at": "2024-06-15 10:30:00"
}
```

### `GET /api/playground/snippets`

List saved snippets, newest first.

**Response:** `list[SnippetOut]`

---

## Glossary (2 endpoints)

### `GET /api/glossary`

List glossary terms. Optional `?technology=iceberg` filter.

**Query Parameters:**

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `technology` | string | No | Filter by technology |

**Response:** `list[GlossaryTermOut]`
```json
[
  {
    "id": 1,
    "term": "Snapshot",
    "definition": "An immutable view of an Iceberg table at a point in time...",
    "technology": "iceberg",
    "related_terms": ["Manifest List", "Metadata File", "Time Travel"]
  }
]
```

### `GET /api/glossary/{term_id}`

Single glossary term.

**Response:** `GlossaryTermOut`

**Error:** `404` -- Term not found

---

## Labs (2 endpoints)

### `GET /api/labs`

List all labs, ordered by module then order_num.

**Response:** `list[LabOut]`
```json
[
  {
    "id": 1,
    "module_id": 1,
    "title": "Build an Iceberg Table",
    "description": "Create and query your first Iceberg table",
    "instructions": "## Step 1: Set up catalog\n...",
    "starter_code": "from pyiceberg.catalog import load_catalog\n...",
    "solution_code": "catalog = load_catalog('demo', type='rest')\n...",
    "hints": ["Start with load_catalog()", "Use bucket partitioning"],
    "difficulty": "intermediate",
    "order_num": 1,
    "technology": "iceberg"
  }
]
```

### `GET /api/labs/{lab_id}`

Single lab with full instructions and code.

**Response:** `LabOut`

**Error:** `404` -- Lab not found

---

## Architecture (1 endpoint)

### `GET /api/architecture/diagrams`

Returns 3 interactive architecture diagrams as node/edge graphs for React Flow.

**Response:** `list[ArchitectureDiagram]`
```json
[
  {
    "title": "Modern Data Lakehouse Stack",
    "description": "How Iceberg, Arrow, Flight, FlightSQL, and the REST Catalog work together...",
    "nodes": [
      {"id": "iceberg", "label": "Apache Iceberg\n(Table Format)", "technology": "iceberg", "description": "Open table format with ACID...", "x": 400, "y": 400}
    ],
    "edges": [
      {"source": "arrow", "target": "iceberg", "label": "Read/write data files"}
    ]
  }
]
```

Diagrams included:
1. **Modern Data Lakehouse Stack** -- Full stack from applications to object storage (8 nodes, 9 edges)
2. **Iceberg Metadata Hierarchy** -- Catalog to metadata to snapshot to manifest list to manifest to data files (6 nodes, 5 edges)
3. **Arrow Flight Data Flow** -- Parallel retrieval: client to coordinator to workers to combined result (6 nodes, 8 edges)

---

## Papers (2 endpoints)

### `GET /api/papers/`

List all 17 curated papers (id, title, authors, year, category, summary).

**Response:** `list[dict]`

### `GET /api/papers/{paper_id}`

Full paper detail including URL and keyInsight.

**Response:**
```json
{
  "id": "iceberg-spec",
  "title": "Apache Iceberg Table Specification",
  "authors": "Apache Iceberg Community",
  "year": 2024,
  "category": "Iceberg",
  "url": "https://iceberg.apache.org/spec/",
  "summary": "The official Iceberg table format specification...",
  "keyInsight": "Iceberg's layered metadata enables O(1) file pruning..."
}
```

Categories: Iceberg (5), REST Catalog (3), Arrow (4), Flight (3), FlightSQL (2)

---

## Patterns (1 endpoint)

### `GET /api/patterns/`

Returns 10 data architecture and design patterns.

**Response:** `list[dict]`
```json
[
  {
    "name": "Lakehouse Architecture",
    "category": "Architecture",
    "complexity": "Intermediate",
    "technologies": ["iceberg", "arrow", "rest_catalog"],
    "description": "Combine data lake storage with warehouse reliability...",
    "flow": ["Ingest", "Write Parquet", "REST Catalog", "Query Engine", "BI"],
    "when": "Building a unified analytics platform...",
    "examples": ["Netflix data platform", "Apple's data lake"]
  }
]
```

Pattern categories: Architecture (4), Data Processing (2), Data Transport (2), Data Governance (2)

---

## Error Responses

All errors follow FastAPI's standard format:

```json
{
  "detail": "Module not found"
}
```

| Status | Meaning |
|--------|---------|
| `404` | Entity not found (module, lesson, question, term, lab, paper) |
| `422` | Validation error (invalid request body) |
