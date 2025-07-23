# TimeBack – Agentic Upgrade Decision Log

*Created: 2025-07-20*

This document records the key architecture / design decisions agreed on 2025-07-20 that address the Priority Gaps listed in `ax-priority-plan.md` §2.  Each section cites the gap, enumerates the decision points considered, the option selected, and the rationale.

---

## 1. Agent-Action / Task Schema  (P0)

| Decision Point | Options Considered | Selected | Rationale |
|----------------|-------------------|----------|-----------|
| Storage model for tasks | **JSON-first** (flexible <br/>• quick to iterate) · **SQL-first** (strong typing, queryable) | **Hybrid** – tasks stored as a JSONB payload inside a relational table (Postgres), with typed columns for `task_id`, `owner_vid`, `status`, `created_at`, `due_at` | Combines rapid evolution of JSON with fast filter / join queries; `GIN` index on JSONB enables ad-hoc search. |
| Task state granularity | Simple 2-state (`pending`,`done`) · **4-state** (`pending`,`in_progress`,`completed`,`cancelled`) · Advanced workflow (multi-step) | **4-state** minimal model | Supports retry / cancellation without over-engineering; steps[] array can be added later. |

Follow-up deliverable: `agent-action-schema.md` (JSON shape + SQL DDL).

---

## 2. Interaction-Trace & Agent-Metrics Tables  (P0)

| Decision Point | Options Considered | Selected | Rationale |
|----------------|-------------------|----------|-----------|
| Event storage pattern | **Event-sourced log** (append-only blob) · **Table row per event** | Table row per event | Simple to query; row counts manageable (<10 M / yr). |
| Metric aggregation timing | Real-time compute | **Nightly batch** (pre-aggregate) | Keeps production queries fast; 24 h staleness acceptable for dashboards. |
| “Trust Score” definition | % accepted suggestions after 3 exposures vs. alternatives | **Adopt** % accepted after 3rd exposure | Aligns with compounding-value KPI. |

Follow-up: extend `analytics-spec.md` with `interaction_trace` & `daily_agent_metrics` schemas.

---

## 3. Live API Endpoint – `/agent/loop`  (P1)

| Decision Point | Options | Selected | Rationale |
|----------------|---------|----------|-----------|
| Transport | **SSE** · WebSocket | **SSE v1** | One-way streaming is enough for notifications; lighter infra; can upgrade later. |
| Auth | No token · **JWT token in `Authorization` header** | JWT token | Consistent with existing session model; protects PII in stream. |

Follow-up: update `api-contract.md` v2 with endpoint spec & security notes.

---

## 4. Front-End “Agent Feed” Component  (P1)

| Decision Point | Options | Selected | Rationale |
|----------------|---------|----------|-----------|
| Framework | **React (current)** · Vue | React | Matches Vite/React codebase. |
| Explanation tapering UI | Accordion · **Fade-in + auto-collapse** | Fade-in | Smoother UX; retains context without extra clicks. |

Follow-up: add component spec to `frontend-plan.md`.

---

## 5. AI Safety Guardrails  (P1)

| Decision Point | Options | Selected | Rationale |
|----------------|---------|----------|-----------|
| Red-team testing | **Internal team + open-source tools** · External firm | Internal + tools (e.g., Garak) | Quick, low-cost start; can escalate later. |
| Output filtering | Keyword blacklist · **Smart guard (LLM / ML filter)** | Smart guard | Higher recall on unsafe content, adaptable. |

Follow-up: add section to `security-checklist.md`.

---

## 6. Sprint Backlog Grouping  (P2)

| Decision Point | Options | Selected | Rationale |
|----------------|---------|----------|-----------|
| Work breakdown | By team · **By agent loop (capability)** | By loop | Keeps focus on end-to-end user value; reduces silo risk. |

Follow-up: rewrite `sprint-roadmap.md` around loops.

---

## Open Questions (carry-over)
1. Long-term: move from SSE to full duplex WebSocket when agent requires real-time user responses?
2. Should we persist task revision history in a separate `task_version` table (mirror of memory-graph pattern)?
3. Evaluate pgvector vs. Pinecone for task semantic search (see §5 Open Questions in `ax-priority-plan.md`).

---

*Prepared by AI assistant, 2025-07-20.  Updates welcome via PR.* 