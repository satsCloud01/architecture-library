---
title: Networking Academy — Domain Model
status: current
---

# Domain Model

## Entities

### 1. Lesson (50 instances)
Primary learning content unit. Each lesson belongs to exactly one category.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier (1-50) |
| category | string | One of 13 categories |
| title | string | Display title |
| type | string | `intro`, `concept`, `summary` |
| description | string | Main content paragraph |
| keyPoints | string[] | Bullet-point takeaways |
| comparison | object? | Table with headers + rows |
| diagram | object? | Flow/architecture diagram (nodes + edges) |
| sequenceDiagram | object? | Protocol sequence (actors + steps) |
| code | string? | Code/config example |
| codeLanguage | string? | Language hint for display |

### 2. Category (13 instances)
Logical grouping of lessons. Derived at runtime, not stored separately.

| Category | Lessons | Coverage |
|----------|---------|----------|
| Fundamentals | 5 | Transmission media, hardware, network types, topologies |
| OSI Model | 4 | 7-layer model, encapsulation, PDUs, troubleshooting |
| TCP/IP | 4 | TCP/IP vs OSI, handshake, TCP vs UDP, ports |
| IP Addressing | 4 | IPv4 anatomy, classes, RFC 1918, IPv6 |
| Subnetting & CIDR | 3 | Subnet fundamentals, CIDR notation, VLSM |
| Routing | 4 | Routing tables, static/dynamic, IGPs, BGP |
| Switching & VLANs | 4 | MAC learning, VLANs, STP, LACP |
| Transport Layer | 3 | Flow control, congestion control, QUIC |
| DNS & DHCP | 3 | DNS resolution, record types, DHCP DORA |
| Network Security | 4 | Firewalls, TLS, Zero Trust, attack vectors |
| Wireless | 3 | Wi-Fi standards, channels, security/OFDMA |
| SDN & Cloud | 3 | SDN architecture, cloud networking, load balancing |
| Advanced Topics | 6 | MPLS, Segment Routing, YANG/NETCONF, emerging tech |

### 3. SimulatorScenario (4 instances)
Interactive step-through of real network operations.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | Scenario name |
| description | string | What the scenario demonstrates |
| steps | Step[] | Ordered execution steps |

### 4. SimulatorStep (~30 total)
Individual step within a scenario, following the ReAct pattern.

| Field | Type | Description |
|-------|------|-------------|
| type | enum | `thought`, `action`, `observation`, `answer` |
| label | string | Step title |
| content | string | Detailed description of what happens |

**Scenarios:**
1. TCP Three-Way Handshake (7 steps)
2. DNS Resolution: www.example.com (8 steps)
3. Packet Journey: LAN to Internet (9 steps)
4. DHCP: Getting an IP Address (7 steps)

### 5. QuizQuestion (15 instances)
Multiple-choice assessment with server-side scoring.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| category | string | Maps to lesson category |
| question | string | Question text |
| options | string[4] | Four answer choices |
| correct | int | Index of correct answer (server-only) |
| explanation | string | Why the answer is correct (server-only) |

### 6. Paper (17 instances)
Curated RFCs and seminal networking papers.

| Field | Type | Description |
|-------|------|-------------|
| id | int | Unique identifier |
| title | string | RFC number + title |
| authors | string | Author names |
| year | int | Publication year |
| category | string | Foundational, Routing, Security, Transport, Cloud, Addressing, Advanced |
| url | string | Link to original document |
| summary | string | One-paragraph summary |
| key_takeaways | string[] | 5-6 most important points |
| related_topics | string[] | Connected concepts/RFCs |

## Entity Relationships

```
Category ──1:N──▶ Lesson
Category ──1:N──▶ QuizQuestion
Category ──1:N──▶ Paper (via paper.category)
SimulatorScenario ──1:N──▶ SimulatorStep
```

All entities are **read-only** at runtime. No CRUD operations. No user-generated content.

## Data Storage

All data is defined as Python lists/dicts in source code:
- `backend/src/networkacademy/data/lessons.py` — LESSONS, QUIZ_QUESTIONS
- `backend/src/networkacademy/routers/simulator.py` — SCENARIOS (inline)
- `backend/src/networkacademy/routers/papers.py` — PAPERS (inline)

No database. No migrations. No ORM.
