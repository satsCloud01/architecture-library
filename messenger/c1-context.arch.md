---
name: "C1 – System Context"
project: "Lightweight Messenger"
project_slug: "messenger"
project_url: "https://messenger.satszone.link"
github: "https://github.com/satsCloud01/lightweight-messenger"
category: "productivity"
type: "c4-context"
icon: "💬"
tags: [FastAPI, WebSockets, Pydantic, itsdangerous]
---

# C1 — System Context Diagram

Lightweight Messenger is a real-time chat application built with FastAPI and WebSockets. It supports multi-user messaging, file sharing with automatic TTL expiry, and form-based authentication — all without heavy frontend frameworks.

```
┌──────────────────────────────────────────────────────────────────┐
│                       User Environment                           │
│                                                                  │
│   [User A]              [User B]              [User C]           │
│      │                     │                     │               │
│      └─────────────────────┴─────────────────────┘               │
│                            │                                     │
│                 HTTPS (Browser + WebSocket)                      │
│                            │                                     │
│                            ▼                                     │
│             ┌─────────────────────────┐                          │
│             │  Lightweight Messenger   │                          │
│             │  Real-time Chat App      │                          │
│             │  (FastAPI + WebSockets)  │                          │
│             └────────────┬────────────┘                          │
│                          │                                       │
│              ┌───────────┼───────────┐                           │
│              ▼           ▼           ▼                            │
│   ┌──────────────┐ ┌──────────┐ ┌──────────────┐               │
│   │ In-Memory    │ │ File     │ │ Session      │               │
│   │ Message Store│ │ Uploads  │ │ Store        │               │
│   │ (Python dict)│ │ (disk +  │ │ (itsdangerous│               │
│   └──────────────┘ │  TTL)    │ │  cookies)    │               │
│                     └��─────────┘ └──────────────┘               │
└──────────────────────────────────────────────────────────────────┘
```

---

## Actors

| Actor | Role |
|---|---|
| **Chat User** | Send/receive real-time messages, share files, view message history |

---

## Key Architecture Decisions

- **No database**: Messages and sessions stored in-memory (Python dicts)
- **WebSocket broadcast**: Real-time messaging via `WebSocketManager` broadcasting to all connected clients
- **File TTL**: Uploaded files automatically expire and self-clean after configurable TTL
- **Authentication**: Form-based login with CSRF-protected signed session cookies via `itsdangerous`
- **Static frontend**: Plain HTML/CSS/JS served from FastAPI static files — no build toolchain
- **No framework dependencies**: Minimal frontend, no React/Vue/Angular needed
