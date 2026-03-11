---
name: "C1 – System Context"
project: "DataAnalyzer Buddy"
project_slug: "dataanalyzer-buddy"
project_url: "https://dataanalyzer.satszone.link"
github: "https://github.com/satsCloud01/DataAnalyzerBuddy"
category: "data-analytics"
type: "c4-context"
icon: "📊"
tags: [FastAPI, React, Claude API, Plotly, pandas]
---

# Architecture — DataAnalyzerBuddy

## Overview

DataAnalyzerBuddy is a single-page React application backed by a FastAPI Python service. Datasets live entirely in server memory for the lifetime of a process; there is no database. The Anthropic Claude API is called on-demand with a per-request key supplied by the user.

---

## System Diagram

```
Browser (React SPA)
│
│  HTTP / Vite proxy (dev)
│  Nginx reverse proxy (prod)
│
├── GET  /                     → Landing page
├── GET  /upload               → Upload page
├── GET  /explore/:id          → Explore + profile page
├── GET  /query/:id            → Ask AI page
├── GET  /visualize/:id        → Chart builder
├── GET  /trends/:id           → Trend forecaster
├── GET  /patterns/:id         → Pattern & outlier page
├── GET  /clean/:id            → Data cleaning pipeline
└── GET  /export/:id           → Download & PDF export

            │  REST API  /api/*
            ▼
        FastAPI (port 8000)
            │
    ┌───────┼───────────────────────┐
    │       │                       │
  Store   Services              Routers (8)
  (RAM)   │                         │
    │     ├── profiler.py     upload.py
    │     ├── forecaster.py   profile.py
    │     ├── pattern_svc.py  query.py
    │     └── claude_svc.py ──visualize.py
    │              │          trends.py
    │         Anthropic API   patterns.py
    │         (external)      clean.py
    └─────────────────────────export.py
```

---

## Component Breakdown

### Frontend

| Component | Responsibility |
|---|---|
| `App.tsx` | Route definitions, `ApiKeyProvider` wrapper |
| `Sidebar.tsx` | Dataset switcher, nav links, API-key status; **refetches dataset list on every route change** to detect server restarts |
| `ApiKeyContext.tsx` | Holds Anthropic key in React `useState` only; `requestKey()` returns a `Promise` that resolves with key or rejects on cancel |
| `api.ts` | Typed fetch wrapper; sends `X-Api-Key` header on AI calls |
| `PlotlyChart.tsx` | Renders any Plotly JSON spec received from backend |
| Pages (9) | Each page owns its own loading/error state; AI features call `requestKey()` on demand |

### Backend

| Module | Responsibility |
|---|---|
| `main.py` | FastAPI app, CORS (`localhost:5173`), mounts all 8 routers |
| `store/memory.py` | Global `Dict[str, DataFrame]` + `Dict[str, DatasetMeta]` + query history. Cleared on process restart. |
| `services/profiler.py` | Per-column statistics (dtype detection, nulls, quartiles, outliers via IQR, top-values, correlation matrix) |
| `services/claude_service.py` | `_get_client()` uses per-request key or env var. `ask_question()` and `generate_profile_narrative()` build Markdown context from the DataFrame and call Claude Sonnet. |
| `services/forecaster.py` | `detect_datetime_columns()` + `forecast()` supporting linear (polyfit deg-1), polynomial (deg-2), and ExponentialSmoothing |
| `services/pattern_service.py` | `detect_outliers()` (IQR + Z-score), `cluster()` (K-Means with elbow auto-k), `run_patterns()` |
| `routers/upload.py` | Multipart file upload; xlsx→all sheets, csv, json, parquet; stores DataFrames in memory |
| `routers/profile.py` | Dataset statistics + optional Claude narrative |
| `routers/query.py` | NL question → Claude → structured `{answer, chart_json, insights}`; maintains per-dataset history |
| `routers/visualize.py` | 7 Plotly chart types built server-side, returned as JSON |
| `routers/trends.py` | Time-series forecast using Forecaster service |
| `routers/patterns.py` | Delegates to PatternService |
| `routers/clean.py` | 8 chainable cleaning operations; optionally saves result as new dataset |
| `routers/export.py` | CSV / Excel / JSON streaming downloads; fpdf2 PDF report with AI narrative |

---

## Data Flow — Upload → Query

```
1. User drags file onto Upload page
2. POST /api/upload (multipart)
   → parse with pandas (csv / openpyxl / etc.)
   → generate 8-char UUID dataset_id
   → store DataFrame in memory.datasets[id]
   → store DatasetMeta in memory.metadata[id]
   → return {uploaded: [{dataset_id, name, rows, cols}]}

3. User navigates to /explore/:id
   → GET /api/datasets/:id/profile?with_narrative=false
      → profiler.profile_dataframe(df)  → column stats
      → (optional) claude_service.generate_profile_narrative(df, profile)
   → GET /api/datasets/:id/preview?rows=50

4. User clicks "Ask AI"
   → requestKey() modal if no key stored
   → POST /api/datasets/:id/query  {question}  X-Api-Key header
      → claude_service.ask_question(df, question, history, profile, api_key)
         → _build_context(df)  → Markdown schema + 30 sample rows
         → Anthropic messages.create()
         → parse <chart>JSON</chart> and <insights>text</insights>
      → append to query_history[id]
      → return {answer, chart_json, insights}
```

---

## API Key Security Model

```
Browser RAM (useState)
    │
    │  X-Api-Key: sk-ant-...  (HTTP header, per-request)
    ▼
FastAPI
    │
    ├─ x_api_key = Header(default=None)
    │
    └─ claude_service._get_client(api_key)
           ├── api_key provided → Anthropic(api_key=api_key)
           └── not provided     → os.getenv("ANTHROPIC_API_KEY")
                                     └── empty → ValueError → HTTP 401
```

Keys are **never** logged, stored in files, or persisted beyond the HTTP request.

---

## In-Memory Store Trade-offs

| Pro | Con |
|---|---|
| Zero configuration | Data lost on server restart |
| Fast (no DB round-trips) | Not multi-process safe |
| Simple to reason about | Not suitable for concurrent users at scale |

**Mitigation:** The sidebar re-fetches `GET /api/datasets` on every route change and shows a "Session expired" amber warning when the active dataset is missing, with a direct link to re-upload.

---

## Production Architecture

```
Internet
  │
  ▼
Route 53 / DNS  (dataanalyzer.satszone.link → EC2 IP)
  │
  ▼
EC2 t2.micro (Ubuntu 22.04)
  │
  ├── Nginx (port 80/443)
  │     ├── / → serve /var/www/dataanalyzer/frontend/dist/ (static)
  │     └── /api/ → proxy_pass http://127.0.0.1:8000
  │
  ├── Certbot (Let's Encrypt, auto-renew)
  │
  └── systemd service: dataanalyzer
        └── uvicorn analyst.main:app --host 127.0.0.1 --port 8000 --workers 2
```
