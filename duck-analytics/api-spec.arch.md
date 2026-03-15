---
name: "API Specification"
project: "Duck Analytics"
project_slug: "duck-analytics"
project_url: null
github: "https://github.com/satsCloud01/duck-analytics"
category: "learning"
type: "api-spec"
icon: "🦆"
tags: [Litestar, REST, Arrow-IPC, DuckDB]
---

# Duck Analytics -- API Specification

**Framework:** Litestar 2.15+ | **Base URL:** `/api` | **Port:** 8009

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | System status + DuckDB/Arrow versions |
| POST | `/api/sql/execute` | Execute SQL query → JSON results |
| GET | `/api/sql/schemas` | List tables with column metadata |
| POST | `/api/sql/import` | Upload CSV/Parquet → create DuckDB table |
| GET | `/api/sql/history` | Query execution history |
| POST | `/api/arrow/from-query` | Execute SQL → binary Arrow IPC stream |
| GET | `/api/arrow/table/{name}/info` | Arrow schema metadata |
| GET | `/api/arrow/table/{name}/layout` | Buffer-level layout |
| GET | `/api/arrow/table/{name}/convert` | Arrow IPC vs JSON comparison |
| POST | `/api/bench/run` | Run N-iteration benchmark |
| GET | `/api/bench/results` | Benchmark history |
| POST | `/api/pipeline/upload` | Full pipeline: upload → ingest → inspect |
| GET | `/api/pipeline/tables` | List all tables |
| GET | `/api/settings/` | Get all settings |
| PUT | `/api/settings/` | Upsert a setting |

## Content Types
- `application/json` — all standard responses
- `multipart/form-data` — file uploads
- `application/vnd.apache.arrow.stream` — binary Arrow IPC
