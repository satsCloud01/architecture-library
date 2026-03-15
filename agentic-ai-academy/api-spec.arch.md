---
name: "API Specification"
project: "Agentic AI Academy"
project_slug: "agentic-ai-academy"
category: "learning"
type: "api-spec"
icon: "🤖"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'agentic-ai']
---

# API Specification

**Base URL:** `http://localhost:8030/api`
**Content-Type:** `application/json`
**Authentication:** None required
**CORS:** All origins allowed

## Health

### GET /api/health
System status and content counts.

**Response 200:**
```json
{
  "status": "healthy",
  "app": "Agentic AI Academy",
  "total_lessons": 36,
  "total_categories": 12,
  "total_quiz_questions": 15
}
```

## Lessons

### GET /api/lessons/
List all lessons (summary view — no content fields).

**Response 200:** Array of lesson summaries.
```json
[
  {"id": 1, "category": "Agentic Foundations", "title": "The Agent Control Loop", "type": "intro"},
  ...
]
```

### GET /api/lessons/{lesson_id}
Full lesson with all content fields.

**Response 200:** Complete lesson object including description, keyPoints, comparison, diagram, code.

**Response 404:** `{"detail": "Lesson not found"}`

### GET /api/lessons/categories/summary
Lessons grouped by category with counts.

**Response 200:**
```json
[
  {"category": "Agentic Foundations", "count": 3, "lessons": [{"id": 1, "title": "..."}, ...]},
  ...
]
```

## Quiz

### GET /api/quiz/questions
All quiz questions **without** correct answers or explanations.

**Response 200:**
```json
[
  {
    "id": 1,
    "category": "MCP Deep Dive",
    "question": "What is the primary transport...",
    "options": ["HTTP", "JSON-RPC over stdio", "gRPC", "WebSocket"]
  }
]
```

### POST /api/quiz/check
Check a single answer. Server-side scoring prevents cheating.

**Request:**
```json
{"question_id": 1, "selected": 1}
```

**Response 200:**
```json
{
  "correct": true,
  "correct_index": 1,
  "explanation": "MCP uses JSON-RPC 2.0 over stdio..."
}
```

## Papers

### GET /api/papers/
All papers with full metadata.

**Response 200:** Array of paper objects including key_takeaways and related_topics.

### GET /api/papers/{paper_id}
Single paper detail.

**Response 404:** `{"detail": "Paper not found"}`

## Error Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 404 | Resource not found |
| 422 | Validation error (Pydantic) |

