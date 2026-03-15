---
name: "C3 Frontend Components"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "c4-component"
icon: "🧊"
tags: [React, Vite, Tailwind, "React Router", "React Flow"]
---

# Iceberg Academy -- C3: Frontend Components

## Pages (12)

| Page | Route | Description |
|------|-------|-------------|
| `Landing` | `/` | Hero section with technology overview and call-to-action |
| `Dashboard` | `/dashboard` | Progress overview: completion %, quiz accuracy, by-technology breakdown |
| `ModuleView` | `/modules/:slug` | Module detail with lesson list and completion status |
| `LessonView` | `/modules/:slug/lessons/:lessonSlug` | Full lesson content with code examples, mark-complete toggle |
| `Quiz` | `/modules/:slug/quiz` | Interactive quiz per module with immediate answer checking |
| `Playground` | `/playground` | Code playground with preloaded examples for all 5 technologies, save/load snippets |
| `Glossary` | `/glossary` | Searchable glossary with technology filter and related terms |
| `Labs` | `/labs` | Lab listing with difficulty badges |
| `LabDetail` | `/labs/:id` | Hands-on lab with instructions, starter code, solution reveal, hints |
| `Architecture` | `/architecture` | Interactive architecture diagrams (React Flow nodes + edges) |
| `Papers` | `/papers` | Curated research papers and specs with key insights |
| `Patterns` | `/patterns` | 10 data architecture patterns with flow diagrams and use cases |

## Shared Components

| Component | Purpose |
|-----------|---------|
| `Layout` | Persistent sidebar/nav + `<Outlet>` for page content |

## API Client

All pages call the backend via fetch to `/api/*` endpoints. No dedicated API client module -- fetch calls are inline in page components.
