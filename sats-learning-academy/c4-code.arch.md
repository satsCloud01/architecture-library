---
project: SatsLearning Academy
project_slug: sats-learning-academy
project_url:
github: https://github.com/satsCloud01/sats-learning-academy
category: ai-agents
icon: "\U0001F393"
name: Code Patterns
type: c4-code
tags: [C4, code, patterns]
---

# C4 — Code Level

## Lesson Data Structure

```python
# Static data defined in Python source (no database)
#
# Each lesson is a dict with the following shape:

LESSONS = [
    {
        "id": "foundations-01",
        "title": "What Are AI Agents?",
        "category": "Foundations",        # One of 7 categories
        "order": 1,                       # Sort order within category
        "content": "...",                 # Markdown or structured content
        "key_concepts": ["autonomy", "tool-use", "reasoning"],
        "diagram_type": "sequence",       # Optional: which visual component to render
        "diagram_data": { ... },          # Optional: data for the visual component
    },
    # ... 45 total lessons
]

# 7 Categories and their lesson counts:
# - Foundations:       6 lessons
# - Protocols:        13 lessons (MCP, A2A, etc.)
# - Design Patterns:   7 lessons
# - Architectures:     5 lessons
# - Memory & State:    4 lessons
# - Frameworks:        5 lessons
# - Advanced:          5 lessons
```

## Simulator State Machine

```python
# The simulator models ReAct (Reason + Act) execution
# Each scenario has a sequence of steps that alternate between
# Thought, Action, and Observation phases.

SCENARIOS = [
    {
        "id": "react-web-search",
        "title": "ReAct Web Search Agent",
        "description": "Step through a ReAct agent performing web search",
        "steps": [
            {
                "step_number": 1,
                "phase": "thought",       # "thought" | "action" | "observation"
                "content": "I need to find the latest...",
                "tool": None,             # Tool name for action steps
                "tool_input": None,       # Tool parameters for action steps
                "tool_output": None,      # Result for observation steps
            },
            {
                "step_number": 2,
                "phase": "action",
                "content": "Calling web_search tool",
                "tool": "web_search",
                "tool_input": {"query": "..."},
                "tool_output": None,
            },
            {
                "step_number": 3,
                "phase": "observation",
                "content": "Search returned results...",
                "tool": None,
                "tool_input": None,
                "tool_output": {"results": ["..."]},
            },
            # ... continues alternating phases
        ],
    },
    # ... 4 total scenarios
]

# POST /api/simulator/step accepts a scenario_id and current_step,
# returns the next step in the sequence for animated playback.
```

## Quiz Scoring Logic

```python
# Quiz questions are multiple-choice with a single correct answer.
# Scoring is computed server-side for integrity.

QUESTIONS = [
    {
        "id": "q01",
        "question": "Which protocol enables tool discovery for LLMs?",
        "options": ["A2A", "MCP", "gRPC", "REST"],
        "correct": "MCP",
        "explanation": "Model Context Protocol (MCP) provides...",
        "category": "Protocols",
    },
    # ... 10 total questions
]

# POST /api/quiz/check
# Request:  { "answers": {"q01": "MCP", "q02": "A2A", ...} }
# Response: {
#     "score": 8,
#     "total": 10,
#     "percentage": 80.0,
#     "results": [
#         {"id": "q01", "correct": true,  "your_answer": "MCP", "correct_answer": "MCP"},
#         {"id": "q02", "correct": false, "your_answer": "A2A", "correct_answer": "gRPC"},
#         ...
#     ],
#     "by_category": {"Protocols": {"correct": 3, "total": 4}, ...}
# }
#
# The frontend renders the by_category breakdown using Recharts bar/pie charts.
```

## Visual Component Pattern

```python
# The 6 custom visual components render protocol and architecture
# diagrams inline within lessons. They receive structured data
# from the lesson's diagram_data field.
#
# React component pattern:
#
# function SequenceDiagram({ participants, messages }) {
#     // participants: ["Client", "MCP Server", "LLM"]
#     // messages: [
#     //   { from: "Client", to: "MCP Server", label: "initialize" },
#     //   { from: "MCP Server", to: "Client", label: "capabilities" },
#     //   ...
#     // ]
#     // Renders vertical lifelines with horizontal arrows
# }
#
# Components are selected per-lesson via diagram_type field:
# "sequence"   -> SequenceDiagram
# "transport"  -> TransportDiagram
# "ecosystem"  -> McpEcosystem
# "state"      -> StateMachine
# "agent"      -> AgentCard
# "message"    -> MessageFormats
```
