---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: System Context
type: c4-context
tags: [C4, system-context, learning]
---

# C1 — System Context

```
+---------------------------------------------------+
|                  Learners                          |
|          (Developers, AI Engineers,                |
|           Solution Architects, Students)           |
+----------------------------+-----------------------+
                             |
                        HTTPS (Browser)
                             |
                             v
              +------------------------------+
              |     SatsLearning Academy      |
              |   Agentic AI Educational      |
              |          Platform             |
              +------------------------------+
                             |
                             |
                    (No external dependencies)
                    100% self-contained static content
```

## Actors

| Actor | Role |
|---|---|
| **Developers** | Learn agentic AI foundations, protocols (MCP, A2A), and design patterns |
| **AI Engineers** | Study advanced architectures, memory systems, and framework comparisons |
| **Solution Architects** | Explore system design patterns and reference architectures |
| **Students** | Work through the 45-lesson curriculum, take quizzes, read research papers |

## External Systems

| System | Integration |
|---|---|
| **None** | The platform is fully self-contained with no external API calls, no database, and no AI service dependencies. All content is static Python data structures served via REST API. |
| **Research Paper Sources** | 17 curated papers from Anthropic, Google, Meta, OpenAI, and Princeton are referenced by URL — the platform links to them but does not host PDFs |
