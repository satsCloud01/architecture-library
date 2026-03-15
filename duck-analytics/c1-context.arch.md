---
name: "C1-C4 Architecture"
project: "Duck Analytics"
project_slug: "duck-analytics"
project_url: null
github: "https://github.com/satsCloud01/duck-analytics"
category: "learning"
type: "c4-context"
icon: "🦆"
tags: [Litestar, React, DuckDB, Arrow, PyArrow]
---

# Duck Analytics -- Architecture Documentation

> Interactive educational platform for exploring DuckDB, Apache Arrow & Litestar. Built with Python 3.12 + React 18 + Vite + Tailwind CSS.

## C1: System Context

```
┌─────────────────────────────────────────────────────────┐
│                    Duck Analytics                        │
│          DuckDB + Arrow Educational Platform             │
└────────────────────────┬────────────────────────────────┘
                         │
          ┌──────────────┼──────────────────┐
          │              │                  │
    ┌─────▼─────┐  ┌────▼─────┐   ┌───────▼───────┐
    │  Browser   │  │  CSV /   │   │  Parquet      │
    │  (User)    │  │  TSV     │   │  Files        │
    └───────────┘  │  Files    │   └───────────────┘
                    └──────────┘
```

**Actors:**
- **End User** — Data engineer, analyst, or student exploring DuckDB, Apache Arrow, and Litestar through an interactive web UI.
- **File Sources** — CSV, TSV, and Parquet files uploaded via browser for ingestion into DuckDB.

**System Boundary:** Self-contained, single-node web application. No external APIs, databases, or cloud services. DuckDB runs in-process (embedded). No authentication.

## C2: Container Diagram

| Container | Technology | Purpose |
|-----------|-----------|---------|
| React SPA | React 18, Vite, Tailwind 3.4, Recharts, CodeMirror | Interactive UI with 7 pages and 22-lesson learning wizard |
| Litestar Backend | Python 3.12, Litestar 2.15+, PyArrow 18+ | REST API serving JSON and binary Arrow IPC streams |
| DuckDB | DuckDB 1.2+ (embedded, in-process) | Columnar OLAP database — zero-copy Arrow integration |

**Communication:**
- Frontend → Backend: HTTP REST (JSON bodies + multipart file uploads)
- Backend → Frontend: JSON responses + binary `application/vnd.apache.arrow.stream`
- Backend → DuckDB: In-process zero-copy Arrow via `fetch_arrow_table()`

## C3: Component Diagram

**Backend:** 5 Controllers (SQL, Arrow, Bench, Pipeline, Settings) → 3 Services (duckdb_service, arrow_service, bench_service) → database.py (thread-safe singleton DuckDB connection, auto-seed 6 tables)

**Frontend:** 7 Route Pages (Landing, Learn, SQLPlayground, ArrowExplorer, DataPipeline, PerfBench, Settings) + Sidebar + GuidedTour + api.js (14 methods)

## C4: Key Data Flows

**SQL Execution:** User → CodeMirror → POST /api/sql/execute → DuckDB → serialize → JSON response
**Arrow IPC:** POST /api/arrow/from-query → DuckDB zero-copy → Arrow IPC stream → binary HTTP response
**Pipeline:** Upload CSV/Parquet → DuckDB ingest → Arrow introspection → conversion stats
**Benchmark:** N iterations × (Arrow IPC + JSON) → speedup calculation → persist to _bench_results
