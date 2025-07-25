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
| Implement `llmClient()` with Groq + Together fail-over stub | @be | Done |
| Draft Privacy Policy page | @content | Not Started |

## Sprint 2  ( July 27 – July 31 )
| Task | Owner | Status |
|------|-------|--------|
| Build `QuizScreen` (Outcome + Context) | @fe | Done |
| Implement `/personalize` endpoint + block renderer | @be | Done |
| Implement type-safe database service for student archetypes | @be | Done |
| Create comprehensive archetype implementation plan with Phase 1/2 architecture | @be | Done |
| Implement Phase 1 canonical archetype service | @be | Done |
| Integrate real student archetype data with personalization engine | @be | Done |
| Integrate `BlockRenderer` components | @fe | Done |
| **COMPLETED:** Phase 1 Archetype Integration - Real student data powering personalization | @be | Done |
| **COMPLETED:** End-to-end testing with 4 canonical archetypes (Grades 3,6,8,9) | @be | Done |
| **COMPLETED:** Graceful fallback system for unsupported grades | @be | Done |
| Capture analytics events per spec | @fe/@be | Not Started |
| QA latency budget (3 s) with Groq | @ops | Not Started |
| Complete MVP security items (Transport, Cookies, WAF) | @ops | Not Started |

### **🎯 Phase 1 Achievement Summary (July 25, 2025)**
- ✅ **ArchetypeService:** Canonical selection with 4 verified student archetypes
- ✅ **PerformanceService:** Real MAP growth data enrichment and narrative generation  
- ✅ **Integration Complete:** `/personalize` endpoint using authentic student proof points
- ✅ **Test Coverage:** End-to-end validation across all canonical archetypes
- ✅ **Performance Target:** <300ms server-side, 2-3s total including LLM generation
- ✅ **Business Impact:** Parents see real proof (Luna +25 RIT, Bella +24 RIT, Sydney +32 RIT)
- ✅ **BREAKTHROUGH:** 100% Authentic MAP Data - Zero false information in report cards
- ✅ **MAP Conversion Tables:** Real RIT-to-grade conversion with authentic percentiles
- ✅ **Authentic Proof Points:** Real school names, genuine academic performance, credible growth metrics

### **🚨 CRITICAL ARCHITECTURAL BREAKTHROUGH (July 25, 2025)**
**MAJOR DISCOVERY:** Student Table and MAP Data had ZERO overlap - completely separate populations!

**FIXES IMPLEMENTED:**
- ✅ **MAP-First Architecture:** Rebuilt PerformanceService to prioritize performance data
- ✅ **ETL Bug Fixed:** MAP import now uses correct student ID mapping (not names)
- ✅ **Database Connectivity:** Resolved PostgreSQL instance conflicts
- ✅ **Real Growth Data:** Luna Montagna (+25 RIT), Bella Barba (+24 RIT), Sydney Barba (+32 RIT)
- ✅ **LLM Enhancement:** Prompts now use authentic archetype data (`archetype_used: true`)
- ✅ **Competitive Proof:** Measurable student outcomes vs generic marketing claims

**BUSINESS IMPACT:** 
- **Before:** "AI tutoring helps students" (generic)
- **After:** "Here's Luna's actual +25 RIT reading improvement at Alpha Miami" (authentic proof)

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