# Student Archetype Implementation Plan
**Version:** 1.0  
**Date:** July 24, 2025  
**Status:** Approved for Implementation  
**MVP Deadline:** July 31, 2025

---

## 🎯 **Executive Summary**

This document defines the comprehensive implementation plan for connecting real Alpha School student data to the TimeBack personalization engine. The approach uses a **two-phase extensible architecture** that meets MVP deadlines while enabling sophisticated enhancements post-launch.

**Phase 1 (MVP):** Pre-validated canonical archetypes with extensible foundation  
**Phase 2 (Enhancement):** Dynamic selection with geographic/contextual matching  

---

## 📐 **Technical Architecture Specification**

### **Extensible Service Interface**
All phases use identical API contract to ensure frontend compatibility:

```typescript
interface ArchetypeService {
  getArchetypeStudent(grade: string, context?: SelectionContext): Promise<EnrichedStudentArchetype>
  getAvailableGrades(): Promise<string[]>
  validateArchetypeAvailability(grade: string): Promise<boolean>
}

interface SelectionContext {
  parentConcerns?: string[];        // Phase 2: ['motivation', 'tough_subjects'] 
  geographicRegion?: string;        // Phase 2: 'US-West', 'International'
  schoolType?: 'traditional' | 'alternative' | 'international';
  previousVisit?: boolean;          // Phase 2: For rotation logic
  sessionId?: string;               // Phase 2: For consistency tracking
}
```

### **Archetype Registry Data Structure**
Designed for Phase 1 simplicity with Phase 2 enhancement hooks:

```typescript
interface ArchetypeRegistry {
  [grade: string]: {
    canonical: string;              // Phase 1: Primary archetype ID
    alternates: string[];           // Phase 2: Additional options  
    selectionCriteria?: {           // Phase 2: Smart selection rules
      international?: string;
      mathFocus?: string;
      readingFocus?: string;
      traditional?: string;
      alternative?: string;
    };
    metadata: {
      lastUpdated: Date;
      dataQuality: 'high' | 'medium' | 'low';
      growthStoryAvailable: boolean;
    };
  };
}
```

### **Performance Data Model**
Structured for immediate use and future enhancement:

```typescript
interface EnrichedStudentArchetype {
  // Core identification (Phase 1)
  student_id: string;
  first_name: string | null;       // Always anonymized
  last_name: string | null;        // Always anonymized  
  grade: string;
  school_name: string;
  district_name: string;
  
  // Performance narrative (Phase 1)
  performance_story: {
    map_growth: MapGrowthData[];
    academic_context: AcademicContext;
    achievement_highlights: AchievementHighlights;
    narrative_summary: string;     // AI-ready description
  };
  
  // Selection metadata (Phase 2 ready)
  selection_metadata?: {
    selection_reason: 'canonical' | 'geographic_match' | 'subject_focus' | 'rotation';
    alternatives_available: number;
    confidence_score: number;      // 0-1 quality score
    fallback_used: boolean;       // Error handling tracking
  };
  
  // Cache control
  cache_ttl: number;               // Seconds to cache
  generated_at: Date;
}
```

---

## 🚀 **Phase 1 Implementation (MVP)**

### **Canonical Archetype Selection**
Based on manual analysis of rawdata, these archetypes provide compelling, verified growth stories:

```typescript
const CANONICAL_ARCHETYPES = {
  "Grade 3": {
    canonical: "0741",  // Luna Montagna - Reading: 176→190→201 (+25 points)
    school: "Alpha Miami",
    story_focus: "reading_breakthrough",
    growth_highlights: ["Fall to Spring: 25-point RIT improvement", "Consistent seasonal progress"]
  },
  
  "Grade 6": {
    canonical: "0717",  // Bella Barba - Math: 220→232→244 (+24 points)  
    school: "Alpha Miami",
    story_focus: "math_acceleration", 
    growth_highlights: ["Winter to Spring: 12-point improvement", "Strong cross-subject performance"]
  },
  
  "Grade 8": {
    canonical: "0734",  // Sydney Barba - Math: 220→244→252 (+32 points)
    school: "Alpha Miami", 
    story_focus: "exceptional_growth",
    growth_highlights: ["Exceptional 32-point math improvement", "Reading growth: 235→249"]
  },
  
  "Grade 9": {
    canonical: "018-0000", // Valiant Leonides Lopez  
    school: "Irvington High School",
    story_focus: "traditional_school_success",
    growth_highlights: ["Traditional high school environment", "US domestic context"]
  }
};
```

### **Phase 1 Service Implementation**
```typescript
class ArchetypeService {
  constructor(
    private performanceService: PerformanceService,
    private config: ArchetypeConfig = DEFAULT_CONFIG
  ) {}
  
  async getArchetypeStudent(grade: string, context?: SelectionContext): Promise<EnrichedStudentArchetype> {
    // Phase 1: Simple canonical lookup
    const studentId = this.getCanonicalArchetype(grade);
    
    if (!studentId) {
      throw new ArchetypeNotFoundError(`No archetype available for grade: ${grade}`);
    }
    
    // Enrich with performance data
    const enrichedArchetype = await this.performanceService.enrichStudent(studentId);
    
    // Add selection metadata
    enrichedArchetype.selection_metadata = {
      selection_reason: 'canonical',
      alternatives_available: 0,  // Phase 1: No alternates
      confidence_score: 1.0,      // Phase 1: High confidence
      fallback_used: false
    };
    
    return enrichedArchetype;
  }
  
  private getCanonicalArchetype(grade: string): string | null {
    return CANONICAL_ARCHETYPES[grade]?.canonical || null;
  }
  
  // Phase 2: Add this method without changing existing code
  private getSmartArchetype(grade: string, context: SelectionContext): string {
    // Implementation ready for Phase 2
    return this.applySelectionCriteria(grade, context);
  }
}
```

### **Performance Enrichment Service**
```typescript
class PerformanceService {
  async enrichStudent(studentId: string): Promise<EnrichedStudentArchetype> {
    // Get basic student data
    const student = await this.getStudentBasicData(studentId);
    
    // Get MAP growth data  
    const mapGrowth = await this.getMapGrowthData(studentId);
    
    // Get assessment results
    const assessmentData = await this.getAssessmentData(studentId);
    
    // Generate performance narrative
    const performanceStory = this.generatePerformanceStory(mapGrowth, assessmentData);
    
    return {
      ...student,
      performance_story: performanceStory,
      cache_ttl: 3600,  // 1 hour cache
      generated_at: new Date()
    };
  }
  
  private async getMapGrowthData(studentId: string): Promise<MapGrowthData[]> {
    const result = await query(`
      SELECT json_payload 
      FROM entity_version 
      WHERE entity_type = 'map-snapshot' 
      AND entity_pk LIKE $1
      AND superseded = false
    `, [`${studentId}|%`]);
    
    return this.parseMapGrowthData(result.rows);
  }
}
```

---

## 🔄 **Phase 2 Enhancement Plan**

### **Enhanced Selection Logic**
```typescript
// Phase 2: Add to ArchetypeService without breaking existing code
private applySelectionCriteria(grade: string, context: SelectionContext): string {
  const registry = ENHANCED_ARCHETYPE_REGISTRY[grade];
  
  if (!context || !registry.selectionCriteria) {
    return registry.canonical; // Fallback to Phase 1 behavior
  }
  
  // Geographic matching
  if (context.geographicRegion === 'International' && registry.selectionCriteria.international) {
    return registry.selectionCriteria.international;
  }
  
  // Subject-specific concerns
  if (context.parentConcerns?.includes('math') && registry.selectionCriteria.mathFocus) {
    return registry.selectionCriteria.mathFocus;
  }
  
  // School type preference
  if (context.schoolType === 'alternative' && registry.selectionCriteria.alternative) {
    return registry.selectionCriteria.alternative;
  }
  
  // Rotation for repeat visitors
  if (context.previousVisit && registry.alternates.length > 0) {
    return this.rotateArchetype(registry, context.sessionId);
  }
  
  // Default to canonical
  return registry.canonical;
}
```

### **Enhanced Registry (Phase 2)**
```typescript
const ENHANCED_ARCHETYPE_REGISTRY = {
  "Grade 6": {
    canonical: "0717",  // Bella Barba (Alpha Miami)
    alternates: ["angel_muswege", "joyance_kizungu"],
    selectionCriteria: {
      international: "angel_muswege",     // Malawi perspective
      mathFocus: "0717",                  // Strong math growth
      traditional: "0717",                // Alpha Miami  
      alternative: "angel_muswege"        // International approach
    },
    metadata: {
      lastUpdated: new Date('2025-07-24'),
      dataQuality: 'high',
      growthStoryAvailable: true
    }
  }
  // ... other grades
};
```

---

## 🗃️ **Database Schema Evolution**

### **Phase 1 Requirements**
Existing schema supports Phase 1 without changes:
- ✅ `student_current_v` materialized view for basic student data
- ✅ `entity_version` table with MAP and assessment data
- ✅ Existing query patterns support enrichment service

### **Phase 2 Database Enhancements**
```sql
-- Add archetype selection tracking
CREATE TABLE IF NOT EXISTS archetype_selections (
    id SERIAL PRIMARY KEY,
    session_vid UUID REFERENCES sessions(vid),
    grade VARCHAR(50),
    selected_student_id VARCHAR(255),
    selection_reason VARCHAR(100),
    alternatives_considered TEXT[],
    context_data JSONB,
    selected_at TIMESTAMPTZ DEFAULT NOW()
);

-- Add performance for selection queries
CREATE INDEX IF NOT EXISTS idx_archetype_selections_grade ON archetype_selections(grade);
CREATE INDEX IF NOT EXISTS idx_archetype_selections_session ON archetype_selections(session_vid);

-- Add archetype performance tracking
CREATE TABLE IF NOT EXISTS archetype_performance (
    student_id VARCHAR(255),
    grade VARCHAR(50),
    data_quality_score DECIMAL(3,2),
    growth_story_completeness DECIMAL(3,2),
    last_verified TIMESTAMPTZ,
    PRIMARY KEY (student_id, grade)
);
```

---

## 📊 **Testing Strategy**

### **Phase 1 Testing Requirements**
1. **Unit Tests:**
   ```typescript
   describe('ArchetypeService', () => {
     it('returns canonical archetype for valid grade', async () => {
       const archetype = await service.getArchetypeStudent('Grade 6');
       expect(archetype.student_id).toBe('0717');
       expect(archetype.performance_story).toBeDefined();
     });
     
     it('throws error for invalid grade', async () => {
       await expect(service.getArchetypeStudent('Grade 99')).rejects.toThrow(ArchetypeNotFoundError);
     });
   });
   ```

2. **Integration Tests:**
   - End-to-end flow: quiz → archetype selection → enrichment → LLM generation
   - Database connectivity and query performance
   - Error handling and fallback scenarios

3. **Performance Tests:**
   - Sub-100ms archetype retrieval target
   - Sub-300ms total enrichment target  
   - Load testing with 10x expected traffic

### **Phase 2 Testing Requirements**
1. **Selection Logic Tests:**
   - Geographic matching accuracy
   - Subject-specific selection validation
   - Rotation logic for repeat visitors
   - Fallback to canonical behavior

2. **A/B Testing Framework:**
   - Canonical vs. smart selection performance
   - Conversion rate comparisons
   - User satisfaction metrics

---

## 📈 **Performance Monitoring Plan**

### **Phase 1 Metrics**
```typescript
interface ArchetypeMetrics {
  // Performance metrics
  average_retrieval_time: number;     // Target: <100ms
  cache_hit_rate: number;             // Target: >80%
  error_rate: number;                 // Target: <1%
  
  // Business metrics  
  archetype_coverage: number;         // % of grades with available archetypes
  data_quality_score: number;         // Completeness of performance stories
  
  // Usage patterns
  most_requested_grades: string[];
  peak_usage_times: Date[];
}
```

### **Phase 2 Enhanced Metrics**
```typescript
interface EnhancedArchetypeMetrics extends ArchetypeMetrics {
  // Selection effectiveness
  selection_accuracy: number;         // User preference validation
  rotation_effectiveness: number;     // Repeat visitor engagement
  geographic_match_success: number;   // Relevance scoring
  
  // System complexity
  selection_criteria_usage: Record<string, number>;
  fallback_frequency: number;
  alternative_utilization: number;
}
```

---

## 🛡️ **Error Handling & Fallback Strategy**

### **Hierarchical Fallback System**
```typescript
class ArchetypeService {
  async getArchetypeStudent(grade: string, context?: SelectionContext): Promise<EnrichedStudentArchetype> {
    try {
      // Primary: Smart selection (Phase 2) or canonical (Phase 1)
      return await this.getPreferredArchetype(grade, context);
    } catch (error) {
      try {
        // Fallback 1: Canonical archetype (always reliable)
        return await this.getCanonicalArchetype(grade);
      } catch (error) {
        try {
          // Fallback 2: Default synthetic archetype
          return await this.getSyntheticArchetype(grade);
        } catch (error) {
          // Fallback 3: Static template response
          throw new ArchetypeServiceUnavailableError('All archetype services unavailable');
        }
      }
    }
  }
}
```

### **Graceful Degradation Strategy**
1. **Smart Selection Failure** → Fall back to canonical archetype
2. **Database Unavailable** → Return cached archetype or synthetic data
3. **Performance Enrichment Failure** → Return basic student data with note
4. **Complete Service Failure** → Frontend displays static template

---

## 🚦 **Deployment & Migration Strategy**

### **Phase 1 Deployment**
1. **Database Preparation:**
   - Verify student data availability for canonical archetypes
   - Run data quality checks on MAP growth data
   - Test query performance under load

2. **Service Deployment:**
   - Deploy PerformanceService with basic enrichment
   - Deploy ArchetypeService with canonical selection only  
   - Integrate with existing `/personalize` endpoint

3. **Validation:**
   - Test all canonical archetypes return valid data
   - Verify <300ms total personalization time
   - Confirm error handling works correctly

### **Phase 2 Migration**
1. **Data Migration:**
   - Populate enhanced archetype registry
   - Create selection tracking tables
   - Import additional archetype candidates

2. **Feature Rollout:**
   - Enable smart selection for 10% of traffic
   - A/B test canonical vs. enhanced selection
   - Gradually increase traffic to enhanced logic

3. **Monitoring:**
   - Track selection effectiveness metrics
   - Monitor performance impact
   - Validate fallback behavior

---

## ✅ **Success Criteria & Acceptance Tests**

### **Phase 1 MVP Success Criteria**
- [ ] All canonical archetypes return complete performance stories
- [ ] Archetype retrieval completes in <100ms (P90)
- [ ] Total personalization flow completes in <300ms (P90)
- [ ] Error rate <1% under normal load
- [ ] Graceful fallback behavior for missing data
- [ ] Frontend integration successful with real archetype data

### **Phase 2 Enhancement Success Criteria**  
- [ ] Smart selection provides measurably more relevant archetypes
- [ ] Geographic matching accuracy >85% based on user feedback
- [ ] Rotation logic prevents repetition for returning visitors
- [ ] Enhanced selection maintains performance targets
- [ ] A/B tests show improved conversion vs. canonical approach

---

## 📝 **Decision Log & Rollback Plan**

### **Key Architectural Decisions**
1. **Decision:** Use extensible interface design for Phase 1/2 compatibility
   - **Rationale:** Enables enhancement without refactoring
   - **Risk Mitigation:** Interface abstraction isolates implementation changes

2. **Decision:** Start with canonical archetypes rather than dynamic selection
   - **Rationale:** Meet MVP deadline with reliable, simple implementation
   - **Risk Mitigation:** Proven data quality and performance characteristics

3. **Decision:** Build hierarchical fallback system
   - **Rationale:** Ensure 100% uptime even with service failures
   - **Risk Mitigation:** Multiple fallback layers prevent user-facing errors

### **Rollback Plan**
- **Phase 2 Rollback:** Toggle feature flag to disable smart selection, fall back to canonical
- **Phase 1 Rollback:** Disable archetype service, fall back to synthetic data in LLM
- **Complete Rollback:** Serve static templates for each ICP

---

## 🔄 **Maintenance & Evolution Plan**

### **Ongoing Maintenance**
- **Monthly:** Review archetype data quality and update canonical selections
- **Quarterly:** Analyze selection effectiveness and optimize criteria
- **Bi-annually:** Expand archetype registry with new student data

### **Future Enhancements**
- **Phase 3:** Subject-specific archetype matching (math/reading focus)
- **Phase 4:** Multi-archetype narratives (showing multiple student stories)
- **Phase 5:** Real-time archetype performance optimization

---

*This implementation plan ensures MVP delivery while establishing foundation for sophisticated personalization capabilities that differentiate TimeBack in the EdTech market.* 