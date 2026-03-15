---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: API Specification
type: api-spec
tags: [api, rest, endpoints]
---

# SatsLearning Academy — API Specification

**Agentic AI Educational Platform**

Base URL: `http://localhost:8025/api`

All endpoints return JSON. The API is entirely read-only (no database mutations). The two POST endpoints (`/api/quiz/check` and `/api/simulator/step`) perform stateless computations and return results without side effects. No authentication or API keys are required.

---

## Health Check

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/health` | Application health check — returns status and version |

### Response Schema

**GET /api/health**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "lessons": 45,
  "scenarios": 4,
  "questions": 10,
  "papers": 17
}
```

---

## 1. Lessons — `/api/lessons`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/lessons/` | List all 45 lessons. Optional query params: `category` |
| `GET` | `/api/lessons/{id}` | Get a single lesson by ID with full content and diagram data |
| `GET` | `/api/lessons/categories/summary` | Get category names, lesson counts, and ordering |

### Response Schemas

**GET /api/lessons/**
```json
[
  {
    "id": "foundations-01",
    "title": "What Are AI Agents?",
    "category": "Foundations",
    "order": 1,
    "key_concepts": ["autonomy", "tool-use", "reasoning"],
    "diagram_type": "agent"
  }
]
```

Note: The list endpoint returns lesson summaries (no `content` or `diagram_data`) to minimize payload size. Use the detail endpoint for full content.

**GET /api/lessons/{id}**
```json
{
  "id": "protocols-01",
  "title": "Model Context Protocol (MCP) Overview",
  "category": "Protocols",
  "order": 1,
  "content": "The Model Context Protocol enables LLMs to discover and invoke tools...",
  "key_concepts": ["MCP", "tool-discovery", "server-client", "capability-negotiation"],
  "diagram_type": "sequence",
  "diagram_data": {
    "participants": ["Client", "MCP Server", "LLM"],
    "messages": [
      {"from": "Client", "to": "MCP Server", "label": "initialize"},
      {"from": "MCP Server", "to": "Client", "label": "capabilities"},
      {"from": "Client", "to": "LLM", "label": "prompt + tools"},
      {"from": "LLM", "to": "Client", "label": "tool_call"},
      {"from": "Client", "to": "MCP Server", "label": "execute_tool"},
      {"from": "MCP Server", "to": "Client", "label": "result"}
    ]
  }
}
```

**GET /api/lessons/categories/summary**
```json
[
  {"name": "Foundations", "lesson_count": 6, "order": 1},
  {"name": "Protocols", "lesson_count": 13, "order": 2},
  {"name": "Design Patterns", "lesson_count": 7, "order": 3},
  {"name": "Architectures", "lesson_count": 5, "order": 4},
  {"name": "Memory & State", "lesson_count": 4, "order": 5},
  {"name": "Frameworks", "lesson_count": 5, "order": 6},
  {"name": "Advanced", "lesson_count": 5, "order": 7}
]
```

---

## 2. Simulator — `/api/simulator`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/simulator/scenarios` | List all 4 simulator scenarios (summary only) |
| `GET` | `/api/simulator/scenarios/{id}` | Get full scenario detail including all steps |
| `POST` | `/api/simulator/step` | Advance one step in a scenario — returns the next step for animated playback |

### Request/Response Schemas

**GET /api/simulator/scenarios**
```json
[
  {
    "id": "react-web-search",
    "title": "ReAct Web Search Agent",
    "description": "Step through a ReAct agent performing iterative web search",
    "agent_type": "react",
    "total_steps": 9
  }
]
```

**POST /api/simulator/step — Request**
```json
{
  "scenario_id": "react-web-search",
  "current_step": 2
}
```

**POST /api/simulator/step — Response**
```json
{
  "scenario_id": "react-web-search",
  "step": {
    "step_number": 3,
    "phase": "observation",
    "content": "Search returned 3 relevant results...",
    "tool": null,
    "tool_input": null,
    "tool_output": {"results": ["Tokyo metropolitan area: 13.96 million..."]}
  },
  "is_last": false,
  "progress": 0.33
}
```

---

## 3. Quiz — `/api/quiz`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/quiz/questions` | Get all 10 quiz questions (without correct answers) |
| `POST` | `/api/quiz/check` | Submit answers and receive scored results with category breakdown |

### Request/Response Schemas

**GET /api/quiz/questions**
```json
[
  {
    "id": "q01",
    "question": "Which protocol enables tool discovery and invocation for LLMs?",
    "options": ["A2A", "MCP", "gRPC", "GraphQL"],
    "category": "Protocols"
  }
]
```

Note: The `correct` and `explanation` fields are intentionally omitted to prevent answer leaking in the browser.

**POST /api/quiz/check — Request**
```json
{
  "answers": {
    "q01": "MCP",
    "q02": "Reasoning"
  }
}
```

**POST /api/quiz/check — Response**
```json
{
  "score": 8,
  "total": 10,
  "percentage": 80.0,
  "results": [
    {
      "id": "q01",
      "correct": true,
      "your_answer": "MCP",
      "correct_answer": "MCP",
      "explanation": "Model Context Protocol (MCP) provides a standardized way for LLMs to discover and invoke tools."
    }
  ],
  "by_category": {
    "Protocols": {"correct": 3, "total": 4},
    "Foundations": {"correct": 2, "total": 2}
  }
}
```

---

## 4. Papers — `/api/papers`

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/papers/` | List all 17 research papers. Optional query params: `organization`, `tag` |
| `GET` | `/api/papers/{id}` | Get a single paper by ID with full metadata |

### Response Schemas

**GET /api/papers/**
```json
[
  {
    "id": "paper-01",
    "title": "Constitutional AI: Harmlessness from AI Feedback",
    "authors": ["Yuntao Bai", "et al."],
    "organization": "Anthropic",
    "year": 2022,
    "url": "https://arxiv.org/abs/2212.08073",
    "tags": ["alignment", "safety", "RLHF"]
  }
]
```

---

## Common Patterns

### Error Responses

```json
{
  "detail": "Lesson not found",
  "status_code": 404
}
```

Standard HTTP status codes: 200 (OK), 400 (Bad Request), 404 (Not Found), 422 (Validation Error).

### No Authentication

All endpoints are publicly accessible. No API keys, tokens, or headers required.

### No Pagination

Given the small, fixed dataset sizes (45 lessons, 4 scenarios, 10 questions, 17 papers), endpoints return full lists without pagination.
