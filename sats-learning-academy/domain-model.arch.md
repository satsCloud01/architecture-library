---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: Domain Model
type: domain-model
tags: [domain, entities, relationships]
---

# SatsLearning Academy — Domain Model

**Agentic AI Educational Platform**

This document describes the 6 domain entities, their key fields, and relationships. All entities are defined as static Python data structures (dicts/lists) — there is no database.

---

## Entity Relationship Overview

```
Category 1───* Lesson
                 |
                 +---(optional)---> diagram_type -> Visual Component

SimulatorScenario 1───* SimulatorStep

QuizQuestion (standalone — grouped by category)

Paper (standalone — curated research references)
```

---

## Entity Definitions

### 1. Lesson

A single educational unit within the 45-lesson curriculum.

| Field | Type | Description |
|---|---|---|
| `id` | str | Unique identifier (e.g., `foundations-01`) |
| `title` | str | Lesson title |
| `category` | str | Parent category name |
| `order` | int | Sort order within the category |
| `content` | str | Lesson body (structured text or markdown) |
| `key_concepts` | list[str] | Key terms and concepts covered |
| `diagram_type` | str (nullable) | Visual component type: `sequence`, `transport`, `ecosystem`, `state`, `agent`, `message` |
| `diagram_data` | dict (nullable) | Structured data passed to the visual component |

**Relationships:** Belongs to a Category (via `category` field). Optionally renders a Visual Component.

---

### 2. Category

A grouping of related lessons. Derived from lesson data, not stored separately.

| Field | Type | Description |
|---|---|---|
| `name` | str | Category name |
| `lesson_count` | int | Number of lessons in this category |
| `order` | int | Display order |

**7 Categories:**

| # | Category | Lessons |
|---|---|---|
| 1 | Foundations | 6 |
| 2 | Protocols | 13 |
| 3 | Design Patterns | 7 |
| 4 | Architectures | 5 |
| 5 | Memory & State | 4 |
| 6 | Frameworks | 5 |
| 7 | Advanced | 5 |
| | **Total** | **45** |

**Relationships:** Has many Lessons.

---

### 3. SimulatorScenario

A ReAct agent execution scenario for step-by-step visualization.

| Field | Type | Description |
|---|---|---|
| `id` | str | Unique identifier (e.g., `react-web-search`) |
| `title` | str | Scenario display name |
| `description` | str | What the scenario demonstrates |
| `agent_type` | str | Type of agent being simulated |
| `total_steps` | int | Number of steps in the scenario |
| `steps` | list[SimulatorStep] | Ordered sequence of execution steps |

**Relationships:** Has many SimulatorSteps.

---

### 4. SimulatorStep

A single step in a ReAct agent execution trace.

| Field | Type | Description |
|---|---|---|
| `step_number` | int | Ordinal position in the scenario |
| `phase` | str | `thought`, `action`, or `observation` |
| `content` | str | Human-readable description of what happens |
| `tool` | str (nullable) | Tool name (for `action` phase only) |
| `tool_input` | dict (nullable) | Tool parameters (for `action` phase only) |
| `tool_output` | dict (nullable) | Tool result (for `observation` phase only) |

**Relationships:** Belongs to a SimulatorScenario.

---

### 5. QuizQuestion

A multiple-choice question for knowledge assessment.

| Field | Type | Description |
|---|---|---|
| `id` | str | Unique identifier (e.g., `q01`) |
| `question` | str | Question text |
| `options` | list[str] | Answer choices (typically 4) |
| `correct` | str | The correct answer (must match one of `options`) |
| `explanation` | str | Explanation shown after answering |
| `category` | str | Topic category for score breakdown |

**Relationships:** Standalone entity. Grouped by `category` for Recharts score visualization.

---

### 6. Paper

A curated research paper reference from a leading AI lab.

| Field | Type | Description |
|---|---|---|
| `id` | str | Unique identifier (e.g., `paper-01`) |
| `title` | str | Paper title |
| `authors` | list[str] | Author names |
| `organization` | str | Publishing lab: `Anthropic`, `Google`, `Meta`, `OpenAI`, `Princeton` |
| `year` | int | Publication year |
| `url` | str | Link to the paper (arXiv, official site, etc.) |
| `abstract` | str | Paper abstract or summary |
| `tags` | list[str] | Topic tags (e.g., `agents`, `reasoning`, `tool-use`) |

**Relationships:** Standalone entity. The frontend renders papers in a filterable list with an embedded iframe reader.

---

## Entity Summary Table

| # | Entity | Storage | Count | Primary Relationships |
|---|---|---|---|---|
| 1 | Lesson | Python list[dict] | 45 | Belongs to Category; optionally links Visual Component |
| 2 | Category | Derived from Lessons | 7 | Groups Lessons |
| 3 | SimulatorScenario | Python list[dict] | 4 | Has many SimulatorSteps |
| 4 | SimulatorStep | Nested in Scenario | ~40 | Belongs to SimulatorScenario |
| 5 | QuizQuestion | Python list[dict] | 10 | Standalone (grouped by category) |
| 6 | Paper | Python list[dict] | 17 | Standalone |

---

## Data Lifecycle

All entities are **read-only at runtime**. Content is authored by developers in Python source files, committed to version control, and served directly from memory by the FastAPI backend. There is no create, update, or delete operation for any entity — the API is purely read-only (except `POST /api/quiz/check` for scoring and `POST /api/simulator/step` for step advancement, which are stateless computations).
