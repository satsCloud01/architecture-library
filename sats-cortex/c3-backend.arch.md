---
name: "C3 – Backend Components"
project: "Sats Cortex"
project_slug: "sats-cortex"
project_url: "https://cortex.satszone.link"
github: ""
category: "productivity"
type: "c4-component"
icon: "🧠"
tags: [FastAPI, SQLAlchemy]
---

# C3 — Backend Component Diagram

> **Scope:** Internal components of the **FastAPI Backend** container.

```mermaid
C4Component
    title Component Diagram — FastAPI Backend

    Person(user, "User / Browser", "Authenticated requests from the SPA")
    Person(recipient, "Share Recipient", "Unauthenticated public share access")
    ContainerDb(db, "SQLite + FTS5", "Async SQLAlchemy session")
    ContainerDb(storage, "File Storage", "LocalStorageBackend or S3StorageBackend")
    System_Ext(web, "External Websites", "Fetched for link articles")

    Container_Boundary(api, "FastAPI Backend  [Python 3.12 · FastAPI · uvicorn]") {

        %% ── Entry point ──────────────────────────────────────────────────────
        Component(main, "main.py", "FastAPI app · lifespan · CORS · StaticFiles",
            "App factory. Registers all routers. On startup calls init_db() to create tables and FTS5 triggers. Mounts React SPA as static fallback on '/'.")

        %% ── Routers ──────────────────────────────────────────────────────────
        Component(r_articles, "articles router", "FastAPI APIRouter  /api/v1/articles/*",
            "Full CRUD for articles. Delegates create/update/delete to article_service. Handles bookmark toggle, file download proxy, and raw text endpoint.")

        Component(r_notes, "notes router", "FastAPI APIRouter  /api/v1/articles/{id}/notes/*",
            "CRUD for notes attached to an article. Validates article existence before each operation.")

        Component(r_shares, "shares router", "FastAPI APIRouter  /api/v1/articles/{id}/share  /public/{token}*",
            "Creates/renews/revokes 30-minute share tokens (secrets.token_urlsafe). Serves an unauthenticated server-rendered HTML page with live JS countdown for recipients. Serves raw file at /public/{token}/file.")

        Component(r_search, "search router", "FastAPI APIRouter  /api/v1/search  /api/v1/keywords",
            "Delegates FTS5 full-text search and keyword prefix suggestions to search_service.")

        Component(r_tags, "tags router", "FastAPI APIRouter  /api/v1/tags/*",
            "Tag CRUD. Deleting a tag cascades via article_tags junction table.")

        %% ── Services ─────────────────────────────────────────────────────────
        Component(svc_article, "article_service", "Pure async functions",
            "Orchestrates article creation flows: write (text/markdown), file upload (pdf/docx/html/md/txt/video/audio), web link. Also handles update and delete. Calls parsers, storage, and DB.")

        Component(svc_search, "search_service", "Pure async functions",
            "Executes FTS5 MATCH queries with Porter stemming. Extracts highlighted snippets via SQLite snippet() function. Returns keyword prefix matches from FTS vocab shadow table.")

        %% ── Parsers ──────────────────────────────────────────────────────────
        Component(p_pdf, "pdf parser", "PyMuPDF (fitz)",
            "Extracts plain text from all pages of a PDF. Returns ParseResult with title_hint from filename, plain_text, summary (first 500 chars), word_count.")

        Component(p_docx, "docx parser", "python-docx",
            "Iterates paragraphs of a DOCX file. Returns ParseResult with title from first non-empty paragraph, concatenated body text.")

        Component(p_html, "html parser", "BeautifulSoup4 + lxml",
            "Parses uploaded HTML files. Extracts <title> tag and visible body text stripping scripts/styles. Returns ParseResult.")

        Component(p_markdown, "markdown parser", "regex + stdlib",
            "Detects first H1 heading as title_hint. Returns full raw text as plain_text for FTS indexing. Also handles plain .txt files.")

        Component(p_link, "link parser", "httpx async + BeautifulSoup4",
            "Fetches URL asynchronously with a browser User-Agent. Extracts <title>, meta description, and visible body text for FTS indexing and summary generation.")

        %% ── Storage ──────────────────────────────────────────────────────────
        Component(st_local, "LocalStorageBackend", "aiofiles  ./data/files/",
            "Stores blobs under DATA_DIR. Files are served via the /api/files/{key:path}/download proxy endpoint.")

        Component(st_s3, "S3StorageBackend", "boto3",
            "Stores blobs in the configured S3 bucket. Generates presigned GET URLs for download links.")

        %% ── Config / Deps / Models ───────────────────────────────────────────
        Component(config, "config.py", "pydantic-settings BaseSettings",
            "Reads env / .env file. Exposes: STORAGE_BACKEND, DATA_DIR, DB_PATH, S3_BUCKET, MAX_UPLOAD_MB, CORS_ORIGINS, PUBLIC_BASE_URL, SHARE_DURATION_MINUTES.")

        Component(deps, "deps.py", "FastAPI Depends",
            "get_db() yields an async SQLAlchemy session. get_storage() factory returns LocalStorageBackend or S3StorageBackend based on STORAGE_BACKEND setting.")

        Component(database, "database.py", "SQLAlchemy async engine + DDL",
            "Creates async engine. init_db() runs CREATE TABLE IF NOT EXISTS for all models, creates the articles_fts FTS5 virtual table (Porter stemmer, content='articles'), and installs three DDL triggers (after insert/update/delete) to keep FTS in sync.")
    }

    %% Relationships ────────────────────────────────────────────────────────────
    Rel(user, main, "HTTP requests", "FastAPI routing")
    Rel(recipient, r_shares, "GET /public/{token}", "HTTP — no auth")

    Rel(main, r_articles, "routes /api/v1/articles/*")
    Rel(main, r_notes, "routes /api/v1/articles/{id}/notes/*")
    Rel(main, r_shares, "routes /api/v1/articles/{id}/share and /public/*")
    Rel(main, r_search, "routes /api/v1/search and /api/v1/keywords")
    Rel(main, r_tags, "routes /api/v1/tags/*")
    Rel(main, database, "calls init_db() on startup")

    Rel(r_articles, svc_article, "delegates create/update/delete")
    Rel(r_articles, deps, "injects db session + storage")
    Rel(r_notes, deps, "injects db session")
    Rel(r_shares, deps, "injects db session + storage")
    Rel(r_search, svc_search, "delegates FTS queries")
    Rel(r_tags, deps, "injects db session")

    Rel(svc_article, p_pdf, "parses .pdf uploads")
    Rel(svc_article, p_docx, "parses .docx uploads")
    Rel(svc_article, p_html, "parses .html uploads")
    Rel(svc_article, p_markdown, "parses .md / .txt uploads and write articles")
    Rel(svc_article, p_link, "fetches and parses web links")
    Rel(svc_article, storage, "uploads / deletes blobs")
    Rel(svc_article, db, "persists Article rows")

    Rel(svc_search, db, "executes FTS5 MATCH queries")

    Rel(deps, st_local, "returns (local mode)")
    Rel(deps, st_s3, "returns (s3 mode)")
    Rel(deps, db, "yields AsyncSession")
    Rel(deps, config, "reads STORAGE_BACKEND")

    Rel(st_local, storage, "implements StorageBackend protocol")
    Rel(st_s3, storage, "implements StorageBackend protocol")

    Rel(database, db, "creates schema + FTS5 triggers")

    UpdateLayoutConfig($c4ShapeInRow="4", $c4BoundaryInRow="1")
```

## Component File Map

```
backend/src/cortex/
├── main.py                    ← main (app factory)
├── config.py                  ← config
├── database.py                ← database
├── models.py                  ← SQLAlchemy ORM models
├── schemas.py                 ← Pydantic request/response schemas
├── deps.py                    ← deps (get_db, get_storage)
├── routers/
│   ├── articles.py            ← r_articles
│   ├── notes.py               ← r_notes
│   ├── shares.py              ← r_shares
│   ├── search.py              ← r_search
│   └── tags.py                ← r_tags
├── services/
│   ├── article_service.py     ← svc_article
│   └── search_service.py      ← svc_search
├── parsers/
│   ├── base.py                ← ParseResult dataclass
│   ├── pdf.py                 ← p_pdf
│   ├── docx.py                ← p_docx
│   ├── html.py                ← p_html
│   ├── markdown.py            ← p_markdown (also handles .txt)
│   └── link.py                ← p_link
└── storage/
    ├── base.py                ← StorageBackend Protocol
    ├── local.py               ← st_local
    └── s3.py                  ← st_s3
```
