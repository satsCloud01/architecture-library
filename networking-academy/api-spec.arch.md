---
title: Networking Academy — API Specification
base_url: http://localhost:8026/api
status: current
---

# API Specification

**Base URL:** `http://localhost:8026/api`
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
  "app": "Networking Academy",
  "total_lessons": 50,
  "total_categories": 13,
  "total_quiz_questions": 15
}
```

## Lessons

### GET /api/lessons/
List all lessons (summary view — no content fields).

**Response 200:** Array of lesson summaries.
```json
[
  {"id": 1, "category": "Fundamentals", "title": "Welcome to Networking Academy", "type": "intro"},
  {"id": 2, "category": "Fundamentals", "title": "Transmission Media", "type": "concept"},
  ...
]
```

### GET /api/lessons/{lesson_id}
Full lesson with all content fields.

**Response 200:** Complete lesson object including description, keyPoints, comparison, diagram, sequenceDiagram, code.

**Response 404:** `{"detail": "Lesson not found"}`

### GET /api/lessons/categories/summary
Lessons grouped by category with counts.

**Response 200:**
```json
[
  {"category": "Fundamentals", "count": 5, "lessons": [{"id": 1, "title": "..."}, ...]},
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
    "category": "Fundamentals",
    "question": "Which network device operates at Layer 2...?",
    "options": ["Hub", "Switch", "Router", "Access Point"]
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
  "explanation": "A switch operates at Layer 2..."
}
```

## Simulator

### GET /api/simulator/scenarios
List available simulation scenarios.

**Response 200:**
```json
[
  {"id": 1, "title": "TCP Three-Way Handshake", "description": "...", "total_steps": 7}
]
```

### GET /api/simulator/scenarios/{scenario_id}
Full scenario with all steps.

**Response 200:** Complete scenario object.

**Response 404:** `{"detail": "Scenario not found"}`

### POST /api/simulator/step
Get a specific step from a scenario (for progressive reveal).

**Request:**
```json
{"scenario_id": 1, "step_index": 0}
```

**Response 200:**
```json
{
  "step": {"type": "thought", "label": "Client Decision", "content": "..."},
  "step_index": 0,
  "total_steps": 7,
  "is_last": false
}
```

**Response 400:** `{"detail": "Invalid step index"}`

## Papers

### GET /api/papers/
All RFCs and papers with full metadata.

**Response 200:** Array of paper objects including key_takeaways and related_topics.

### GET /api/papers/{paper_id}
Single paper detail.

**Response 404:** `{"detail": "Paper not found"}`

## Error Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Invalid request (bad step index) |
| 404 | Resource not found |
| 422 | Validation error (Pydantic) |
