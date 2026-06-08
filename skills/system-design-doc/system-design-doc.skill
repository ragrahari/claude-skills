---
name: system-design-doc
description: >
  Generate a structured system design document from requirements and context.
  Use this skill whenever someone provides a system to design, describes a product or feature
  needing architecture, or asks for a design doc, HLD, or technical design.
  Also trigger when the user provides functional requirements, constraints, or scale targets
  and wants a structured output — even if they don't say "system design doc" explicitly.
  Prompts like "design a ride-sharing app", "how would you architect X", "help me think
  through the design for Y", or "create a design doc for Z" should all trigger this skill.
---

# System Design Doc Skill

Produce a clear, structured system design document from provided context. Default depth is
**interview-level** (concise, covers the essentials) unless the user signals they want
production-level depth (e.g. "production-ready", "for my team", "include ops/runbook").

---

## Step 0 — Gather context before writing

Before generating the doc, check what the user has provided:

- **Functional requirements** — what the system must do
- **Scale targets** — users, RPS, data volume, geography
- **Constraints** — tech stack, budget, existing infra, compliance
- **Open questions** — ambiguities that would change the design

If critical information is missing (e.g. no sense of scale, no clarity on core use case),
ask 2–3 focused clarifying questions before proceeding. Do NOT ask more than 3 questions.
If scale is unspecified, assume a mid-scale internet product (10M users, ~1K RPS peak) and
state that assumption explicitly in the doc.

---

## Step 1 — Choose output format

Ask (or infer from context) whether the user wants:
- **Markdown** — for learning, notes, GitHub, or quick reference
- **Word doc (.docx)** — for sharing with a team or as a formal deliverable

If the user hasn't specified, default to Markdown and offer the Word doc at the end.

For `.docx` output: read `/mnt/skills/public/docx/SKILL.md` before generating the file.

---

## Step 2 — Generate the document

Use the structure below. At interview-level depth, each section should be concise — a few
sentences or a short list. At production depth, expand with specifics, failure modes, and ops.

---

### Document Structure

```
# System Design: [System Name]

## 1. Problem Statement
One paragraph: what problem is being solved, for whom, and why it matters.

## 2. Requirements
### Functional Requirements
- Core features the system must support (bullet list, prioritized)

### Non-Functional Requirements (NFRs)
- Scale: expected users, RPS, data volume
- Latency: p99 targets for critical paths
- Availability: SLA (e.g. 99.9%)
- Consistency: strong vs eventual
- Durability: data loss tolerance
- Security / Compliance: any constraints

## 3. Capacity Estimation
Back-of-envelope numbers:
- Read/write ratio
- Storage estimate (per record × expected records)
- Bandwidth estimate
- Cache hit rate assumptions

## 4. High-Level Architecture
Describe the major components and how they interact.
Include a component diagram using Mermaid syntax (flowchart LR).

Key components to cover where relevant:
- Clients (web, mobile, API consumers)
- Load balancer / API gateway
- Core services (name them based on the domain)
- Data stores (which type and why)
- Cache layer
- Message queue / event bus (if async)
- CDN (if media/static assets involved)

## 5. Data Modeling
- Core entities and their key fields
- Relationships between entities
- Choice of data store per entity (SQL / NoSQL / blob / graph) with brief justification
- Indexing strategy for hot query paths

## 6. API Design
List the core API endpoints. Use REST unless GraphQL/gRPC is justified.

Format:
METHOD /path — description
Request: key fields
Response: key fields

Cover at minimum: the primary read path, the primary write path, and any async callbacks.

## 7. Scalability & NFR Mapping
For each NFR, state the architecture decision that addresses it:

| NFR | Decision |
|-----|----------|
| High availability | ... |
| Low latency reads | ... |
| Durability | ... |
| Horizontal scale | ... |

## 8. Trade-off Analysis
Discuss 2–4 real design choices made and what was traded off:
- Option A vs Option B: why A was chosen, what B would have given
- Be honest about downsides of the chosen approach

## 9. Gotchas & Failure Modes
This section is critical. Cover:
- What breaks first at scale (the bottleneck)
- Hot partition / hot key problems
- Clock skew or ordering issues in distributed flows
- Cascading failures (what happens if service X goes down)
- Data consistency edge cases (e.g. double writes, partial failures)
- Common mistakes engineers make for this type of system
- Any compliance or security landmines

## 10. Out of Scope / Future Considerations
What was explicitly NOT designed here, and why.
What would you add in v2?
```

---

## Diagram guidance

Always include a Mermaid component diagram in Section 4. Use `flowchart LR` direction.
Keep it readable — 6–10 nodes max at interview level. Example style:

```
flowchart LR
  Client["Client"] --> LB["Load Balancer"]
  LB --> API["API Service"]
  API --> Cache["Redis Cache"]
  API --> DB["PostgreSQL"]
  API --> Queue["Kafka"]
  Queue --> Worker["Async Worker"]
  Worker --> DB
```

Render this using the Mermaid Chart MCP tool if available, otherwise include as a fenced
code block so the user can paste it into mermaid.live.

---

## Depth calibration

| Signal | Depth |
|--------|-------|
| "design X for an interview" / no qualifier | Interview-level (concise) |
| "production", "for my team", "detailed" | Production-level (expand each section) |
| "learning" / "understand" | Interview-level + extra explanation of *why* |

At interview-level: total doc should be readable in 10 minutes.
At production-level: expand failure modes, add runbook hooks, SLOs, alerting hints.

---

## Tone & style

- Be opinionated. Make a decision and justify it — don't hedge with "you could use X or Y".
- Call out gotchas clearly. Engineers learn more from "here's what breaks" than from happy paths.
- Use tables for NFR mappings and trade-offs — they're easier to scan.
- Avoid generic filler like "this ensures scalability." Say *how* it ensures scalability.
