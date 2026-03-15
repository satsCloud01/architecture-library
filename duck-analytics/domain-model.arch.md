---
name: "Domain Model"
project: "Duck Analytics"
project_slug: "duck-analytics"
project_url: null
github: "https://github.com/satsCloud01/duck-analytics"
category: "learning"
type: "domain-model"
icon: "🦆"
tags: [Litestar, DuckDB, Arrow, domain-model]
---

# Duck Analytics -- Domain Model

## Bounded Contexts

1. **SQL Execution** — Query authoring, execution, schema introspection, file import, history
2. **Arrow Introspection** — Column types, memory layout, buffer visualization, format conversion
3. **Performance Benchmarking** — Arrow IPC vs JSON serialization comparison across iterations
4. **Data Pipeline** — End-to-end: upload → ingest → inspect → compare
5. **Learning** — 22-lesson interactive wizard (DuckDB, Arrow, Litestar, integration)

## Database Schema (DuckDB)

### Demo Data Tables (auto-seeded)
| Table | Rows | Columns |
|-------|------|---------|
| `demo_sales` | 5,000 | id, category, region, amount, quantity, sale_date |
| `demo_customers` | 1,000 | id, name, tier, joined_date, lifetime_value |
| `demo_products` | 200 | id, name, category, price, stock |

### System Tables
| Table | Purpose |
|-------|---------|
| `_settings` | Key-value configuration store |
| `_query_history` | SQL execution audit log |
| `_bench_results` | Benchmark result history |
