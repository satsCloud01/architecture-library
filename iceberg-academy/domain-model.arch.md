---
name: "Domain Model"
project: "Iceberg Academy"
project_slug: "iceberg-academy"
project_url: null
github: null
category: "learning"
type: "domain-model"
icon: "🧊"
tags: [FastAPI, SQLite, domain-model, Iceberg, Arrow]
---

# Iceberg Academy -- Domain Model

## Bounded Contexts

1. **Curriculum** -- Modules, lessons, and their ordering across 5 technologies
2. **Assessment** -- Quiz questions, answer checking, and attempt tracking
3. **Progress Tracking** -- Lesson completion, notes, and per-technology statistics
4. **Hands-On Learning** -- Labs (guided exercises) and Playground (free-form code snippets)
5. **Reference** -- Glossary terms, research papers, architecture diagrams, design patterns

## Technologies (Enum)

The curriculum is organized around 5 core technologies:

| Value | Technology |
|-------|-----------|
| `iceberg` | Apache Iceberg table format |
| `rest_catalog` | Iceberg REST Catalog (Polaris, Nessie) |
| `arrow` | Apache Arrow in-memory columnar format |
| `flight` | Arrow Flight high-performance data transport |
| `flightsql` | Arrow Flight SQL protocol + ADBC |

## Database Schema (SQLite)

### Core Curriculum Tables

| Table | Columns | Relationships |
|-------|---------|---------------|
| `modules` | id, slug (unique), title, description, icon, order_num, technology | Parent of lessons, quiz_questions, labs |
| `lessons` | id, module_id (FK), title, slug, content, code_example, order_num, difficulty | Belongs to module; unique(module_id, slug) |
| `quiz_questions` | id, module_id (FK), question, options (JSON), correct_answer, explanation, difficulty | Belongs to module |
| `glossary_terms` | id, term, definition, technology, related_terms (JSON) | Standalone; filtered by technology |

### User Activity Tables

| Table | Columns | Relationships |
|-------|---------|---------------|
| `user_progress` | id, lesson_id (FK, unique), completed, completed_at, notes | One-to-one with lesson |
| `quiz_attempts` | id, question_id (FK), selected_answer, is_correct, attempted_at | Many-to-one with quiz_question |
| `playground_snippets` | id, title, code, language, technology, created_at | Standalone |
| `chat_messages` | id, session_id, role, content, technology, created_at | Grouped by session_id |

### Hands-On Tables

| Table | Columns | Relationships |
|-------|---------|---------------|
| `labs` | id, module_id (FK), title, description, instructions, starter_code, solution_code, hints (JSON), difficulty, order_num | Belongs to module |

## Entity Relationship Diagram

```
┌──────────┐       ┌──────────┐       ┌───────────────┐
│ modules  │──1:N──│ lessons  │──1:1──│ user_progress │
│          │       └──────────┘       └───────────────┘
│          │
│          │──1:N──┌────────────────┐──1:N──┌───────────────┐
│          │       │ quiz_questions │       │ quiz_attempts │
│          │       └────────────────┘       └───────────────┘
│          │
│          │──1:N──┌──────┐
└──────────┘       │ labs │
                   └──────┘

┌────────────────┐   ┌──────────────────────┐   ┌───────────────┐
│ glossary_terms │   │ playground_snippets  │   │ chat_messages │
│ (standalone)   │   │ (standalone)         │   │ (by session)  │
└────────────────┘   └──────────────────────┘   └───────────────┘
```

## Pydantic Models (22 total)

### Enums

| Enum | Values |
|------|--------|
| `Technology` | iceberg, rest_catalog, arrow, flight, flightsql |
| `Difficulty` | beginner, intermediate, advanced |

### Curriculum Models

| Model | Fields | Usage |
|-------|--------|-------|
| `ModuleOut` | id, slug, title, description, icon, order_num, technology, lesson_count, completed_count | List modules response |
| `ModuleDetail` | id, slug, title, description, icon, order_num, technology, lessons: list[LessonSummary] | Single module with lesson list |
| `LessonOut` | id, module_id, title, slug, content, code_example, order_num, difficulty, completed | Full lesson response |
| `LessonSummary` | id, title, slug, order_num, difficulty, completed | Abbreviated lesson in module detail |

### Progress Models

| Model | Fields | Usage |
|-------|--------|-------|
| `CompleteRequest` | lesson_id, notes (optional) | Mark/unmark lesson complete |
| `ProgressSummary` | total_lessons, completed_lessons, completion_pct, total_quizzes, correct_quizzes, quiz_accuracy, by_technology | Dashboard stats |
| `CompletedLesson` | lesson_id, lesson_title, module_title, completed_at, notes | Completed lessons list |

### Quiz Models

| Model | Fields | Usage |
|-------|--------|-------|
| `QuizQuestionOut` | id, module_id, question, options: list[str], difficulty | Quiz question (excludes correct_answer) |
| `CheckAnswerRequest` | question_id, selected_answer | Submit answer |
| `CheckAnswerResponse` | is_correct, correct_answer, explanation | Answer feedback |
| `QuizStats` | total_attempts, correct_count, accuracy, by_module | Aggregate quiz statistics |

### Playground Models

| Model | Fields | Usage |
|-------|--------|-------|
| `SaveSnippetRequest` | title, code, language, technology | Save user code |
| `SnippetOut` | id, title, code, language, technology, created_at | Saved snippet response |
| `CodeExample` | title, code, language, description | Preloaded example (in-memory) |

### Glossary & Lab Models

| Model | Fields | Usage |
|-------|--------|-------|
| `GlossaryTermOut` | id, term, definition, technology, related_terms: list[str] | Glossary entry |
| `LabOut` | id, module_id, title, description, instructions, starter_code, solution_code, hints: list[str], difficulty, order_num, technology | Full lab detail |
| `LabCheckRequest` | code | Submit lab code for checking |
| `LabCheckResponse` | feedback, score | Lab check result |

### Architecture Models

| Model | Fields | Usage |
|-------|--------|-------|
| `DiagramNode` | id, label, technology, description, x, y | Node in architecture diagram |
| `DiagramEdge` | source, target, label | Edge connecting diagram nodes |
| `ArchitectureDiagram` | title, description, nodes: list[DiagramNode], edges: list[DiagramEdge] | Complete diagram for React Flow |

### AI Tutor Models (defined but not actively routed)

| Model | Fields | Usage |
|-------|--------|-------|
| `ChatRequest` | session_id, message, technology | Chat input |
| `ChatResponse` | role, content, session_id | Chat response |
| `Suggestion` | title, reason, module_slug, lesson_slug | Learning suggestion |

## Seed Data Summary

| Entity | Seeded Count | Notes |
|--------|-------------|-------|
| Modules | 5+ | One per technology (iceberg, rest_catalog, arrow, flight, flightsql) |
| Lessons | Multiple per module | Ordered within each module |
| Quiz Questions | Multiple per module | With JSON options array |
| Glossary Terms | Multiple | Spanning all technologies |
| Labs | Multiple | Tied to modules |
| Papers | 17 | In-memory (not in SQLite) |
| Patterns | 10 | In-memory (not in SQLite) |
| Architecture Diagrams | 3 | In-memory (Modern Lakehouse, Metadata Hierarchy, Flight Data Flow) |
| Playground Examples | 13 | In-memory across 5 technology categories |
