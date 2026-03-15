---
name: "Constraints & Decisions"
project: "AWS Academy"
project_slug: "aws-academy"
category: "learning"
type: "constraints"
icon: "☁️"
tags: ['FastAPI', 'React', 'Recharts', 'learning', 'aws']
---

# Constraints & Architectural Decisions

## Hard Constraints

1. **No database** — All content is static Python data structures. Zero SQLite/ORM/migration complexity. Content changes require code deployment.

2. **No AI API keys** — No `.env` file, no BYOK, no ANTHROPIC_API_KEY. 100% offline-capable. No AI features in this academy.

3. **No authentication** — Public learning platform. No user accounts, no login, no session management.

4. **No server-side persistence** — No user progress stored on backend. Browser localStorage only.

5. **No user-generated content** — All content is author-curated. No comments, no ratings, no uploads.

## Technology Choices

| Concern | Decision | Rationale |
|---------|----------|-----------|
| Backend framework | FastAPI | Consistent with all Sats Solution Registry apps |
| Frontend framework | React 18 + Vite | Same as all other portfolio apps |
| Styling | Tailwind CSS | Dark theme with custom AWS orange `aws` palette |
| Charts | Recharts | Quiz score visualization (bar chart) |
| Data storage | Python dicts | No CRUD needed — content is static |
| Port (backend) | 8032 | Available in the Sats port range |
| Port (frontend) | 5191 | Available in the Sats port range |
| Package name | `awsacademy` | Matches academy pattern |
| Docker port | 8033 | For consolidated EC2 instance |

## Content Scope

| Dimension | Count |
|-----------|-------|
| Lessons | 42 |
| Categories | 14 |
| Quiz Questions | 15 |
| Papers & References | 14 |
| Tour Steps | 6 |

## Dependencies

### Backend (minimal)
```
fastapi
uvicorn[standard]
pydantic
httpx
python-multipart
```

### Frontend
```
react, react-dom, react-router-dom
recharts
@vitejs/plugin-react
tailwindcss, postcss, autoprefixer
```

## Deployment

- **Local:** `./start.sh` (backend 8032 + frontend 5191)
- **Docker:** Multi-stage build, single container, port 8033

## Non-Goals

- AWS sandbox or live console integration
- Lab environments with real AWS resources
- Certification tracking or exam prep modes
- Real-time collaboration or multiplayer features
- AI-generated explanations or tutoring

