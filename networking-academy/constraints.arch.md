---
title: Networking Academy — Constraints & Decisions
status: current
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
| Styling | Tailwind CSS | Same brand/config as other apps |
| Charts | Recharts | Quiz score visualization (bar chart) |
| Data storage | Python dicts | No CRUD needed — content is static |
| Port (backend) | 8026 | Available in the Sats port range |
| Port (frontend) | 5182 | Available in the Sats port range |
| Package name | `networkacademy` | Matches pattern: `academy`, `duckanalytics`, etc. |
| Docker port | 8027 | For consolidated EC2 instance |

## Content Scope

| Dimension | Count |
|-----------|-------|
| Lessons | 50 |
| Categories | 13 |
| Quiz Questions | 15 |
| Simulator Scenarios | 4 |
| Network Patterns | 10 (frontend-only) |
| Papers & RFCs | 17 |
| Tour Steps | 6 |

## Dependencies

### Backend (minimal)
```
fastapi
uvicorn[standard]
pydantic
httpx
pytest
```

### Frontend
```
react, react-dom, react-router-dom
recharts
@vitejs/plugin-react
tailwindcss, postcss, autoprefixer
```

## Deployment

- **Local:** `./start.sh` (backend 8026 + frontend 5182)
- **Docker:** Multi-stage build, single container, port 8027
- **ECR:** `637423642269.dkr.ecr.us-east-1.amazonaws.com/networking-academy:latest`
- **Production:** On-demand deploy via Solution Registry (30-min instances)

## Non-Goals

- Real network simulation (GNS3/EVE-NG integration)
- Lab environments or sandbox VMs
- Certification tracking or exam prep modes
- Real-time collaboration or multiplayer features
- AI-generated explanations or tutoring
