# TimeBack Website – Sprint Roadmap

*Created: 2025-07-20*

## Key Dates
| Milestone | Date | Definition of Done |
|-----------|------|--------------------|
| MVP ready for guide training | **July 31** | Email capture + 2-question quiz + personalized dashboard running on Groq; static fallback works; basic analytics live |
| Production launch | **Aug 7** | Fail-over provider wired, security checklist items “Pending” closed, mobile QA pass, legal pages live |
| Parent notification campaign | **Aug 14** | Copy reviewed, email + landing links point to production |

## Team Roles (Tentative)
| Role | Name | Handle |
|------|------|--------|
| Product / PM | Timothy | @tim |
| Front-end Lead | TBD | @fe |
| Back-end Lead | TBD | @be |
| Infra / DevOps | TBD | @ops |
| Content Ops | TBD | @content |
| Analytics / Growth | TBD | @data |

## Sprint 1  ( July 20 – July 26 )
| Task | Owner | Status |
|------|-------|--------|
| Build `HeroScreen` + email + role cards | @fe | Done |
| Implement `/session/start` endpoint | @be | Done |
| Set up PostHog project + feature flag `hero_copy` | @data | Not Started |
| Create initial content files for parents (`content/parents/*.md`) | @content | Not Started |
| Provision Groq API keys in env | @ops | Done |
| Implement `llmClient()` with Groq + Together fail-over stub | @be | In-Progress |
| Draft Privacy Policy page | @content | Not Started |

## Sprint 2  ( July 27 – July 31 )
| Task | Owner | Status |
|------|-------|--------|
| Build `QuizScreen` (Outcome + Context) | @fe | Done |
| Implement `/personalize` endpoint + block renderer | @be | In-Progress |
| Integrate `BlockRenderer` components | @fe | Done |
| Capture analytics events per spec | @fe/@be | Not Started |
| QA latency budget (3 s) with Groq | @ops | Not Started |
| Complete MVP security items (Transport, Cookies, WAF) | @ops | Not Started |

## Sprint 3  ( Aug 1 – Aug 7 )
| Task | Owner | Status |
|------|-------|--------|
| Fail-over to Together AI + health checks | @be/@ops | Not Started |
| Polish Framer Motion transitions | @fe | In-Progress |
| Responsive/mobile QA + accessibility audit | @fe | Not Started |
| Session replay consent UI (future work flag) | @fe | Deferred |
| Load testing `/personalize` 1 req/s peak | @ops | Not Started |
| PostHog dashboard for exec review | @data | Not Started |

## Post-Launch Hardening  ( Aug 8 – Aug 14 )
| Task | Owner | Status |
|------|-------|--------|
| Implement `/profile/delete` endpoint | @be | Future |
| Begin A/B test “Quiz order” (future work) | @data | Future |
| Vendor security reviews completed | @ops | Future |

---
Legend: **Not Started** / In-Progress / Done / Future / Deferred 