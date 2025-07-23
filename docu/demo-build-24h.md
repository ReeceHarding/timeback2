# 24-Hour Demo Build Plan  –  Parent ICP Happy-Path

*Goal:* By end of day **July 21** have a locally running demo that walks through:
1. Landing (email) → 2-question quiz → Parent dashboard.
2. All data is stubbed locally; no back-end or LLM.
3. Polished UI with Tailwind + shadcn/ui + Framer Motion.

---
## 0. Prereqs
```bash
pnpm i          # install deps in mockup/
cp .env.example .env.local   # dummy env vars
```

---
## 1. Landing Screen
- File: `src/pages/Landing.tsx`
- Elements:
  - Headline & sub-headline (risk-reversal copy).
  - Email `<Input>` component.
  - Hidden select preset to `parent`.
  - CTA `<Button>` ➜ `navigate('/quiz')`.
- Motion: fade-in on mount (`motion.div` opacity 0→1).
- Validation: simple regex; error toast.

---
## 2. Quiz Screen
- File: `src/pages/Quiz.tsx`
- In state: `email` from context.
### Q1 – Outcome
```ts
const outcomes = [
  {id: 'raise_grades', label: 'Raise my child’s grades fast'},
  {id: 'free_time', label: 'Free up after-school time'},
  {id: 'ai_safe', label: 'Protect against harmful AI shortcuts'},
];
```
- Radio-card buttons; on select, auto-advance with Framer slide-left.
### Q2 – Grade
- Simple `<Select>` list `K-1` … `K-12`.
- On submit → `navigate('/dash')` and push `{email, outcome, grade}` into context.

---
## 3. Dashboard Screen
- File: `src/pages/Dash.tsx`
- Mock JSON (import) representing `/personalize` response:
```json
[
  {"type":"markdown","payload":"## 31st → 84th percentile…"},
  {"type":"image","payload":"/brownsville_chart.png","alt":"Chart"},
  {"type":"cta","payload":{"text":"Schedule a Call","url":"#"}}
]
```
- Use `BlockRenderer` to map block types.
- Framer Motion stagger fade-in.

---
## 4. Global Setup
- `App.tsx` wraps `BrowserRouter` + `CookieGuard` (returns null for demo).
- `Context` provider holds `{email, outcome, grade}`.
- Tailwind config already present; import base styles.

---
## 5. Polish / QA Checklist
- Mobile viewport test (iPhone SE & Pixel).
- Ensure FCP <1.5 s via Lighthouse.
- All buttons keyboard accessible (`tabIndex`).

---
## 6. Build & Share
```bash
pnpm build
pnpm preview     # localhost:4173  demo link
```
Optionally run `npx serve dist` and expose via ngrok for quick stakeholder demo.

---
## Stretch (only if time)
- Add PostHog snippet with `capture_disabled=true` flag.
- Scroll-triggered Framer Motion on dashboard blocks. 