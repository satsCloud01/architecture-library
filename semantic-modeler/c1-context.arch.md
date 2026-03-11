---
name: "C1 – System Context"
project: "Semantic Modeler"
project_slug: "semantic-modeler"
project_url: "https://semantic-modeler.satszone.link"
github: ""
category: "data-analytics"
type: "c4-context"
icon: "🕸️"
tags: [Streamlit, NetworkX, RDFLib]
---

# Semantic Modeler — Architecture Overview

## System Summary

The Semantic Modeler is a Streamlit multi-page application that ingests raw data assets
(databases, files, documents, media, semi-structured files), uses Claude AI to suggest
semantic entity mappings for every field, and lets users build, govern, and export a
standards-compliant ontology — all backed by a local SQLite database.

---

## Layer Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  📺  Streamlit UI   (pages/)                                                 │
│                                                                              │
│  1_Ingest  ──►  2_Enrich  ──►  3_Graph  ──►  4_Export                       │
│                    │                                                         │
│              5_Domain_Model   6_Glossary   7_Lineage   8_Registry   9_Docs  │
└──────────────┬───────────────────────────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────────────────────────────────────────┐
│  📥  Ingestion Layer   (core/ingestion/)                                     │
│                                                                              │
│  sql_extractor.py         → SQLAlchemy  (SQL databases)                      │
│  file_extractor.py        → pandas      (CSV, Excel, JSON)                   │
│  api_extractor.py         → requests    (REST API payloads)                  │
│  unstructured_extractor.py → pypdf · python-docx · Pillow · mutagen         │
│                              (PDF, DOCX, TXT/MD/LOG, images, audio, video)   │
│  xml_extractor.py         → ElementTree + PyYAML  (XML, YAML)                │
└──────────────┬───────────────────────────────────────────────────────────────┘
               │  produces DataAsset objects
┌──────────────▼───────────────────────────────────────────────────────────────┐
│  🧠  Semantic Layer   (core/semantic/)                                       │
│                                                                              │
│  model.py      → 6 dataclasses (see Data Model section)                      │
│  enricher.py   → Anthropic SDK · batch prompt · JSON parse                   │
│  graph.py      → NetworkX DiGraph builder · Pyvis HTML renderer              │
│                  SOURCE_TYPE_COLORS / SOURCE_TYPE_ICONS                      │
└──────────────┬───────────────────────────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────────────────────────────────────────┐
│  💾  Storage Layer   (core/storage/)                                         │
│                                                                              │
│  repository.py  → sqlite3  →  semantic_modeler.db  (6 tables)                │
└──────────────────────────────────────────────────────────────────────────────┘
               │  external calls
┌──────────────▼───────────────────────────────────────────────────────────────┐
│  🌐  External Services                                                       │
│                                                                              │
│  Anthropic Claude API  (claude-sonnet-4-6)  — semantic enrichment            │
│  Vis.js CDN  (via Pyvis)                    — interactive graph rendering    │
│  Mermaid CDN                                — docs page flow diagrams        │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Pages Reference

| # | File | Icon | Purpose |
|---|------|------|---------|
| 1 | `1_Ingest.py` | 📥 | Connect databases, upload files; extract schema + content metadata into `DataAsset` |
| 2 | `2_Enrich.py` | 🧠 | AI-powered field→entity mapping via Claude; human approve / reject / edit workflow |
| 3 | `3_Graph.py` | 🕸️ | Interactive semantic graph of assets, entities, and relationships; entity + relationship CRUD |
| 4 | `4_Export.py` | 📤 | Export ontology as JSON-LD, YAML, RDF Turtle, or plain JSON |
| 5 | `5_Domain_Model.py` | 🏛️ | Manage business domain hierarchy (parent/child); entity taxonomy browser |
| 6 | `6_Glossary.py` | 📖 | Canonical business term definitions; link to entities; CSV import/export; coverage metrics |
| 7 | `7_Lineage.py` | 🔍 | Source-to-domain lineage graph; field traceability table; impact analysis |
| 8 | `8_Registry.py` | 📋 | Search and discover approved models; publish/tag entities; source-type filter |
| 9 | `9_Docs.py` | 📚 | Interactive documentation with Mermaid diagrams |

---

## Core Modules

### Ingestion  —  `core/ingestion/`

| Module | Library | Handled formats |
|--------|---------|----------------|
| `sql_extractor.py` | SQLAlchemy | PostgreSQL, MySQL, SQLite, SQL Server, Oracle |
| `file_extractor.py` | pandas, openpyxl | CSV, Excel (.xlsx/.xls), JSON |
| `api_extractor.py` | requests | REST API response payloads (JSON) |
| `unstructured_extractor.py` | pypdf, python-docx, Pillow, mutagen | PDF, DOCX, TXT/MD/LOG, JPEG/PNG/TIFF/WebP/BMP/GIF, MP3/FLAC/WAV/M4A/AAC, MP4/MKV/MOV/AVI/WebM |
| `xml_extractor.py` | stdlib ElementTree, PyYAML | XML (with `@attr` notation), YAML/YML |

All extractors return a `DataAsset` object with a `fields` list where each item is:
```python
{
    "name":          str,          # field / column / element name
    "dtype":         str,          # int | float | string | boolean | datetime | text | …
    "nullable":      bool,
    "sample_values": List[Any],    # up to 5 representative values
}
```

### Semantic  —  `core/semantic/`

| Module | Responsibility |
|--------|---------------|
| `model.py` | Six dataclasses: `DataAsset`, `SemanticMapping`, `SemanticEntity`, `Relationship`, `Domain`, `GlossaryTerm` |
| `enricher.py` | `SOURCE_TYPE_GUIDANCE` dict, `FIELD_PROMPT` template, `_call_claude()`, `enrich_asset()` |
| `graph.py` | `build_graph()` → NetworkX DiGraph, `graph_to_pyvis_html()` → HTML string, `SOURCE_TYPE_COLORS`, `SOURCE_TYPE_ICONS` |

### Storage  —  `core/storage/`

`repository.py` is a thin SQLite wrapper using only the Python standard library (`sqlite3`).
Schema is created in `_init_db()` on first run; backward-compatible columns are added in
`_migrate_db()` via `ALTER TABLE ADD COLUMN` wrapped in try/except.

---

## Data Model

```
DataAsset ──has──► fields[]
    │
    └──mapped-to──► SemanticMapping  ──references──► SemanticEntity
                        (status: pending / approved / rejected)            │
                                                                           │
                    Relationship ──links──► SemanticEntity ◄──────────────┘

SemanticEntity ──belongs-to──► Domain (optional)
GlossaryTerm   ──linked-to──► SemanticEntity (optional)
Domain         ──child-of──► Domain (parent_domain, optional)
```

### Six SQLite Tables

| Table | PK | Key constraint | Notes |
|-------|----|---------------|-------|
| `data_assets` | UUID | — | `fields` stored as JSON text |
| `semantic_entities` | UUID | `name` UNIQUE | `published`, `tags` added via migration |
| `semantic_mappings` | UUID | `asset_id` index | Cascade-deleted with asset |
| `relationships` | UUID | — | Referenced by entity name (not FK) |
| `domains` | UUID | `name` UNIQUE | Self-referential via `parent_domain` |
| `glossary_terms` | UUID | — | Optional `entity_name` soft-link |

---

## Enrichment Flow (Summary)

```
1. User selects DataAsset  (pages/2_Enrich.py)
2. enrich_asset() called   (core/semantic/enricher.py)
3. Fields batched → 20/call
4. Per batch: SOURCE_TYPE_GUIDANCE + FIELD_PROMPT → Claude API
5. JSON response parsed → SemanticMapping[] + Relationship[]
6. All saved as status="pending"
7. Human review: approve → SemanticEntity upserted, status="approved"
                 reject  → status="rejected"
8. Approved entities flow to Graph / Lineage / Registry
```

See [enrichment_flow.md](enrichment_flow.md) for the detailed step-by-step reference.

---

## Supported Source Types

| Category | Types | `source_type` value |
|----------|-------|---------------------|
| Structured | SQL Database | `sql` |
| Structured | CSV | `csv` |
| Structured | Excel | `excel` |
| Structured | JSON | `json` |
| Structured | REST API | `api` |
| Document | PDF | `pdf` |
| Document | Word Document | `docx` |
| Document | Plain text / Markdown / Log | `text` |
| Media | JPEG, PNG, TIFF, WebP, BMP, GIF | `image` |
| Media | MP3, FLAC, WAV, M4A, AAC | `audio` |
| Media | MP4, MKV, MOV, AVI, WebM | `video` |
| Semi-structured | XML | `xml` |
| Semi-structured | YAML / YML | `yaml` |

Each source type has a dedicated entry in `SOURCE_TYPE_GUIDANCE` (enricher),
`SOURCE_TYPE_COLORS` (graph node colour), and `SOURCE_TYPE_ICONS` (UI badges).

---

## Configuration

| Setting | Default | Environment variable |
|---------|---------|---------------------|
| Claude model | `claude-sonnet-4-6` | — (edit `config.py`) |
| Anthropic API key | `""` | `ANTHROPIC_API_KEY` |
| Database path | `semantic_modeler.db` | `SEMANTIC_DB_PATH` |
| Batch size | `20` fields | — (edit `enricher.py`) |

---

## Technology Stack

| Library | Version pin | Role |
|---------|-------------|------|
| Python | 3.12 | Runtime |
| Streamlit | latest | Web UI |
| anthropic | latest | Claude API client |
| networkx | latest | Graph data structure |
| pyvis | latest | Graph HTML renderer |
| pandas | latest | Tabular display + CSV |
| pypdf | ≥ 4.0 | PDF extraction |
| python-docx | ≥ 1.1 | Word extraction |
| Pillow | ≥ 10.0 | Image + EXIF |
| mutagen | ≥ 1.47 | Audio/video metadata |
| PyYAML | latest | YAML parsing |
| rdflib | latest | RDF/Turtle export |
| pytest | ≥ 7.0 | Test runner |

---

## Testing

```
tests/
  test_unstructured_extractor.py   32 tests — PDF/DOCX/TXT/image/audio extractors
  test_xml_extractor.py            38 tests — XML/YAML extractors + sample file integration

Run: python -m pytest tests/ -v   (from project root)
All 70 tests pass.
```

Sample data files for manual testing:
```
sample_data/
  unstructured/
    patient_record.txt       medical record with labelled fields
    vendor_contract.txt      contract with dates, amounts, emails
    product_image.jpg        JPEG with Nikon Z9 EXIF metadata
    employee_badge.jpg       JPEG with Sony EXIF metadata
    invoice_scan.jpg         JPEG with HP Scanner EXIF
    sales_dashboard.png      PNG chart (no EXIF)
  semi_structured/
    product_catalog.xml      multi-level XML product catalog
    employees.yaml           employee + department YAML
```
