---
name: "API Specification"
project: "Azure Academy"
project_slug: "azure-academy"
category: "learning"
type: "api-spec"
icon: "🔷"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'azure']
---

# API Specification

**Base URL:** `http://localhost:8034/api`
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
  "app": "Azure Academy",
  "total_lessons": 42,
  "total_categories": 14,
  "total_quiz_questions": 15
}
```

## Lessons

### GET /api/lessons/
List all lessons (summary view — no content fields).

**Response 200:** Array of lesson summaries.
```json
[
  {"id": 1, "category": "Cloud Foundations", "title": "What is Azure?", "type": "intro"},
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
  {"category": "Cloud Foundations", "count": 3, "lessons": [{"id": 1, "title": "..."}, ...]},
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
    "category": "Identity and Access",
    "question": "Which Azure service provides identity management...?",
    "options": ["Entra ID", "Key Vault", "Defender", "Sentinel"]
  }
]
```

### POST /api/quiz/check
Check a single answer. Server-side scoring prevents cheating.

**Request:**
```json
{"question_id": 1, "selected": 0}
```

**Response 200:**
```json
{
  "correct": true,
  "correct_index": 0,
  "explanation": "Microsoft Entra ID (formerly Azure AD) provides..."
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

