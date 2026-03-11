---
name: "C2 – Container Diagram"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "c4-container"
icon: "🧠"
tags: [FastAPI, React, S3]
---

# C2 — Container Diagram

> **Scope:** The containers (deployable/runnable units) that make up SatsKnowledge Cortex.

```mermaid
C4Container
    title Container Diagram — SatsKnowledge Cortex

    Person(user, "User (Sats)", "Accesses the app through a web browser")
    Person(recipient, "Link Recipient", "Reads a shared article via public URL")

    System_Ext(s3, "AWS S3", "Binary file storage in production")
    System_Ext(web, "External Websites", "Fetched at link-save time")
    System_Ext(embed, "YouTube / Vimeo / SoundCloud", "Embedded media players")

    System_Boundary(cortex, "SatsKnowledge Cortex") {

        Container(spa, "Single-Page Application", "React 18 · TypeScript · Vite · Tailwind CSS · shadcn/ui",
            "Renders the full knowledge base UI: article list, search, viewers, modals, notes pane, share popover. Built to static assets served by the API in production.")

        Container(api, "FastAPI Backend", "Python 3.12 · FastAPI · uvicorn · SQLAlchemy 2.0",
            "REST API for all CRUD operations, file upload/download, full-text search, note management, share token generation, and public share page rendering. Serves the React SPA as a static fallback in production.")

        ContainerDb(db, "SQLite Database", "SQLite 3 · FTS5 virtual table · SQLAlchemy async · aiosqlite",
            "Persists all articles, tags, notes, and share tokens. FTS5 virtual table with Porter stemmer enables full-text search across title, body, and summary with snippet highlighting.")

        ContainerDb(fs, "File Storage", "Local filesystem (dev) · AWS S3 (prod) · StorageBackend abstraction",
            "Binary blobs: PDFs, DOCX files, HTML pages, uploaded video, uploaded audio. Accessed via a protocol-based abstraction that switches between local (aiofiles) and S3 (boto3) at startup.")
    }

    Rel(user, spa, "Uses", "HTTPS · Browser")
    Rel(recipient, api, "Views shared article", "HTTPS · Browser (server-rendered HTML)")

    Rel(spa, api, "REST/JSON API calls", "HTTP (dev: proxied via Vite) · same-origin (prod)")

    Rel(api, db, "Reads / writes entities", "SQL async via SQLAlchemy")
    Rel(api, fs, "Stores / retrieves files", "aiofiles (local) · boto3 (S3)")
    Rel(api, web, "Fetches page text at save time", "httpx async · HTML/BeautifulSoup4")
    Rel(api, embed, "Generates embed URLs for link articles", "URL construction (no API key needed)")

    Rel(fs, s3, "Replicates to cloud in prod", "AWS SDK · TLS")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Container Responsibilities

| Container | Technology | Deployed where |
|---|---|---|
| SPA | React · Vite build → `backend/src/cortex/static/` | Served by FastAPI `StaticFiles` mount in prod; Vite dev server on `:5173` in dev |
| FastAPI Backend | Python 3.12 · uvicorn | Local: `uvicorn cortex.main:app` on `:8000` · EC2: systemd `cortex.service` behind Nginx |
| SQLite | `./data/cortex.db` | Local disk (dev) · `/opt/cortex/data/cortex.db` (EC2) |
| File Storage | `./data/files/` (dev) | Local disk (dev) · AWS S3 bucket `sats-knowledge-cortex-prod` (EC2) |

## Dev vs Production Topology

```
DEV (two processes)                PROD (single process + Nginx)
──────────────────                 ─────────────────────────────
npm run dev  → :5173               Nginx :80
  └─ proxies /api → :8000            ├─ /api/*  → uvicorn :8000
                                     ├─ /public/*  → uvicorn :8000
uvicorn :8000                        └─ /*  → uvicorn :8000 (serves index.html)
  └─ SQLite + local FS               uvicorn :8000
                                       └─ SQLite + AWS S3
```
