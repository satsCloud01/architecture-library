---
name: "C2 – Enrichment Flow"
project: "Semantic Modeler"
project_slug: "semantic-modeler"
project_url: "https://semantic-modeler.satszone.link"
github: ""
category: "data-analytics"
type: "c4-container"
icon: "🕸️"
tags: [Claude API, RDFLib]
---

# AI Enrichment Flow — Reference

## Overview

The **Enrich** page (`pages/2_Enrich.py`) uses **Claude** (Anthropic's AI) to
automatically suggest semantic entity mappings for every field in an ingested
data asset.  Users review each suggestion — approving, editing, or rejecting it —
before it enters the live semantic model.

---

## End-to-End Flow

```
User selects DataAsset
        │
        ▼
Click "✨ Generate AI Suggestions"
        │
        ├─── No API key? ──► Show warning, stop
        │
        ▼
enrich_asset(asset, api_key, model)          [core/semantic/enricher.py]
        │
        ▼
Split asset.fields into batches of 20        [BATCH_SIZE = 20]
        │
        ▼  ◄──────────────────────────────── repeat for each batch
_call_claude(client, model, asset_name, source_type, fields_subset)
        │
        ├─► Lookup SOURCE_TYPE_GUIDANCE[source_type]
        │     Format-specific context string injected into the prompt
        │
        ├─► Format fields_text:
        │     "  - column_name (dtype) | nullable=True | samples: v1, v2, v3"
        │
        ├─► Render FIELD_PROMPT template
        │     (asset_name, source_type, source_guidance, fields_text)
        │
        └─► anthropic.Anthropic.messages.create(
                model      = "claude-sonnet-4-6",
                system     = SYSTEM_PROMPT,
                messages   = [{"role": "user", "content": prompt}],
                max_tokens = 8192
            )
                │
                ▼
        Parse JSON response
        (_strip_code_fence → json.loads)
                │
                ├─► "mappings": [...]
                │       └─► SemanticMapping(status="pending") × N
                │
                └─► "relationships": [...]
                        └─► Relationship() × M  (deduped by src+tgt+type)
        │
        ▼
repo.save_mapping(m)        [core/storage/repository.py]
repo.save_relationship(r)
        │
        ▼
UI renders pending review cards
        │
        ├─── ✅ Approve ──► SemanticEntity upserted, mapping.status = "approved"
        │                    Entity visible in Graph / Lineage / Registry
        │
        ├─── ❌ Reject  ──► mapping.status = "rejected"
        │
        └─── ✏️ Edit    ──► Manual mapping form, confidence = 1.0
```

---

## How Claude Generates Suggestions

Claude does **not** query any external database, ontology server, or URL during
inference.  All suggestions come from **parametric knowledge** — patterns encoded
in the model's weights during pre-training on a large corpus of text.

### Knowledge Sources

| Domain | What Claude absorbed | How it helps |
|--------|---------------------|-------------|
| **Enterprise schemas** | ERP/CRM/analytics patterns (Salesforce, SAP, Oracle), SaaS API designs, data warehouse conventions | Recognises `account_id`, `opportunity_stage`, `mrr_amount` immediately |
| **Semantic web standards** | OWL 2, RDF/RDFS, Dublin Core, schema.org, FIBO, HL7 FHIR | Maps fields to well-known ontology classes |
| **DB naming conventions** | `_id`/`_key` = PK/FK; `_at`/`_date` = timestamp; `is_`/`has_` = boolean; `_count`/`_amount` = measure | Infers `entity_type` from suffix patterns |
| **Industry verticals** | Healthcare (FHIR, ICD), finance (XBRL, SWIFT), media (IPTC, ID3), e-commerce, logistics | Assigns correct `domain` for specialist fields |
| **API/schema specs** | OpenAPI 3, JSON Schema, Protobuf, Avro, Parquet naming | Handles dot-notation nested fields accurately |
| **Natural language** | Abbreviated English field names | Expands `cust_addr_st` → CustomerAddressState |

### Signals Per Field (Highest → Lowest Influence)

1. **Field name** — primary signal.
   `customer_email` → `EmailAddress / Customer` (near-certain).
   `code` → ambiguous.

2. **Sample values** — resolves ambiguity.
   `['alice@corp.com', 'bob@firm.io']` confirms email even if the field is named `addr1`.

3. **Data type** — narrows entity type:
   `datetime` → `Date`, `boolean` → `Flag`, `float` → `Measure`, long `text` → `DocumentContent`.

4. **Source-type guidance** — the `SOURCE_TYPE_GUIDANCE` string tells Claude
   what format conventions mean (e.g., `exif_Make` = camera, `tag_artist` = audio metadata).

5. **Batch co-occurrence** — seeing `order_id`, `customer_id`, `product_id`,
   `total_amount` together confirms the `Order` domain and reinforces each mapping.

---

## Confidence Score

> The confidence value (0.0 – 1.0) is **self-reported by Claude** as part of its JSON
> response.  It is **not** a statistical probability computed by the application.
> Claude estimates how certain it is based on the signals available.

### Factors that raise confidence

| Factor | Example |
|--------|---------|
| Unambiguous field name | `customer_id` → 0.97 |
| Sample values confirm the entity | `exif_Make` with samples `['Nikon']` → 0.93 |
| Data type matches entity type | `boolean` for `is_active` → `Flag` 0.98 |
| Source-type guidance aligns | SQL `_id` + "→ Identifier" guidance → 0.95 |
| Co-occurring fields confirm domain | `order_id` + `order_date` + `order_total` together |

### Factors that lower confidence

| Factor | Example |
|--------|---------|
| Generic / ambiguous name | `code`, `value`, `type`, `data` → 0.40–0.60 |
| Missing sample values | Claude relies on name alone |
| Multiple plausible interpretations | `date` → BirthDate? OrderDate? CreatedDate? |
| Very short abbreviation without context | `dt`, `amt`, `id` alone |

### Practical thresholds

| Range | Recommended action |
|-------|--------------------|
| 0.90 – 1.00 | Standard approval candidate |
| 0.70 – 0.89 | Review description before approving |
| 0.50 – 0.69 | Double-check entity name and type |
| < 0.50 | Treat as a hint only; consider manual override |

Manual mappings (added via the UI form) always receive `confidence = 1.0`.

---

## Prompt Architecture

### System Prompt

```
You are a semantic data modeling expert.
Analyze data asset schemas and suggest semantic entity mappings.
Respond ONLY with valid JSON — no markdown, no commentary.
```

### User Prompt Template (`FIELD_PROMPT`)

```
Asset Name   : {asset_name}
Source Type  : {source_type}

Source-Type Context
-------------------
{source_guidance}

Fields
------
{fields_text}

For each field suggest:
  entity_name, entity_type, domain, description, mapping_type, confidence

Also suggest entity-to-entity relationships.

Respond ONLY with:
{ "mappings": [...], "relationships": [...] }
```

### Source-Type Guidance (all 11 entries)

| Source type | Key guidance |
|-------------|-------------|
| `csv` / `excel` | Tabular — infer from column name + samples |
| `sql` | SQL dtypes; `_id` / `_key` → Identifier |
| `json` / `api` | Dot-notation keys; REST API context |
| `pdf` | `raw_content` → DocumentContent; label:value extraction |
| `docx` | Similar to pdf; paragraph labels + table cells |
| `text` | Log/Markdown/plain; regex fields: email, phone, date, amount |
| `image` | `exif_Make/Model` → CameraDevice; `vision_*` → AI-detected content |
| `audio` | `tag_title/artist/album` → Track/Artist/Album; `duration_seconds` → Measure |
| `video` | `tag_tv_show/season/episode` → TVSeries; `VideoBitrate/Duration` |
| `xml` | Dot-notation + `@attr` attributes; `@id` → Identifier |
| `yaml` | Dot-notation keys; `bool` → Flag; config vs data |

---

## Batching

```
Total fields N
      │
      ▼
ceil(N / 20) Claude calls   [BATCH_SIZE = 20]
      │
      ├─► Batch 1  → SemanticMapping[0..19]  + Relationship subset
      ├─► Batch 2  → SemanticMapping[20..39] + Relationship subset
      └─► ...
      │
      ▼
Merge all mappings
Deduplicate relationships by (source, target, type)
```

Batching keeps prompts comfortably below context limits while letting Claude
reason about co-occurrence within each batch.

---

## Data Model Lifecycle

```
DataAsset.fields[]
    │  (populated at Ingest)
    ▼
SemanticMapping  status="pending"    ← AI
    │
    ├── approved ──► SemanticEntity upserted  ──► Graph / Lineage / Registry
    └── rejected ──► soft-deleted from queue

Relationship ──► links SemanticEntity ↔ SemanticEntity
```

---

## Model Card

| Attribute | Value |
|-----------|-------|
| Provider | Anthropic |
| Model ID | `claude-sonnet-4-6` |
| API method | `anthropic.Anthropic.messages.create()` — synchronous, single-turn |
| Context window | 200 K tokens |
| Max output tokens | 8 192 (configured in `_call_claude()`) |
| No tools / function-calling | Claude returns raw JSON, parsed client-side |
| No external lookups | Purely parametric — no web or DB calls during inference |

---

## Configuration

| Setting | Default | Override |
|---------|---------|----------|
| Claude model | `claude-sonnet-4-6` | `config.py → CLAUDE_MODEL` |
| API key | `""` | `ANTHROPIC_API_KEY` env var or sidebar |
| Batch size | `20` fields | `enricher.py → BATCH_SIZE` |
| Max tokens | `8 192` | `enricher.py → _call_claude()` |

---

## External References

- [Anthropic Claude API — Messages](https://docs.anthropic.com/en/api/messages)
- [Anthropic Python SDK](https://github.com/anthropics/anthropic-sdk-python)
- [Claude model overview](https://docs.anthropic.com/en/docs/about-claude/models/overview)
- [Anthropic prompt engineering guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [W3C OWL 2](https://www.w3.org/TR/owl2-overview/)
- [schema.org](https://schema.org)
- [FIBO Financial Ontology](https://spec.edmcouncil.org/fibo/)
