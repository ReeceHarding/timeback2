# TimeBack Website – Analytics & A/B Experiment Specification

*Last updated: 2025-07-20*

## 1. Tracking Stack
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Event collector | PostHog Cloud | Self-host option later; built-in A/B, funnels, GDPR tools |
| Client lib | `@posthog/posthog-js` | 8 KB gzip, supports autocapture off |
| Server events | Node SDK via API Gateway | Logs personalization latency, provider stats |
| Dashboarding | PostHog Insights + Google Data Studio export | Fast exploratory + shareable exec views |

## 2. Core Events
| Event Name | Trigger | Properties |
|------------|---------|------------|
| `hero_email_submit` | Visitor hits CTA on hero | `icp`, `email_hash` (SHA-256), `variant_id` |
| `session_start_success` | 200 from `/session/start` | `new_user` (bool), `duplicate` (bool) |
| `quiz_answer` | Each tap in Quiz | `question_id`, `answer_value`, `time_ms` |
| `quiz_complete` | All quiz Qs answered | `quiz_time_ms`, `outcome`, `context_key`, `context_value` |
| `personalize_request` | POST `/personalize` sent | `vid`, `variant_id` |
| `personalize_success` | 200 `/personalize` | `latency_ms`, `provider`, `block_count`, `ttl` |
| `personalize_fail` | 503 or error | `error_code`, `provider` |
| `cta_click` | Any CTA block clicked | `cta_text`, `cta_url`, `vid` |
| `video_play` | HTML5 video starts playing | `video_id`, `position_s` (0 for first play) |
| `video_pause` | Visitor pauses video | `video_id`, `position_s` |
| `video_complete` | Playback reaches 95 % | `video_id`, `duration_s` |
| `video_exit_viewport` | Video leaves viewport before completion | `video_id`, `played_s`, `position_pct` |
| `scroll_depth` | First time visitor scrolls past 25/50/75/90 % of page height | `percent` (25\|50\|75\|90) |
| `static_fallback_render` | `/static/:templateId` displayed | `template_id`, `reason` |
| `hydration_ms` | React hydration callback | `value` (ms) |

### Implementation Hints – Front-End Event Hooks (MVP)

```ts
// Scroll depth (25/50/75/90 %) – add once in a top-level React effect
useEffect(() => {
  const fired: Record<number, boolean> = {};
  const marks = [25, 50, 75, 90];
  const handler = () => {
    const pct = ((window.scrollY + window.innerHeight) / document.body.scrollHeight) * 100;
    marks.forEach(m => {
      if (!fired[m] && pct >= m) {
        fired[m] = true;
        posthog.capture('scroll_depth', { percent: m });
      }
    });
  };
  window.addEventListener('scroll', handler, { passive: true });
  return () => window.removeEventListener('scroll', handler);
}, []);

// Video interaction (play / pause / complete / exit viewport)
function trackVideo(videoEl: HTMLVideoElement, videoId: string) {
  function send(event: string, extra = {}) {
    posthog.capture(event, { video_id: videoId, ...extra });
  }
  videoEl.addEventListener('play', () => send('video_play', { position_s: 0 }));
  videoEl.addEventListener('pause', () => send('video_pause', { position_s: videoEl.currentTime }));
  videoEl.addEventListener('ended', () => send('video_complete', { duration_s: videoEl.duration }));

  const obs = new IntersectionObserver(([entry]) => {
    if (!entry.isIntersecting && videoEl.currentTime / videoEl.duration < 0.95) {
      send('video_exit_viewport', {
        played_s: Math.round(videoEl.currentTime),
        position_pct: Math.round((videoEl.currentTime / videoEl.duration) * 100)
      });
    }
  }, { threshold: 0 });
  obs.observe(videoEl);
}
```

---

Notes
• `email_hash` is SHA-256 of lower-cased email to avoid sending raw PII.  
• Auto-capture and session recording disabled by default; enable only with consent.

## 3. A/B Testing Framework
### Variant assignment
1. On first load, `posthog.getFeatureFlag("hero_copy")` returns `'A' | 'B' | null`.  
2. Library uses PostHog Flags targeting 50/50 split; stored in cookie (30 d).  
3. Front-end passes `variant_id` to `/personalize` so content generation can match copy.

### Experiment 01 – Hero Headline
| Variant | Headline | Sub-headline |
|---------|----------|--------------|
| A | “Prove It Yourself: See Measurable Progress in One Week.” | “Enter your email to unlock a personalized dashboard—20 s, zero spam.” |
| B | “Cut Class Time in Half. Double the Results.” | “Get a data-backed plan in seconds.” |

Success metric = email-capture rate (`hero_email_submit` / unique visitors).  
Run length: 5 000 visitors per variant (~1 day at projected traffic) or 95 % stat-sig.

## 4. Funnel & Retention
Primary MVP funnel
```
Visited   → hero_email_submit → quiz_complete → personalize_success → cta_click
```
Goals
• 25 %+ hero → email conversion  
• 90 % quiz completion  
• <3 % `/personalize` failure rate  
• 5 %+ cta_click

Retention cohort
• `personalize_success` users returning within 7 days (token cookie).

## 5. Latency & Provider Dashboard
Collected via server events every `/personalize` request.
| Metric | Target | Alert |
|--------|--------|-------|
| P90 latency | <300 ms | Slack alert if >350 ms for 5 min |
| Provider error rate | <1 % | Auto switch to secondary provider |

## 6. Privacy & Consent
• Banner copy: “We use cookies to personalize content and measure conversion. By using the site you agree to our Privacy Policy.”  
• On “Reject All,” store opt-out flag; client lib disabled (`posthog.optOutCapturing()`).  
• GDPR delete: `/profile/delete` endpoint removes email & events; link in privacy page.

## 7. Agent Interaction Trace & Metrics (Marketing Site)

### 7.1 `interaction_trace` – raw event log
| Column | Type | Description |
|--------|------|-------------|
| `trace_id` | bigint pk | auto-increment |
| `vid` | uuid | visitor id |
| `task_id` | uuid null | fk → `tasks.task_id` |
| `event_type` | text | `task_created` · `task_updated` · `tip_accepted` · `tip_ignored` |
| `details` | jsonb | event-specific payload |
| `ts` | timestamptz | event time |

Indexes
* `(vid, ts DESC)` – session timeline
* GIN on `details` for ad-hoc queries

### 7.2 `daily_agent_metrics` – roll-ups
| Column | Type | Notes |
|--------|------|-------|
| `metric_date` | date pk |
| `vid` | uuid pk |
| `tips_viewed` | int |
| `tips_accepted` | int |
| `trust_score` | numeric(5,2) | tips_accepted / max(tips_viewed,1) |
| `demo_clicks` | int | forward-fill from `cta_click` events |

Nightly job outline
1. Select previous day’s `interaction_trace` rows.
2. Aggregate counts per vid.
3. Compute `trust_score`.
4. Upsert into `daily_agent_metrics`.

Data retention: raw trace 90 d, roll-ups 2 y.

## 8. Open Questions
- (future work) Session replay behind explicit consent?  
- (future work) Additional experiment: quiz order vs. single page? 