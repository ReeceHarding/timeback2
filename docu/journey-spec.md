# TimeBack Website – MVP Journey Specification

This document defines the minimum-viable user journey for the July 31 website prototype. It covers the three interactive steps, copy placeholders, data captured, and component requirements.

## Overview
1. **Step 1 – Email Capture (Hero Screen)**
2. **Step 2 – 3-Click Quiz (Demographic + Pain-Point)**
3. **Step 3 – Personalized Landing Page**

All interactions must complete ≤ 3 seconds per screen and work on mobile first. No typing except the email field.

---
## Step 1 – Email Capture
```
┌───────────────────────────────────────────┐
│  Hero Headline                           │
│  Sub-headline                            │
│  [ big CTA button / email field ]        │
│  Trust badges (media logos)              │
└───────────────────────────────────────────┘
```
*Components*
• Headline: “Prove It Yourself: See Measurable Progress in One Week.”  
• Sub-headline: “Enter your email to unlock a personalized data-driven dashboard—takes 20 s, zero spam.”  
• Email input + CTA button (*copy*: “Unlock My Dashboard”)  
• Micro-copy under CTA: “You’ll view your custom insights immediately on the next page.”  
• Media trust bar (USA Today, Forbes, NBC, etc.)

*Data out*
```json
{
  "email": "string",
  "timestamp": "ISO8601"
}
```

*Edge cases*: missing @, duplicate email → inline error.

---
## Step 2 – 2-Question Quiz (dynamic)
After email capture, visitors answer just two taps. Question labels and options adapt to the role selected on the hero screen.

| Q# | Universal Label | Role-Specific Options | Maps To |
|----|-----------------|-----------------------|---------|
| 1  | I am a… | Parent, School Leader, Entrepreneur, Government Official, Philanthropist | ICP |
| 2  | What do you care about most right now? | *Dynamic list — see matrix below* | Outcome |
| 3  | Context question | *Dynamic by role — see matrix below* | Context |

> The quiz UI shows only two screens after email: **Outcome** then **Context**. The ICP is captured on the hero.

### Outcome Options Matrix
| Role | Outcome Options |
|------|-----------------|
| Parent | **"My child is falling behind and I'm worried about their grades."** (maps to `catch_up`) · **"I want to give my child a competitive edge for the future."** (maps to `competitive_edge`) · **"I'm concerned my child isn't learning *how* to learn effectively."** (maps to `life_skills`) |
| School Leader | Boost test scores · Reduce teacher burnout · Close achievement gaps |
| Entrepreneur | Lower customer-acquisition cost · Accelerate user onboarding · Differentiate with AI |
| Government | Improve national rankings · Cut per-student costs · Meet SDG targets |
| Philanthropist | Maximize social ROI · Reach underserved learners · Verify transparent data |

### Context Question Matrix
| Role | Question Copy | Options |
|------|--------------|---------|
| Parent | What grade level is your child in? | K-12 dropdown |
| School Leader | Which state/district are you in? | Auto-detect + search |
| Entrepreneur | What stage is your product at? | Idea · MVP · Scaling |
| Government | How many students are you responsible for? | <10k · 10-50k · 50k+ |
| Philanthropist | How much impact capital are you considering? | <$100k · $100k-$1M · $1M+ |

*Data out*
```json
{
  "icp": "parent|school|entrepreneur|government|philanthropist",
  "outcome": "string",
  "context_key": "grade|district|stage|students|capital",
  "context_value": "string",
  "quiz_time_ms": 8000
}
```

*UX notes*
• Progress indicator (●○ → ●●)  
• “Why we ask” tooltip explains relevance (<10 s)  
• Back button enabled; cannot skip without answer.

---
## Step 3 – Personalized Landing Page
Key modules (order weighted by ICP):
1. **Hero Block** – Familiar trope headline (`icp` + `pain_point` substitution).  
2. **Outcome Snapshot** – before/after chart or KPI dashboard aligned to ICP.
3. **Explainer Section** – 70 % familiar visuals (report-card makeover, etc.).
4. **Proof Pillars** – learning-science badges, case-study carousel.
5. **Primary CTA** – schedule call / wait-list signup (opens Calendly modal).

*API call*
Front-end sends combined payload `{email, icp, pain_point, region}` to `/personalize`. Response:
```json
{
  "headline": "string",
  "sub_blocks": [ {"type":"chart","data":{…}}, … ],
  "cta_text": "string"
}
```

*Success metric*: email-capture → personalized page load ≤ 3 s (P90).

---
## Return Visitor Logic
1. When a visitor completes the quiz, the server returns a `tb_vid` token (opaque, UUID) stored in a 1st-party cookie (`expires=180d`).
2. On subsequent visits:
   - If `tb_vid` is present, front-end calls `/profile?vid=…` and immediately renders the personalized dashboard—quiz skipped.
   - If cookie is missing but visitor re-enters a known email on the hero, server maps the email → vid, sets new cookie, skips quiz.
3. Personalized dashboard includes a small “Update my info” link that clears `tb_vid` and re-launches the 2-question quiz.
4. Email follow-ups can embed `?vid=` query param for one-click return on new devices.

*Privacy note:* Cookie stores no PII; deletion link in footer satisfies GDPR/CCPA.

---
## State Machine Snapshot
```
START → EmailScreen  --emailValid--> QuizQ1 → QuizQ2 → QuizQ3  --submit-->  PersonalizedPage → END
             |invalidEmail|                                      |API error|
             +------------+                                      v         
             errorMsg <---<-------------------------------- fallbackStatic
```

---
## Open Questions
1. Should we collect child’s grade level (parents only) as an optional Q4?  
2. Calendly vs in-house scheduler for CTA?  
3. Static template fallback per ICP lives in `/static/*.html` – confirm copy owner.

---
*Last updated:* 2025-07-20 