# TimeBack Website – Personalization API Contract

_Scope: this contract covers the **public marketing site (timeback.com)**. Internal product dashboards or future SaaS portals are out-of-scope for this version._

This specification defines the HTTP endpoints, payload shapes, and latency targets that connect the front-end 
(React SPA) with the personalization engine.

---
## Core Data Concepts

To prevent ambiguity, it is critical to understand the separation of two key entities in the TimeBack ecosystem: **Users** and **Students**.

### 1. Users (Website Visitors)
- **Definition:** A **User** is an individual who visits the TimeBack website and provides their email address. They are the audience for our personalized content.
- **Identification:** The primary key for a User is their `email` address, which must be unique.
- **Represents:** A `parent`, `school leader`, `entrepreneur`, or any other ICP we target. The User is the person consuming the information.
- **Database Table:** `users`

### 2. Students
- **Definition:** A **Student** is an individual whose educational data is part of the raw datasets we import (e.g., from CSV files). They are the *subject* of the data, not the consumer of the website.
- **Identification:** The primary key for a Student is their `StudentID` from the source data files.
- **Represents:** A learner whose progress, assessments, and school information we process to generate insights *for* the Users.
- **Database Table:** `students`

**Crucial Distinction:** The `users` table and the `students` table are completely separate and must not be conflated. A "parent" is a User; their child is a "Student".

---

> All endpoints are JSON over HTTPS. Responses include `Content-Type: application/json; charset=utf-8`.

## 1. `/session/start` – Create or Resume Visitor Session
```
POST /session/start
```
### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | yes | Visitor’s email captured on the hero screen. |
| icp   | string enum | yes | `parent` · `school` · `entrepreneur` · `government` · `philanthropist` |
| user_agent | string | no | Sent automatically by browser; logged for analytics. |

### Response 200
| Field | Type | Description |
|-------|------|-------------|
| vid | string | Opaque visitor token (UUID v4). Persisted as `tb_vid` cookie. |
| new_user | boolean | `true` if this email hasn’t been seen before. |
| duplicate | boolean | `true` if email already existed (same vid re-issued). |

*Cookie flags:* `tb_vid` is set with `Secure; HttpOnly; SameSite=Lax`.

### Error Codes
| HTTP | Error | Meaning |
|------|-------|---------|
| 400 | `invalid_email` | Malformed or missing address. |
| 429 | `rate_limited` | Too many attempts from same IP/email. |

### Latency Target
P90 ≤ **150 ms** (server processing).

---
## 2. `/personalize` – Generate Page Blocks
```
POST /personalize
```
### Request Body
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| vid | string | yes | Visitor token from `/session/start` or cookie. |
| outcome | string | yes | Value chosen in Quiz Q2 (see outcome matrix). |
| context_key | string | yes | One of: `grade`, `district`, `stage`, `students`, `capital`. |
| context_value | string | yes | Concrete value (e.g., `K-6`, `US-TX-Brownsville`, `MVP`, `25000`, `$100k-$1M`). |
| preview | boolean | no | If `true`, engine returns draft blocks (for CMS preview). |

### Response 200
```json
{
  "components": [
    {
      "id": "hero_block",
      "type": "markdown",
      "payload": "## Cut Class Time in Half…"
    },
    {
      "id": "before_after_chart",
      "type": "image",
      "payload": "/assets/brownsville_chart.png",
      "alt": "Percentile jump chart"
    },
    {
      "id": "call_to_action",
      "type": "cta",
      "payload": {
        "text": "Schedule a Demo",
        "url": "https://calendly.com/timeback/30min"
      }
    }
  ],
  "variant_id": "A",            // for A/B test logging
  "content_version": "2025-07-20T12:00:00Z",
  "ttl": 3600,                   // seconds to cache
  "latency_ms": 240
}
```
### Error Codes
| HTTP | Error | Meaning | Front-end Action |
|------|-------|---------|------------------|
| 400 | `invalid_context` | context_key/value pair unknown | Restart quiz |
| 401 | `invalid_vid` | Token not found/expired | Clear cookie → reload hero |
| 422 | `missing_fields` | Required quiz answer absent | Restart quiz |
| 503 | `provider_down` | Upstream LLM or DB unavailable | Render static fallback template ID matching `icp` |

### Latency Target
P90 ≤ **300 ms** server-side (allows 2.7 s for front-end render to hit 3 s budget).

---
## 3. `/profile?vid=…` – Fetch Stored Profile (Return Visitors)
```
GET /profile?vid=xyz
```
Returns minimal profile object:
```json
{ "email":"alex@example.com", "icp":"parent", "outcome":"raise_grades", "context_key":"grade", "context_value":"K-6" }
```
Used to pre-fill quiz or skip directly to `/personalize`.

Latency P90 ≤ **100 ms**.

---
## 4. `/agent/loop` – Server-Sent Events (Smart Tips Stream)
```
GET /agent/loop
```
### Purpose
Push personalised blocks or tip-ready notifications to the browser without polling. One-way stream (v1) keeps infra simple for the marketing site.

### Request
| Header / Param | Required | Example | Notes |
|----------------|----------|---------|-------|
| `Authorization: Bearer <JWT>` | yes | – | JWT `sub` = `vid`, `aud` = `agent_loop`, expires ≤15 m |
| `channels` query | no | `tasks,metrics` | comma-sep list; defaults to all |

### Events
SSE frames use the standard `id`, `event`, `data` fields.
| event | Payload JSON | Sent when |
|-------|-------------|-----------|
| `task_created` | Task object (see agent-action schema) | new task enqueued |
| `task_updated` | Task object (status change) | status flips |
| `metric_rollup` | `{ "trust_score":0.67 }` | nightly after roll-up |
| `ping` | – | keep-alive every 25 s |

### Reconnect & Back-off
Client resends `Last-Event-ID` to resume. Exponential back-off capped at 30 s.

### Error Codes
| HTTP | Error | Meaning | UI action |
|------|-------|---------|-----------|
| 401 | `invalid_jwt` | signature/exp invalid | refresh token → reconnect |
| 429 | `rate_limited` | too many connects | back-off |
| 503 | `provider_down` | agent offline | show toast, retry 30 s |

### Latency Target
First byte ≤100 ms P95; keep-alive ensures CF doesn’t close idle conns.

---
## 5. Security & Rate-Limiting
• All endpoints behind Cloudflare WAF.  
• `/session/start` limited to 5 requests / IP / min; `/personalize` limited to 30 / IP / min.  
• `tb_vid` cookie uses `Secure; HttpOnly; SameSite=Lax`.  
• CORS restricted to `https://timeback.com` and staging subdomains.  
• CAPTCHA (hCaptcha) can be enabled on `/session/start` for production traffic spikes.  
• JWT-signed `vid` planned for V2.

## 6. Fail-Over & Health
• Health endpoints: `/health/live` (always returns 200 when container up) and `/health/ready` (checks DB + LLM provider).  
• On `/personalize` 503, back-end may include `{"static_template_id":"parent_default"}` to guide front-end.  
• Static fallback `/static/{static_template_id}.html` rendered instantly.  
• Background job retries personalization and emails link when service recovers.

---
## 7. Open Questions
1. Should we embed A/B test variant in the response (`variant_id`)? **→ now included**  
2. Auth for internal preview (basic auth header vs signed JWT)?  
3. GDPR data-deletion endpoint `/profile/delete` needed for launch?  
4. Do we expose `ttl` in response to optimise CDN caching?

*Last updated*: 2025-07-20 

### Implementation Hint – React hook for SSE connection
```ts
import { useEffect, useRef, useState } from 'react';

export function useAgentLoop(jwt: string) {
  const [status, setStatus] = useState<'connecting'|'open'|'error'>('connecting');
  const lastId = useRef<string>('');
  const retryMs = useRef(1000);

  useEffect(() => {
    let es: EventSource | undefined;
    function connect() {
      const url = `/agent/loop${lastId.current ? `?lastEventId=${lastId.current}` : ''}`;
      es = new EventSource(url, { withCredentials:false });
      es.onopen = () => { setStatus('open'); retryMs.current = 1000; };
      es.onerror = () => { setStatus('error'); es?.close(); setTimeout(connect, retryMs.current); retryMs.current = Math.min(retryMs.current+1000, 30000); };
      ['task_created','task_updated','metric_rollup'].forEach(evt => {
        es!.addEventListener(evt, e => {
          lastId.current = (e as MessageEvent).lastEventId;
          // handleMessage(JSON.parse((e as MessageEvent).data));
        });
      });
    }
    connect();
    return () => es?.close();
  }, [jwt]);

  return { status };
}
```
--- 