---
name: "C1 – System Context"
project: "MyLegal Doctor"
project_slug: "mylegal-doctor"
project_url: "https://legal.satszone.link"
github: "https://github.com/satsCloud01/mylegal-doctor"
category: "productivity"
type: "c4-context"
icon: "⚖️"
tags: [Streamlit, Claude API, ChromaDB, sentence-transformers, SQLite]
---

# C1 — System Context Diagram

MyLegal Doctor is an AI-powered USCIS immigration assistant that answers user questions grounded in official government documentation. It scrapes USCIS pages, indexes them in a vector store, and uses Claude AI with RAG to synthesise cited answers.

```
┌──────────────────────────────────────────────────────────────────┐
│                       User Environment                           │
│                                                                  │
│   [Immigration Applicant]   [HR Professional]   [Attorney]       │
│           │                       │                │             │
│           └───────────────────────┴────────────────┘             │
│                                │                                 │
│                           HTTPS (Browser)                        │
│                                │                                 │
│                                ▼                                 │
│                 ┌─────────────────────────┐                      │
│                 │    MyLegal Doctor        │                      │
│                 │  AI Immigration Asst.    │                      │
│                 │  (Streamlit Chat UI)     │                      │
│                 └────────────┬────────────┘                      │
│                              │                                   │
│              ┌───────────────┼───────────────┐                   │
│              ▼               ▼               ▼                   │
│   ┌─────────────────┐ ┌──────────────┐ ┌──────────────┐        │
│   │ Anthropic Claude │ │  ChromaDB    │ │  USCIS.gov   │        │
│   │ API (answers)    │ │  Vector DB   │ │  (scraped)   │        │
│   └─────────────────┘ │  (embeddings)│ └──────────────┘        │
│                        └──────────────┘                          │
│                              ▲                                   │
│                     sentence-transformers                        │
│                     (embedding model)                            │
└──────────────────────────────────────────────────────────────────┘
```

---

## Actors

| Actor | Role |
|---|---|
| **Immigration Applicant** | Ask questions about visa types, green card processes, timelines |
| **HR Professional** | Research H1B/L1 sponsorship requirements for employees |
| **Immigration Attorney** | Verify USCIS policy details with source citations |

---

## External Systems

| System | Direction | Purpose |
|--------|-----------|---------|
| **Anthropic Claude API** | Outbound | Synthesises cited answers from retrieved context |
| **ChromaDB** | Local | Persistent vector store for USCIS document embeddings |
| **sentence-transformers** | Local | Generates embeddings for document chunks and queries |
| **USCIS.gov** | Scraping (offline) | Playwright scrapes JS-rendered USCIS pages for the knowledge base |
| **SQLite** | Local | Stores query history and session data |

---

## Key Architecture Decisions

- **RAG pipeline**: USCIS docs → Playwright scraper → sentence-transformer embeddings → ChromaDB → Claude synthesis
- **Source attribution**: Every answer includes links to original USCIS source pages
- **Visa coverage**: H1B, L1A/L1B, O-1A/O-1B, E-2, TN, EB-1/EB-2/EB-3 green cards
- **Agent pattern**: `MyLegalDoctorAgent` orchestrates retrieval → context building → Claude completion
- **Config-driven**: All settings (API keys, model names, chunk sizes) via `config.py`
