# Product Requirements Document – TimeBack Dynamic Website (Draft v1)

## 1. Purpose
Deliver a public-facing TimeBack website that:
1. Feels hyper-personalised for every visitor (text, media, layout).
2. Captures *all* behavioural data (from first pixel to exit) and permanently links it to the visitor’s e-mail once provided.
3. Converts parents (primary ICP) into high-quality, nurture-ready leads.

## 2. Primary KPI
```
% of unique parent visitors that submit e-mail
```
Secondary metrics: scroll-depth ≥ 50 %, CTA interaction rate, compare-tool usage.

## 3. Target Segments
| ICP | Key Need | Tone |
|-----|----------|------|
| Parents (Montessori moms) | Proof of outcomes, emotional safety | Warm, story-driven |
| Educators / Admins | Evidence of mastery & implementation detail | Professional |
| Entrepreneurs | Turn-key school model, ROI | Visionary |
| Governments | Scale & sovereignty | Pragmatic |
| Philanthropists | Impact transparency | Inspirational |

## 4. Experience Principles
1. **Emotional hook first, tech second** – avoid “AI-Terminator” vibe.
2. **Progressive disclosure** – 1-click questionnaire → deeper content.
3. **< 3 s first paint** even when LLM is streaming.
4. **Zero dead-ends** – always a next, valuable action.

## 5. Data & Tracking Requirements
| Event | Payload | Frequency |
|-------|---------|-----------|
| `page_view` | route, referrer, ts | on load & route change |
| `scroll` | pct, delta, ts | 10 % increments |
| `click` | element_id, xy | every click |
| `hover` | element_id, dwell_ms | > 600 ms |
| `video_*` | start / q25 / q50 / q75 / complete | as fired |
| `form_focus/blur` | field | immediate |
| `idle` | secs_idle | 15 s heartbeat |

• Anonymous `sid` cookie until e-mail captured  
• `identify` event links historic sid → e-mail

All events stream to **Kafka/Redpanda → Tracker Worker → Visitor Vector Store (Redis/Pinecone)**.

## 6. Personalisation Engine
```
Edge → fetch(visitor_vector) → prompt LLM → JSON component tree → React render
```
Component grammar (v1): `Hero`, `StatBlock`, `StoryVideo`, `Diagram`, `CTAButton`, `FAQ`.

### Guardrails
1. LLM cannot invent facts – must reference **Fact Store** record id.  
2. Output validated against JSON schema; fallback to safe template.

## 7. Media Generation
| Media | How generated |
|-------|---------------|
| Text | LLM (mixtral-8x7b via Groq) |
| Images/Diagrams | Mermaid / PlantUML → SVG |
| Video | Pre-produced 5-20 s clips, LLM selects `videoId` |
| Charts | LLM inserts numbers → D3 component |
| Layout | LLM outputs JSON; front-end tailwind components render |

## 8. Analytics & Heat-maps
• Forward identical events to **PostHog Cloud** for click/scroll maps & session replay.  
• Use shared `distinct_id` (sid / e-mail) for cross-tool identity.

## 9. Privacy & Compliance
1. Cookie banner + TCF v2 for EU visitors.  
2. Store IP only as /24 hash.  
3. `/delete-me` endpoint removes sid & e-mail vector.  
4. No PII in session-replay payload.

## 10. Technical Stack
| Layer | Tech |
|-------|------|
| Front-end | Remix / React, Tailwind, shadcn-ui |
| Edge | Cloudflare Workers + KV |
| Event Bus | Redpanda (Kafka-compat) |
| Feature Store | Redis Vector / Pinecone |
| Fact Store | Neon/Postgres |
| LLM | Groq (mixtral-8x7b) → fallback GPT-4o |
| Analytics | PostHog |
| CI/CD | Turborepo → Cloudflare Pages |

## 11. Roll-out Plan
1. **Wk 1-2** – SDK + PostHog live, e-mail merge verified.  
2. **Wk 3** – Edge renderer (Hero + StatBlock).  
3. **Wk 4** – Video & Diagram components, Fact Store guardrails.  
4. **Wk 5** – Real-time re-render loop, A/B flags (e-mail-first vs after).  
5. **Wk 6** – Load test 5 k qps, sub-3 s FP.  
6. **Wk 7** – Security review (DDoS, prompt-inj, XSS).  
7. **Wk 8** – Guide-training launch (31 Jul).

## 12. Risks & Mitigations
| Risk | Mitigation |
|------|-----------|
| LLM hallucination | Strict Fact Store gating; JSON schema validation |
| DDoS cost-amplification | Cloudflare rate-limit; edge-cache anon content |
| Performance > 3 s | Token streaming; hero skeleton fallback |
| Privacy breach | Pseudonymised IP; replay masking; GDPR endpoint |

## 13. Front-End & UI/UX Architecture

### 13.1 Core Concept – “Living Story”
The shipped site is a thin React shell; every hero, stat block, chart, video, diagram – and even section layout – is **generated at runtime** from an LLM-produced JSON component tree that is re-evaluated _each time behaviour changes_.

```
Flow:  section-enter → fetch(visitor_vector) → prompt LLM → JSON tree → render
```

### 13.2 Component Grammar (v1)
| Allowed Component | Purpose |
|-------------------|---------|
| `Hero` | Emotional headline & subhead |
| `StatBlock` | Single bold stat + caption |
| `StoryVideo` | 5-20 s case-study clip (auto-captioned) |
| `Diagram` | Mermaid / PlantUML converted to SVG |
| `Chart` | D3 bar/line chart from Fact Store numbers |
| `FAQ` | Question/answer accordions |
| `CTAButton` | Primary action (eg. “Compare my district”) |
| `TwoColLayout` | Responsive two-column wrapper |

LLM output **cannot** contain raw HTML – it must reference this grammar and pass a prop object validated via Zod. Unknown or invalid specs fall back to a safe template.

### 13.3 Streaming Experience
1. Edge worker prompts LLM and streams first tokens immediately.  
2. Client shows skeleton (“Personalising for Austin families…”) within 300 ms.  
3. Sections hydrate progressively as JSON arrives – no full page flash.

### 13.4 Continuous Mutation
• Mini-quiz answers, scroll velocity, video completeness, etc. update the visitor-vector in <200 ms.  
• The very next unseen section is re-prompted with the _new_ vector, so the narrative adapts while the user is still on the page.

### 13.5 Visual Asset Pipeline
| Asset | Generation Path |
|-------|-----------------|
| Text | LLM (Groq – mixtral-8x7b) |
| Video | Pre-rendered clips selected by `videoId` |
| Diagrams | Mermaid/PlantUML → Web-Worker SVG |
| Charts | Fact IDs → numbers → D3 render |
| Background flair | Pre-approved image pool |

### 13.6 Performance Targets
• First Contentful Paint < **3 s** on 4 G mobile (95th pct).  
• Interaction ready (FID) < **100 ms**.  
• LLM JSON payload ≤ **25 KB** per section.

### 13.7 State & Identity in Browser
```ts
interface LocalState {
  sid: string // uuid
  email?: string
  lastVectorHash?: string
}
```
Stored in `localStorage`; if `email` exists, visitor bypasses the gate on subsequent visits.

### 13.8 Degradation & Error Modes
| Failure | Fallback |
|---------|----------|
| LLM timeout (>1.5 s) | Serve cached “generic parent” section |
| Edge store down | Static hero + single CTA |
| Diag/Chart render error | Replace with StatBlock variant |

### 13.9 Instrumentation Hooks
Every rendered component adds `data-tb` attributes (`data-tb-comp`, `data-tb-id`) so the event SDK can attribute clicks/hover/scroll precisely for heat-maps and replay.

---

---
Owner: Reece Harding / Tim Joo  
Version: Draft v1 (19 Jul 2025) 