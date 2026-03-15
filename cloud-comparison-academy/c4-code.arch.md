---
name: "C4 - Code Level"
project: "Cloud Comparison Academy"
project_slug: "cloud-comparison-academy"
category: "learning"
type: "c4-code"
icon: "⚖️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'multi-cloud']
---

# C4 — Code Level

## C4 · Code Level

### Lesson Data Structure

```python
{
    "id": 3,
    "category": "Service Mapping",
    "title": "Compute Services Comparison",
    "type": "concept",
    "description": "Each cloud provider offers compute at...",
    "keyPoints": ["AWS: EC2, Lambda, ECS, EKS...", ...],
    "comparison": {
        "title": "Compute Services",
        "headers": ["Capability", "AWS", "Azure", "GCP"],
        "rows": [
            ["VMs", "EC2", "Virtual Machines", "Compute Engine"],
            ...
        ]
    }
}
```

### Calculator Model (frontend-only)

```javascript
// 8 criteria with adjustable weights (0-10)
const CRITERIA = [
  "Service Breadth", "Enterprise Governance", "Data & AI",
  "Developer Experience", "Hybrid/Multi-Cloud", "Cost Optimization",
  "Security & Compliance", "Operational Maturity"
];

// 3 providers with baseline ratings (0-5 scale)
// 4 presets: Balanced, Microsoft Enterprise, Data/AI First, Ops Simplicity
// Weighted average scoring: sum(weight * rating) / sum(weights)
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
"cloud-comparison-completed" → [1, 2, 3, ...]  // Completed lesson IDs
"cloud-comparison-academy-tour" → "true"        // Tour dismissed
```

No server-side persistence. Progress is per-browser.

