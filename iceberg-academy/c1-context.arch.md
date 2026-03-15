---
name: "C1 System Context"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "c4-context"
icon: "🧊"
tags: [FastAPI, React, SQLite, Iceberg, Arrow, Flight, FlightSQL]
---

# Iceberg Academy -- C1: System Context

> Interactive educational platform for mastering Apache Iceberg, Arrow, Flight, FlightSQL, and the REST Catalog. Built with Python 3.12 + FastAPI + React 18 + Vite + Tailwind CSS.

## System Context Diagram

```
┌─────────────────────────────────────────────────────────┐
│                   Iceberg Academy                        │
│     Apache Data Lakehouse Technologies Education         │
└────────────────────────┬────────────────────────────────┘
                         │
          ┌──────────────┼──────────────────┐
          │              │                  │
    ┌─────▼─────┐  ┌────▼──────┐   ┌───────▼───────┐
    │  Browser   │  │  arXiv /  │   │  Apache       │
    │  (Learner) │  │  Specs /  │   │  Project      │
    └───────────┘  │  Papers   │   │  Docs         │
                    └───────────┘   └───────────────┘
```

## Actors

- **Learner** — Data engineer, analyst, or student learning Apache Iceberg, Arrow, Flight, FlightSQL, and the REST Catalog through interactive lessons, quizzes, labs, and a code playground.
- **External Papers / Specs** — 17 curated research papers, specifications, and blog posts from the Apache ecosystem (arXiv, iceberg.apache.org, arrow.apache.org, Dremio, Snowflake). Referenced by URL only; no runtime fetching.

## System Boundary

Self-contained, single-node web application. No external AI APIs. No authentication. No cloud services at runtime. All educational content is pre-authored and seeded into SQLite on startup.
