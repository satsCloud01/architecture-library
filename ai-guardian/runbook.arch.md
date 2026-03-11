---
name: "Deployment Runbook"
project: "Enterprise AI Guardian"
project_slug: "ai-guardian"
project_url: "https://ai-guardian.satszone.link"
github: "https://github.com/satsCloud01/ai-governance-bank"
category: "ai-agents"
type: "runbook"
icon: "🏦"
tags: [Docker, Nginx, AWS]
---

# Deployment Runbook — Enterprise AI Guardian

---

## Prerequisites

| Requirement | Version | Check |
|---|---|---|
| Python | 3.12+ | `python3 --version` |
| Node.js | 18+ | `node --version` |
| npm | 9+ | `npm --version` |
| Git | Any | `git --version` |

---

## Local Development (Demo)

### 1. Clone & Setup

```bash
cd /path/to/CodexFolder/ai-governance-bank
```

### 2. Backend Setup

```bash
cd backend

# Create virtual environment
python3 -m venv .venv

# Activate
source .venv/bin/activate          # macOS / Linux
.venv\Scripts\activate             # Windows

# Install dependencies
pip install -r requirements.txt

# Start server (auto-seeds demo data on first run)
PYTHONPATH=src .venv/bin/uvicorn governance.main:app --reload --port 8000
```

**Environment Variables (optional):**

| Variable | Default | Description |
|---|---|---|
| `DATABASE_URL` | `sqlite+aiosqlite:///./governance.db` | Database connection string |
| `CORS_ORIGINS` | `http://localhost:5173` | Allowed frontend origins |

### 3. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Start dev server (Vite proxies /api → :8000)
npm run dev
```

### 4. Quick Start (both servers)

```bash
# From project root
./start.sh
```

**URLs:**

| URL | Purpose |
|---|---|
| `http://localhost:5173` | Landing page |
| `http://localhost:5173/dashboard` | Main platform |
| `http://localhost:8000/docs` | Swagger API docs |
| `http://localhost:8000/redoc` | ReDoc API reference |

---

## Resetting Demo Data

```bash
# Stop servers, delete DB, restart
rm backend/governance.db
# Restart backend — seed runs automatically
```

---

## Production Deployment

### Option A: Docker Compose

```yaml
# docker-compose.yml
version: "3.9"
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: governance
      POSTGRES_USER: guardian
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pg_data:/var/lib/postgresql/data

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql+asyncpg://guardian:${DB_PASSWORD}@db:5432/governance
      CORS_ORIGINS: https://your-domain.com
    depends_on: [db]
    ports: ["8000:8000"]

  frontend:
    build: ./frontend
    ports: ["80:80"]
    depends_on: [backend]

volumes:
  pg_data:
```

**Backend Dockerfile:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY src/ ./src/
ENV PYTHONPATH=src
CMD ["uvicorn", "governance.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

**Frontend Dockerfile:**
```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json .
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

**Nginx config:**
```nginx
server {
  listen 80;
  root /usr/share/nginx/html;
  index index.html;

  location /api/ {
    proxy_pass http://backend:8000;
    proxy_set_header Host $host;
  }

  location / {
    try_files $uri $uri/ /index.html;
  }
}
```

### Option B: Kubernetes

```yaml
# Helm chart values (summary)
replicaCount: 2
backend:
  image: registry/ai-guardian-backend:latest
  resources:
    requests: { cpu: 500m, memory: 512Mi }
    limits:   { cpu: 2,    memory: 2Gi }
frontend:
  image: registry/ai-guardian-frontend:latest
  resources:
    requests: { cpu: 100m, memory: 128Mi }
postgresql:
  enabled: true
  auth:
    database: governance
```

---

## Switching to PostgreSQL

1. Install asyncpg: `pip install asyncpg`
2. Set `DATABASE_URL=postgresql+asyncpg://user:password@host:5432/governance`
3. Remove `aiosqlite` from requirements (optional)
4. Run Alembic migrations (recommended over `create_all` in production)

```bash
pip install alembic
alembic init alembic
# Configure alembic/env.py to use governance.database.Base and DATABASE_URL
alembic revision --autogenerate -m "initial"
alembic upgrade head
```

---

## Adding Authentication (Production)

1. Add `python-jose` and `passlib` to requirements
2. Create `governance/auth.py` with JWT encode/decode
3. Add `Depends(get_current_user)` to all write endpoints
4. Add role checking middleware per ADR-001 role table
5. Update frontend to store JWT in `httpOnly` cookie or `localStorage`

---

## Monitoring & Observability

| Tool | Purpose |
|---|---|
| Uvicorn access logs | Request-level logging |
| FastAPI middleware | Response time tracking |
| Prometheus + Grafana | Metrics (add `prometheus-fastapi-instrumentator`) |
| Sentry | Error tracking |
| PgBouncer | PostgreSQL connection pooling |

---

## Backup & Recovery

```bash
# SQLite backup
cp backend/governance.db backend/governance.db.backup.$(date +%Y%m%d)

# PostgreSQL backup
pg_dump -h localhost -U guardian governance > backup_$(date +%Y%m%d).sql

# Restore
psql -h localhost -U guardian governance < backup_20250301.sql
```
