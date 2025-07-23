# TimeBack Website – Content Tagging & RAG Schema

This document specifies the metadata each content asset must include so the personalization engine can assemble role-specific pages.

## 1. Core Metadata Fields
| Field | Type | Description | Example |
|-------|------|-------------|---------|
| id | string (slug) | Unique identifier for the asset | `brownsville_case` |
| icp | array<string> | Target ICPs (one or many) | `["parent","school"]` |
| outcome_tags | array<string> | Outcomes this asset supports | `["raise_grades","free_time"]` |
| context_key | string | Which context variable makes this asset relevant | `grade` / `district` / `stage` / `students` / `capital` |
| context_values | array<string> | Values that qualify; `"*"` for all | `["K-7","K-8"]` |
| trope | string | Familiar framing pattern | `before_after` / `dashboard` / `scorecard` |
| geography | string | ISO-3166 code or `global` | `US-TX` |
| proof_tier | enum | `testimonial` · `case_study` · `data_chart` · `explainer` | `case_study` |
| priority | int | 1-5 weighting for ranking within section | `1` |
| expires_at | date (optional) | Hide asset after this date; omit for evergreen | `2026-01-01` |
| tone | enum | `playful` · `expert` | `playful` |
| body | markdown | Renderable content or path to component template | *(see sample)* |

## 2. Storage Format
For MVP we will store assets as Markdown files with YAML front-matter in `content/`.
- Simple to edit in GitHub.
- Plays nicely with static fallback templates.
- RAG engine can parse front-matter → build vector or keyword index.

```
content/
  parents/
    brownsville_case.md
  schools/
    district_dashboard.md
```

## 3. Sample Asset File
```markdown
---
id: brownsville_case
icp: ["parent","school"]
outcome_tags: ["raise_grades"]
context_key: grade
context_values: ["K-6","K-7"]
trope: before_after
geography: US-TX
proof_tier: case_study
priority: 1
expires_at: 2026-01-01
tone: playful
---

## Brownsville Before / After

**31st → 84th percentile** in just **12 months**.  
_TimeBack students at Brownsville ISD crushed state averages after switching to 2-hour mastery blocks._

![Score Chart](../assets/brownsville_chart.png)

*Source: Brownsville ISD internal SAT10 data, 2024.*
```

## 4. Query Logic (Pseudo-code)
```js
function selectAssets(user) {
  return db.filter(a =>
    a.icp.includes(user.icp) &&
    a.outcome_tags.includes(user.outcome) &&
    (a.context_key === "*" || a.context_values.includes(user.context_value)) &&
    (a.geography === "global" || a.geography === user.region)
  ).sort(byPriority);
}
```

## 5. Open Questions
1. Do we allow wildcards in `context_values`?  
2. Versioning: add `rev` field or rely on git history?  
3. Image hosting path vs. base64 embed—performance trade-off.

*Last updated*: 2025-07-20 