# TimeBack Website – Agent-Action / Task Schema (Marketing Site)

*Created: 2025-07-20*

## Purpose
Define the minimal data contract for any “action” the LLM-powered agent can schedule for a visitor on the **public marketing site**.  Actions are rendered as personalised blocks (immediate) or queued follow-ups (e-mail, reminder banner).

## 1. JSON Payload (canonical)
```jsonc
{
  "task_id": "uuid",          // server-generated
  "vid": "uuid",              // visitor id (cookie)
  "action_type": "show_block", // show_block | schedule_email | analytic_ping
  "summary": "Send grade-4 reminder banner",
  "payload": {
    "block_id": "before_after_chart",
    "ttl": 3600
  },
  "status": "pending",        // pending | in_progress | completed | cancelled
  "created_at": "2025-07-20T12:00:00Z",
  "due_at": null               // optional (follow-ups)
}
```

## 2. SQL DDL  (PostgreSQL ≥ 15)
```sql
CREATE TABLE tasks (
  task_id        uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  vid            uuid NOT NULL,
  action_type    text NOT NULL CHECK (action_type IN ('show_block','schedule_email','analytic_ping')),
  summary        text NOT NULL,
  payload        jsonb NOT NULL,
  status         text NOT NULL CHECK (status IN ('pending','in_progress','completed','cancelled')),
  created_at     timestamptz NOT NULL DEFAULT now(),
  due_at         timestamptz,
  completed_at   timestamptz,
  cancelled_at   timestamptz
);

-- Indexes
CREATE INDEX tasks_vid_status_idx      ON tasks (vid, status);
CREATE INDEX tasks_due_at_idx          ON tasks (due_at);
CREATE INDEX tasks_payload_gin_idx     ON tasks USING GIN (payload);
```

## 3. Relation to `interaction_trace`
```mermaid
erDiagram
  tasks ||--o{ interaction_trace : "emits"
  tasks {
    uuid task_id PK
    uuid vid
    text status
  }
  interaction_trace {
    bigint trace_id PK
    uuid vid
    uuid task_id FK
    text event_type
    timestamptz ts
  }
```

## 4. Lifecycle State Machine
```
pending → in_progress → completed
     ↘──────────────┘
     cancelled
```
Transition rules:
* Only `pending` tasks can move to `in_progress` or `cancelled`.
* Only `in_progress` can move to `completed`.

## 5. Open Questions
1. Do we need per-step subtasks (`steps[]`) now or can we postpone?
2. Should `payload` be versioned similar to Memory Graph pattern?

---
*Prepared by AI assistant – feedback welcome via PR.* 

## 6. Payload Legend (MVP)

| action_type | JSON Schema file | Required Fields | Example Snippet |
|-------------|-----------------|-----------------|-----------------|
| `show_block` | `/schemas/show_block.json` | `block_id` (string) · `ttl` (int s) | `{ "block_id":"before_after_chart", "ttl":3600 }` |
| `schedule_email` | `/schemas/schedule_email.json` | `template_id` (string) · `send_at` (ISO ts) | `{ "template_id":"demo_invite_parent", "send_at":"2025-07-22T12:00:00Z" }` |
| `analytic_ping` | `/schemas/analytic_ping.json` | `label` (string) | `{ "label":"variant_B_winner" }` |

> Schemas live under `/backend/schemas/` and are validated at insert/update time.

---

## 7. Post-MVP Enhancements (Recorded for Road-Map)

| Feature | Rationale | Proposed Approach | Priority |
|---------|-----------|-------------------|----------|
| `priority`, `attempts`, `max_attempts` columns | Needed when multiple workers or SLAs require ordering/retries (e.g., ESP downtime). | Add nullable smallint columns with defaults `priority=0`, `max_attempts=3`. Increment `attempts` per retry. | P1 (after successful SSE launch) |
| `steps[]` or `task_step` table | Model multi-email drips or multi-block reveal sequences. | Keep current flat tasks; later add child table `task_step (task_id, step_idx, status, due_at)`. | P2 |
| Immutable `task_version` history | Full audit trail and rollback of marketing copy changes. | Mirror Memory Graph pattern: `task_version` table + pointer column on `tasks`. | P3 |
| Additional `action_type`s | Future interactive elements (e.g., `show_modal`, `push_notification`). | Extend enum and add matching JSON Schema. | P1-P2 as features demand |
| Extended security metadata | Trace content safety decisions (moderation flags). | Add `moderation_result` JSONB in `payload` or separate column. | P2 |

These items are **NOT** in the MVP migration but are documented to avoid scope amnesia.

--- 