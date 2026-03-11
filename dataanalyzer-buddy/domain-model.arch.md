---
name: "Domain Model"
project: "DataAnalyzer Buddy"
project_slug: "dataanalyzer-buddy"
project_url: "https://dataanalyzer.satszone.link"
github: "https://github.com/satsCloud01/DataAnalyzerBuddy"
category: "data-analytics"
type: "domain-model"
icon: "📊"
tags: [pandas, Pydantic, Plotly]
---

# Domain Model

## Bounded Contexts

DataAnalyzer Buddy is organised into four bounded contexts, all operating against an in-memory dataset store:

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Ingest Context   │  │ Analysis Context │  │  AI Query Ctx   │
│                 │  │                 │  │                 │
│ Upload (CSV/XLS)│──►│ Profiler        │  │ ClaudeService   │
│ DatasetMeta     │  │ Trends          │  │ QueryHistory    │
│ MemoryStore     │  │ Patterns        │  │ Insights        │
└─────────────────┘  └─────────────────┘  └─────────────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │ Output Context   │
                     │                 │
                     │ Visualize (Plotly│
                     │   JSON specs)   │
                     │ Clean (impute,  │
                     │   dedupe, etc.) │
                     │ Export (PDF,    │
                     │   Excel, MD)    │
                     └─────────────────┘
```

---

## Core Entities (In-Memory)

```
MemoryStore (Dict[str, pd.DataFrame])
  └── key: dataset_id (UUID string)
      value: pandas DataFrame (the raw uploaded data)

DatasetMeta
  ├── id (UUID string, matches MemoryStore key)
  ├── name (original filename)
  ├── row_count
  ├── column_count
  └── uploaded_at

QueryHistory (Dict[str, List[dict]])
  └── key: dataset_id
      value: list of {role, content} conversation turns

DataProfile (computed, not persisted)
  ├── columns: List[ColumnProfile]
  │     ├── name / dtype
  │     ├── null_count / unique_count
  │     ├── min / max / mean / std (numeric)
  │     └── top_values (categorical)
  ├── row_count
  └── shape

ChartSpec (Plotly JSON, transient)
  ├── data: list of trace objects
  ├── layout: dict
  └── chart_type: str

TrendResult (transient)
  ├── column
  ├── direction (up / down / stable)
  ├── slope / r_squared
  └── description

PatternResult (transient)
  ├── pattern_type (correlation | cluster | outlier | distribution)
  ├── columns involved
  ├── score / description
  └── chart_json (optional)

CleanOperation
  ├── operation (drop_nulls | fill_mean | fill_median | deduplicate | ...)
  ├── columns (optional subset)
  └── rows_affected / rows_remaining
```

---

## Data Flow

```
User uploads CSV/Excel
        │
        ▼
  parse → pd.DataFrame → MemoryStore[dataset_id]
        │
        ├──► /profile   → DataProfile (column stats, dtypes, nulls)
        ├──► /query     → Claude API → { answer, chart_json, insights }
        ├──► /visualize → Plotly ChartSpec
        ├──► /trends    → statsmodels regression → TrendResult[]
        ├──► /patterns  → scikit-learn (corr, clustering) → PatternResult[]
        ├──► /clean     → transform DataFrame in place → CleanOperation
        └──► /export    → PDF / Excel / Markdown byte stream
```

---

## Key Design Decisions

- **No database** — all state is held in a Python `dict` keyed by UUID; simplicity over durability
- **Stateless AI** — each Claude call receives the full dataset profile plus recent query history; no embeddings or vector store
- **Server-side chart generation** — Plotly JSON specs are built in Python and rendered by `react-plotly.js`, keeping the frontend thin
