# Student Archetype Integration - Implementation Plan

**Version:** 1.0  
**Date:** July 24, 2025  
**Purpose:** Iterative plan to connect real student data to personalization pipeline

---

## 🎯 **Objective**

**SUPERSEDED:** This document has been superseded by the comprehensive `archetype-implementation-plan.md` which provides detailed technical specifications, extensible architecture design, and Phase 1/Phase 2 migration strategy.

**Original Objective:** Connect the working dashboard service to the `/personalize` endpoint so that real Alpha School student performance data powers the AI-generated narratives instead of synthetic examples.

**Current Approach:** Two-phase extensible implementation starting with canonical archetypes (MVP) and evolving to dynamic selection (enhancement).

---

## 📋 **Implementation Phases**

### **Phase 1: Data Enrichment Foundation** 
**Goal:** Create service to extract rich performance stories from our data

**Tasks:**
1. **Create `PerformanceService`** (`backend/src/services/performance.service.ts`)
   - Query MAP growth data from `entity_version` table
   - Query assessment results for percentile improvements
   - Calculate growth narratives and achievement highlights
   - Return structured `PerformanceStory` object

2. **Define TypeScript Interfaces** (add to `backend/src/types/db.d.ts`)
   - `PerformanceStory` interface
   - `EnrichedStudentArchetype` interface
   - Supporting types for MAP growth data

**Validation:** Service returns real growth stories for test student IDs

### **Phase 2: Dashboard Service Enhancement**
**Goal:** Upgrade existing dashboard service to return enriched archetypes

**Tasks:**
1. **Modify `DashboardService.getArchetypeStudent()`**
   - Integrate with `PerformanceService`
   - Change return type to `EnrichedStudentArchetype`
   - Add error handling for missing performance data
   - Implement fallback to basic archetype if enrichment fails

**Validation:** Test endpoint returns complete archetype with performance story

### **Phase 3: LLM Integration**
**Goal:** Update AI prompts to use real student data

**Tasks:**
1. **Enhance `getPersonalizedContent()` in `llmClient.ts`**
   - Add `archetypeStudent?: EnrichedStudentArchetype` parameter
   - Modify prompts to include real performance data
   - Replace synthetic chart data with authentic growth stories
   - Ensure privacy compliance (no real names)

**Validation:** AI generates narratives using real RIT scores and percentile jumps

### **Phase 4: Endpoint Integration**
**Goal:** Complete the connection in `/personalize` endpoint

**Tasks:**
1. **Update `/personalize` endpoint** (`backend/src/index.ts`)
   - Call enhanced `dashboardService.getArchetypeStudent()`
   - Pass enriched archetype to `getPersonalizedContent()`
   - Handle graceful fallback when real data unavailable

**Validation:** End-to-end test from quiz → real data narrative

---

## 🔄 **Iterative Development Strategy**

### **Iteration 1: Minimum Viable Integration**
- Basic performance service with MAP data only
- Simple enhancement to dashboard service
- One real data point in LLM prompts
- Test with Grade 9 archetype (most data available)

### **Iteration 2: Full Feature Set**  
- Complete performance enrichment with all data sources
- Enhanced error handling and fallbacks
- Real data across all narrative blocks
- Test with all available grades (3, 6, 9)

### **Iteration 3: Production Polish**
- Performance optimization (sub-100ms data retrieval)
- Comprehensive error handling
- A/B testing real vs synthetic content
- Monitoring and analytics

---

## 🛡️ **Risk Mitigation**

### **Technical Risks**
- **Data Availability**: Not all students may have complete MAP data
  - *Solution*: Graceful fallback to synthetic approach
- **Performance**: Complex queries may slow personalization
  - *Solution*: Optimize queries, add caching if needed
- **Privacy**: Accidental exposure of student identifiers
  - *Solution*: Anonymization validation in each phase

### **Business Risks**
- **Authenticity vs Privacy**: Balance real data with anonymization
  - *Solution*: Use program context, not individual identifiers
- **Data Staleness**: Old student data may reduce credibility
  - *Solution*: Include data freshness indicators

---

## 📊 **Success Criteria**

### **Phase Completion Gates**
1. **Phase 1**: Service returns structured performance story for test student
2. **Phase 2**: Dashboard service provides enriched archetype with real growth data
3. **Phase 3**: LLM generates narrative using real RIT scores and percentiles  
4. **Phase 4**: Parent completes quiz → sees dashboard with authentic case study

### **Quality Metrics**
- **Data Authenticity**: >80% of narratives use real student data
- **Performance**: <300ms total personalization time (existing target)
- **Privacy Compliance**: 0% real student names exposed
- **Fallback Success**: 100% uptime even when enrichment fails

---

## 🚀 **Next Steps**

### **Immediate Actions**
1. **Start Phase 1**: Create `PerformanceService` with basic MAP data query
2. **Validate Data**: Confirm we have MAP growth data for our test grades
3. **Define Interfaces**: Add TypeScript types for enriched archetypes
4. **Plan Testing**: Set up test cases for each grade level

### **Decision Points**
- **Data Scope**: How much performance detail to include initially?
- **Privacy Level**: How specific can we be while maintaining anonymization?
- **Fallback Strategy**: Synthetic data vs static templates when real data fails?

---

*This plan ensures we build incrementally while maintaining system reliability and moving toward our competitive advantage of real proof points.* 