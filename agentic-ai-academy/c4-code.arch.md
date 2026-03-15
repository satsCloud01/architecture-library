---
name: "C4 - Code Level"
project: "Agentic AI Academy"
project_slug: "agentic-ai-academy"
category: "learning"
type: "c4-code"
icon: "🤖"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'agentic-ai']
---

# C4 — Code Level

## C4 · Code Level

### Lesson Data Structure

```python
{
    "id": 6,
    "category": "MCP Deep Dive",
    "title": "MCP Architecture: Host-Client-Server",
    "type": "concept",
    "description": "Model Context Protocol standardizes...",
    "keyPoints": ["Host Application manages clients...", ...],
    "diagram": {
        "type": "architecture",
        "nodes": [...],
        "edges": [...]
    },
    "comparison": {
        "title": "...",
        "headers": [...],
        "rows": [[...], ...]
    },
    "code": "...",
    "codeLanguage": "JSON-RPC"
}
```

### Quiz Flow

```
Browser                    Backend
  │                          │
  │ GET /quiz/questions ────▶│  Returns questions WITHOUT correct answers
  │◀──── [{id, question,     │
  │        options}]          │
  │                          │
  │ POST /quiz/check ───────▶│  Server-side scoring
  │  {question_id, selected} │
  │◀──── {correct, index,    │
  │        explanation}       │
```

### Progress Tracking

```javascript
// Browser localStorage
"agentic-academy-completed" → [1, 2, 3, ...]  // Completed lesson IDs
"agentic-ai-academy-tour" → "true"            // Tour dismissed
```

No server-side persistence. Progress is per-browser.

