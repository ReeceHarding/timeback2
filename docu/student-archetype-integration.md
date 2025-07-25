# Student Archetype Data Contract

**Version:** 1.0  
**Date:** July 24, 2025  
**Purpose:** Defines how real Alpha School student data flows through the personalization pipeline to create authentic, compelling case studies for prospective parents.

---

## 🎯 **Core Strategy: Real Data for Authentic Narratives**

Our competitive advantage is using **real student outcome data** from Alpha School to create compelling case studies for prospective parents. This document defines the data contract for how we:

1. **Select** a representative student archetype from our database
2. **Extract** their performance story (growth, context, outcomes)  
3. **Transform** it into compelling narrative blocks via LLM
4. **Present** it as relevant proof to the prospective parent

---

## 📊 **Available Data Assets**

### **Data Sources** (from `rawdata/` → `entity_version` table)
| Entity Type | Source File | Key Data | Purpose |
|-------------|-------------|----------|---------|
| `student` | `StudentsBySchool.csv` | Demographics, school context | Archetype selection, context |
| `map_snapshot` | `Spring 2024-2025 MAP Snapshot` | **RIT scores by season/subject** | **Before/after growth proof** |
| `assessment` | `AssessmentResults.csv` | Detailed performance metrics | Skill-specific improvements |
| `class_assignment` | `ClassAssignments.csv` | Coursework progress | Learning trajectory |
| `program_assignment` | `ProgramAssignments.csv` | Program enrollment | Context/credibility |

### **Key Performance Indicators Available**
- **RIT Score Growth**: Fall → Winter → Spring progression by subject
- **Percentile Improvements**: National ranking jumps (e.g., 37th → 92nd percentile)
- **Goal-Specific Progress**: Skill area breakdowns (e.g., "Operations and Algebraic Thinking")
- **College Readiness**: ACT/SAT track indicators
- **Real School Context**: Alpha Miami, Virtual 2 Hour Learning program names

---

## 🔄 **Data Flow Architecture**

### **Step 1: Archetype Selection** (`dashboardService.getArchetypeStudent()`)
**Input:** Parent's quiz response (grade level)  
**Process:** Query `student_current_v` for first student matching grade  
**Output:** Basic student demographics

```typescript
interface StudentArchetype {
  student_id: string;           // "000-0783"
  first_name: string | null;    // "Anonymous" (privacy)
  last_name: string | null;     // "Student" (privacy)  
  grade: string | null;         // "Grade 9" 
  school_name: string | null;   // "Alpha Miami"
  district_name: string | null; // "Virtual 2 Hour Learning"
  uploaded_at: Date;            // Data freshness
}
```

### **Step 2: Performance Data Enrichment** (New Function Needed)
**Input:** `student_id` from Step 1  
**Process:** Query related `entity_version` records for performance data  
**Output:** Enriched archetype with growth story

```typescript
interface EnrichedStudentArchetype extends StudentArchetype {
  performance_story: {
    map_growth: {
      subject: string;           // "Mathematics"
      fall_rit: number | null;   // 215 (starting point)
      spring_rit: number;        // 253 (ending point) 
      percentile_jump: string;   // "37th → 92nd percentile"
      growth_narrative: string;  // "Exceptional 38-point RIT improvement"
    }[];
    
    academic_context: {
      program_name: string;      // "Virtual 2 Hour Learning"
      school_location: string;   // "Alpha Miami"
      grade_context: string;     // "Grade 9" 
    };
    
    achievement_highlights: {
      college_readiness: string; // "On Track for ACT 24"
      skill_improvements: string[]; // ["Operations and Algebraic Thinking: High"]
    };
  };
}
```

### **Step 3: LLM Narrative Generation** (Enhanced `getPersonalizedContent()`)
**Input:** Parent context + `EnrichedStudentArchetype`  
**Process:** AI weaves real data into persuasive narrative blocks  
**Output:** Personalized component blocks with authentic proof

---

## 📝 **Implementation Plan**

### **Phase 1: Data Enrichment Service** (New)
Create `backend/src/services/performance.service.ts`:

```typescript
class PerformanceService {
  async getStudentPerformanceStory(studentId: string): Promise<PerformanceStory> {
    // Query MAP growth data
    // Query assessment results  
    // Calculate growth narratives
    // Return structured performance story
  }
  
  async getEnrichedArchetype(grade: string): Promise<EnrichedStudentArchetype> {
    // Get basic student from dashboard service
    // Enrich with performance story
    // Return complete archetype
  }
}
```

### **Phase 2: Enhanced Dashboard Service** (Modify Existing)
Update `dashboardService.getArchetypeStudent()`:
- Change return type to `EnrichedStudentArchetype`
- Integrate with `PerformanceService`
- Add error handling for missing performance data

### **Phase 3: LLM Prompt Enhancement** (Modify Existing)  
Update `backend/src/llmClient.ts`:
- Add `archetypeStudent?: EnrichedStudentArchetype` parameter
- Modify prompts to use real performance data
- Replace synthetic examples with authentic case studies

### **Phase 4: Integration** (Modify Existing)
Update `/personalize` endpoint in `backend/src/index.ts`:
- Call enhanced dashboard service
- Pass enriched data to LLM client
- Handle fallback scenarios

---

## 🛡️ **Privacy & Anonymization**

### **Student Privacy Protection**
- **Names**: Always anonymized ("Anonymous Student", never real names)
- **IDs**: Internal IDs only, never exposed to frontend  
- **Context**: School/program names preserved (public institutions)
- **Performance**: Aggregate patterns, not individual identifiers

### **Data Usage Consent**
- Alpha School student data used for **aggregated case studies only**
- No individual student identifiable in final narratives
- Performance patterns represent **program effectiveness**, not individual students

---

## 📈 **Expected Narrative Improvements**

### **Before** (Current Synthetic Approach)
```markdown
## Expected Performance Jump in Grade 9
*Generic chart showing hypothetical improvement*
```

### **After** (Real Data Approach)  
```markdown
## Real Results: Alpha Miami Grade 9 Student
**Mathematics: 37th → 92nd percentile in one school year**

This Grade 9 student started with a 215 RIT score in Fall 2024 and achieved 253 RIT by Spring 2025—a 38-point improvement that exceeded typical growth by 2.3x. Now "On Track for ACT 24" with particular strength in Operations and Algebraic Thinking.

*Source: NWEA MAP Growth assessment, Alpha Miami, Virtual 2 Hour Learning program*
```

---

## 🔄 **Fallback Strategy**

### **When Real Data Unavailable**
- **Missing Grade**: Use closest available grade archetype
- **Missing Performance**: Fall back to current synthetic approach  
- **Service Error**: Graceful degradation to static templates
- **Privacy Concerns**: Always prioritize anonymization over specificity

### **Error Handling**
- Log data availability for each grade level
- Monitor archetype selection success rates
- Alert when key data sources become stale

---

## 🎯 **Success Metrics**

### **Technical Metrics**
- **Data Coverage**: % of grades with available archetypes
- **Performance Enrichment**: % of archetypes with complete growth stories
- **Response Time**: Enriched data retrieval under 100ms

### **Business Metrics**  
- **Narrative Authenticity**: Real vs synthetic content ratio
- **Conversion Impact**: A/B test real vs synthetic case studies
- **Credibility**: Parent feedback on "believability" of proof points

---

## 🚀 **Future Enhancements**

### **Phase 2 Roadmap**
- **Multiple Archetypes**: Show 2-3 students per grade for variety
- **Subject-Specific Matching**: Match student by parent's subject concerns
- **Geographic Alignment**: Prefer students from similar demographic contexts
- **Longitudinal Stories**: Multi-year growth trajectories for advanced narratives

---

*This contract enables our core competitive advantage: **real proof points** that no competitor can match.* 