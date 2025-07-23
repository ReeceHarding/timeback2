# TimeBack AX Priority Plan

> Last updated: {{DATE}}

## 1. AX-Readiness Scorecard (current documentation)

| AX Capability | Status | Key Evidence / Gap |
| ------------- | ------ | ------------------ |
| **Memory Graph** | ⚠️ Partial | `analytics-spec.md` defines core student/assessment tables, but no versioning or merge logic. |
| **Real-Time Agent Loop** | ⚠️ Partial | `personalization-strategy.md` specifies quiz → refine loop; missing automated task creation & telemetry. |
| **Preference / Context Capture** | ⚠️ Partial | Session & quiz context captured; no interaction-trace schema. |
| **Trust-Building UX** | ⚠️ Partial | Explain panel noted; no tapering rules for progressive disclosure. |
| **Compounding-Value Metrics** | ❌ Missing | Growth metrics exist, but not tied to agent actions (accept rate, autonomy ratio). |
| **Security & PII Guardrails** | ✅ Solid | `security-checklist.md` covers encryption & masking; lacks LLM safe-completion. |
| **Typed API Contract** | ⚠️ Partial | REST parse/personalize endpoints exist; need streaming / fn-call endpoint for live agent. |
| **Demo Alignment** | ⚠️ Partial | 24-h demo is dashboard-centric, not agent-centric. |
| **Road-map Cohesion** | ⚠️ Partial | Sprint backlog feature-siloed; not mapped to capability loops. |

**Overall AX readiness ≈ 65 %.**

---

## 2. Priority Gaps & Fixes

| Priority | Gap to Close | Concrete Deliverable | Owner | ETA |
| -------- | ------------ | -------------------- | ----- | --- |
| P0 | No formal **agent-action/task** contract | `agent-action-schema.md` (JSON + SQL DDL) | Data Eng | ☐ |
| P0 | Missing **interaction-trace** + **agent_metrics** tables | Amend `analytics-spec.md` | Data Eng | ☐ |
| P1 | API lacks streaming / function-call loop | `api-contract.md` v2 (`POST /agent/loop`) | Backend | ☐ |
| P1 | Front-end lacks Agent Feed | Update `frontend-plan.md` with component spec & tapering rules | Front-end | ☐ |
| P1 | LLM safe-completion & red-team playbook | Add section to `security-checklist.md` | SecOps | ☐ |
| P2 | Sprint backlog mis-aligned | Rewrite `sprint-roadmap.md` around capability loops | PM | ☐ |

---

## 2.a Decisions Snapshot (2025-07-20)

| Gap Addressed | Key Decision (tl;dr) | Reference |
|---------------|----------------------|-----------|
| Agent-Action / Task Schema | **Hybrid** JSONB in SQL table · 4-state lifecycle | `docu/agentic-decisions.md#1-agent-action-/-task-schema` |
| Interaction Trace & Metrics | Row-per-event table · metrics nightly | `docu/agentic-decisions.md#2-interaction-trace-&-agent-metrics-tables` |
| Live API Streaming | `/agent/loop` via **SSE** + JWT auth | `docu/agentic-decisions.md#3-live-api-endpoint-–-/agent/loop` |
| Front-End Agent Feed | React component · fade-in tapering | `docu/agentic-decisions.md#4-front-end-“agent-feed”-component` |
| AI Safety Guardrails | Internal red-team + smart output filter | `docu/agentic-decisions.md#5-ai-safety-guardrails` |
| Sprint Backlog Grouping | Group tasks by **agent loop** | `docu/agentic-decisions.md#6-sprint-backlog-grouping` |

*See the linked doc for full option matrix & rationale.*

---

## 3. Road-map (90-Day)

1. **Week 1-2 – Data Contracts**
   * Ship `agent-action-schema.md` + analytics extensions.
   * Migrate ETL to write versioned entity deltas.
2. **Week 3-4 – API & Front-end Stub**
   * Implement `/agent/loop` endpoint (stream ≥ SSE).
   * Ship minimal "Agent Feed" UI with full chain-of-thought.
3. **Week 5-6 – Security & Telemetry**
   * Integrate safe-completion filters.
   * Funnel interaction traces to warehouse.
4. **Week 7-8 – First Agent Loop (HiAvg Plateau)**
   * Detection ↔ suggestion ↔ teacher-task cycle.
   * Track acceptance & growth deltas.
5. **Week 9-10 – Equity Gap Loop**
   * Auto-detect demographic growth gaps.
   * Push recommended PD / resource allocation.
6. **Week 11-12 – Demo & Retrospective**
   * Live demo with principal persona.
   * Collect feedback → refine metric weights.

---

## 4. Success Metrics

* **Trust Score** – % of agent suggestions accepted after 3 exposures.
* **Autonomy Ratio** – tasks auto-completed ÷ total tasks over time.
* **Compounded Growth Delta** – ∆(student growth) post-agent vs. pre-agent.
* **Engagement** – reduction in RapidGuessing% after agent-nudged interventions.

---

## 5. Open Questions

1. **Storage** – pgvector vs. Pinecone for semantic memory?
2. **Explainability UI** – best pattern for tapering chain-of-thought?
3. **Offline districts** – fallback workflow when PDFs cannot be parsed on-prem?

---

*Document prepared by AI assistant based on current `/docu` and `/context` folders.* 