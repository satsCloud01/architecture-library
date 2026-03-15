---
name: "C4 - Code Level"
project: "AWS Academy"
project_slug: "aws-academy"
category: "learning"
type: "c4-code"
icon: "☁️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'aws']
---

# C4 — Code Level

## C4 · Code Level

### Lesson Data Structure

```python
{
    "id": 11,
    "category": "Networking",
    "title": "VPC Architecture and Subnet Design",
    "type": "concept",
    "description": "AWS networking uses regional VPCs...",
    "keyPoints": ["VPC: Regional virtual network...", ...],
    "diagram": {
        "type": "architecture",
        "nodes": [...],
        "edges": [...]
    },
    "comparison": {
        "title": "...",
        "headers": [...],
        "rows": [...]
    },
    "code": "aws ec2 run-instances ...",
    "codeLanguage": "bash"
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
"aws-academy-completed" → [1, 2, 3, ...]  // Completed lesson IDs
"aws-academy-tour" → "true"               // Tour dismissed
```

No server-side persistence. Progress is per-browser.

