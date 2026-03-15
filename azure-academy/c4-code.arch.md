---
name: "C4 - Code Level"
project: "Azure Academy"
project_slug: "azure-academy"
category: "learning"
type: "c4-code"
icon: "🔷"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'azure']
---

# C4 — Code Level

## C4 · Code Level

### Lesson Data Structure

```python
{
    "id": 5,
    "category": "Identity and Access",
    "title": "Microsoft Entra ID and RBAC",
    "type": "concept",
    "description": "Azure identity management centers on...",
    "keyPoints": ["Entra ID (formerly Azure AD)...", ...],
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
    "code": "az role assignment create ...",
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
"azure-academy-completed" → [1, 2, 3, ...]  // Completed lesson IDs
"azure-academy-tour" → "true"               // Tour dismissed
```

No server-side persistence. Progress is per-browser.

