# TimeBack Website – Front-End Implementation Plan

_Scope: all guidelines in this doc apply to the **public marketing site (timeback.com)** and its hyper-personalized landing pages._

*Last updated 2025-07-20*

## 1. Tech Stack
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Framework | React 18 + Vite | Already scaffolded in `mockup/`, fast dev HMR |
| Styling | TailwindCSS | Utility-first, quick responsive tweaking |
| State | React Context (vid + profile) | Lightweight; no Redux needed |
| Router | React Router v6 | Nested routes + code-splitting |
| Data Fetch | `fetch()` + SWR | Simple auto-revalidate, supports cache TTL |
| Component Lib | shadcn/ui base | Matches existing mockup components |
| Animation | Framer Motion | Lightweight (~25 KB gzip); adds high-polish transitions |

## 2. Route Map
| Path | Component | Data Source |
|------|-----------|-------------|
| `/` | `HeroScreen` | local state; sends `/session/start` |
| `/quiz` | `QuizScreen` | Context; on submit calls `/personalize` |
| `/dash` | `DashboardScreen` | Renders blocks from `/personalize` |
| `/static/:templateId` | `StaticTemplate` | Pre-rendered HTML fallback |
| `/*` | `NotFound` | 404 |

## 3. Component Breakdown
| Component | Responsibility | API Types Rendered |
|-----------|----------------|--------------------|
| `HeroScreen` | Email + role cards, sets cookie | – |
| `QuizScreen` | Outcome + Context Qs, progress dots | – |
| `BlockRenderer` | Map block `type` → React element | markdown · image · cta · component |
| `MarkdownBlock` | Render sanitized MD via `react-markdown` | markdown |
| `ImageBlock` | Lazy-load responsive img | image |
| `CTABlock` | Styled button + link | cta |
| `ComponentBlock` | Dynamic import of custom chart components | component |
| `CookieGuard` | Reads `tb_vid`; redirects return users | – |

## 4. Cookie & Auth Flow
1. On app load, `CookieGuard` reads `tb_vid`.
2. If present → `GET /profile` → redirect to `/dash` after `POST /personalize`.
3. If absent → show `HeroScreen`.
4. Cookie flags: `Secure; HttpOnly = false` (JS needs to read), `SameSite=Lax`.

## 5. Data Fetch Patterns
```
POST /session/start   → store vid in cookie, ctx in React context
POST /personalize     → SWR cache 1h (uses ttl from response)
GET  /profile          → pre-fill quiz or skip
```
Error handling: On 503 `provider_down` → navigate to `/static/{static_template_id}`.

## 6. Performance Budget
| Metric | Target |
|--------|--------|
| First Contentful Paint | <1.5 s (3G) |
| Interactivity (Hydrate) | <100 ms |
| `/personalize`→dash render | <2.7 s |

Optimization tactics
• Code-split each route, lazy-load `BlockRenderer` children.  
• Use `react-markdown` remark-gfm only when a MD block is present.  
• Pre-connect to LLM provider domain.

## 7. Accessibility & i18n MVP
• WCAG AA colors via Tailwind palette.  
• `aria-live` announcements on page transitions.  
• Only EN copy at launch; wrap strings with central `t()` for future i18n.

## 8. Dev & Build
```
# dev
pnpm dev

# prod build
pnpm build && pnpm preview
```
CI: GitHub Actions → run `pnpm lint && pnpm test && pnpm build`.

## 9. Open Questions
- (resolved) Animation library → **Framer Motion selected**
1. Do we need offline fallback (PWA) for low-bandwidth parents?  
2. Should static templates live in `public/static` or served from CDN edge? 

## 10. Smart Tips Feed (Visitor-Facing)

### Purpose
Give returning visitors lightweight, contextual nudges ("Smart Tips") without cluttering the main personalised page.

### Connection Flow
```
<CookieGuard> ▶ useAgentLoop() ── SSE ── /agent/loop
```

### Component States
| State | When | UI |
|-------|------|----|
| `connecting` | initial EventSource open | skeleton list shimmer |
| `feed` | ≥1 event received | Tip cards list |
| `empty` | 5 s, no events | illustration + “Agent is thinking…” |
| `error` | SSE error / 401 | toast + retry spinner |

### Card Anatomy
```
┌────────────┐
│  [Icon] Bold summary line
│  Faded rationale (auto-collapses after 8 s)
│  [Accept]  [Ignore]
└────────────┘
```
Framer Motion handles fade + collapse; `aria-expanded` toggled for accessibility.

### Analytics Hook
Emit `tip_view` when card visible (IntersectionObserver ≥50 %). Emit `tip_action` on Accept/Ignore.

### Performance
Virtualize list >50 items, keep DOM ≤100 nodes. SSE payloads capped at 2 KB each, fits 5 KB/s worst-case.

--- 