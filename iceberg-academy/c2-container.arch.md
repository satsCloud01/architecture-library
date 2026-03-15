---
name: "C2 Container Diagram"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "c4-container"
icon: "🧊"
tags: [FastAPI, React, SQLite, Vite, Tailwind, aiosqlite]
---

# Iceberg Academy -- C2: Container Diagram

## Containers

| Container | Technology | Purpose |
|-----------|-----------|---------|
| React SPA | React 18, Vite, Tailwind CSS, React Router v6 | Interactive UI with 12 pages: learning modules, quizzes, labs, playground, glossary, architecture diagrams, papers, and patterns |
| FastAPI Backend | Python 3.12, FastAPI, aiosqlite, Pydantic v2 | REST API serving curriculum content, tracking progress, managing quizzes and playground snippets |
| SQLite Database | SQLite 3 (WAL mode, foreign keys enabled) | Persistent storage for modules, lessons, quizzes, progress, glossary, labs, playground snippets, and chat messages |

## Communication

- **Frontend to Backend:** HTTP REST (`/api/*`) with JSON request/response bodies
- **Backend to SQLite:** aiosqlite async connection per request (WAL journal mode)
- **Frontend dev proxy:** Vite proxy forwards `/api` requests to the backend

## Container Diagram

```
┌──────────────────────────────────────────────────┐
│                   Browser                         │
│  ┌────────────────────────────────────────────┐  │
│  │           React SPA (Vite)                 │  │
│  │  React 18 + Tailwind + React Router v6     │  │
│  │  12 pages, inline fetch() API calls        │  │
│  └────────────────┬───────────────────────────┘  │
└───────────────────┼──────────────────────────────┘
                    │ HTTP /api/*
                    ▼
┌──────────────────────────────────────────────────┐
│           FastAPI Backend (Python 3.12)           │
│  9 routers, 22 Pydantic models, aiosqlite        │
│  In-memory: papers (17), patterns (10),           │
│             diagrams (3), playground examples (13) │
└────────────────────┬─────────────────────────────┘
                     │ aiosqlite (WAL + FK)
                     ▼
┌──────────────────────────────────────────────────┐
│              SQLite Database                      │
│  9 tables: modules, lessons, quiz_questions,      │
│  glossary_terms, user_progress, quiz_attempts,    │
│  playground_snippets, chat_messages, labs          │
└──────────────────────────────────────────────────┘
```
