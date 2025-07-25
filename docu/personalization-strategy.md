# Personalization Strategy & Roadmap

This document outlines the multi-phase strategy for enhancing the personalization of the TimeBack user dashboard. The goal is to move beyond our MVP personalization and create an experience that feels uniquely insightful and valuable to each user.

Our strategy is divided into three distinct paths, each building on the last.

---

### **Core Personalization Logic: The Case Study Archetype**

**This is the foundational concept that powers our MVP and all subsequent personalization efforts.**

It is critical to distinguish between our two primary assets:

1.  **Alpha School (Internal Data):** This is our "lab" where we apply our learning methods to real students. The data from `rawdata/` is the ground truth from Alpha School. It serves as our internal, evidence-based proof that the TimeBack method works.

2.  **Timeback.com (Public Website):** This is our marketing platform for prospective customers (e.g., parents of children in traditional schools). These users are **not** in our database.

**The Strategy:**

We use the real, granular data of a **single, anonymized Alpha School student** as a **powerful and representative case study** to persuade the prospective parent visiting our website.

The user flow is as follows:

1.  **Context Gathering:** A new parent arrives and completes the 2-question quiz, providing their context (e.g., "I'm a parent concerned about my 9th-grade child's grades").
2.  **Archetype Selection:** The backend receives this context and queries our internal Alpha School data to find a **representative student archetype**. For example, it will look for an anonymized 9th-grade student from our database who has shown significant grade improvement.
3.  **Data-Driven Narrative:** The backend then takes the **real performance data** from this "case study student" and uses our LLM (Groq) to weave it into the persuasive narrative defined in `dashboard-narrative-spec.md`.
4.  **Personalized Result:** The visiting parent sees a dashboard that feels uniquely relevant to their situation because it's backed by a real, data-driven story. Instead of a vague marketing claim, they see a concrete example: "Here is the actual performance jump we achieved with a 9th grader who was in a similar situation to your child."

This "single student as archetype" model is how we bridge the gap between our internal proof points and our external marketing, making our claims tangible and trustworthy.

---

### **Path 1: Grade-Level Deep Dive (Immediate Priority)**

This path focuses on making our existing personalization much more nuanced by tailoring content to the user's specific educational stage.

*   **Concept:** The advice and tone for a parent of a 5th grader should be fundamentally different from that for a parent of a 10th grader. We will create three distinct content tracks based on the child's grade level.
    *   **Elementary School (K-5):** Focus on foundational skills, confidence, and fostering a love of learning.
    *   **Middle School (6-8):** Focus on navigating academic complexity, developing study habits, and high school preparation.
    *   **High School (9-12):** Focus on college readiness, AP/IB courses, standardized tests, and building a standout academic profile.
*   **Implementation:** This is primarily a backend AI-prompting task. We will create "sub-prompts" within `llmClient.ts` that are dynamically chosen based on the `grade` provided in the quiz. No new UI is required.
*   **Effort:** **Medium.**
*   **Status:** **Completed.**

---

### **Path 2: The Interactive Dashboard**

This path transforms the dashboard from a static report into a dynamic, two-way conversation that progressively personalizes itself.

*   **Concept:** After the initial dashboard is delivered, we will present an optional, high-impact question within a new, interactive block. For example: *"What subject is the biggest challenge right now?"* with buttons for [Math], [Reading], etc. Clicking an option would instantly refire the personalization engine and replace a generic block with a hyper-specific one (e.g., a sample math tutoring plan).
*   **Implementation:** This is a full-stack effort. It requires a new interactive block type on the frontend, updated state management, and a new backend endpoint (e.g., `/personalize/refine`) with more complex AI logic.
*   **Effort:** **High.**
*   **Status:** **Completed.** The frontend and backend are fully implemented. The dashboard now features an interactive prompt that allows users to refine their results and dynamically see new, more specific content.

---

### **Path 3: Hyper-Local Proof (Advanced RAG)**

This path doubles down on our geographic personalization to make our proof points feel irrefutable and highly relevant.

*   **Concept:** We will go beyond simply using the state name in a headline and weave in actual, localized data and resources.
    *   **Phase 1 (Resource Matching):** We will categorize our state-specific resources so we can provide more relevant links. For instance, a parent concerned about their child "catching up" would receive a link to a state-funded tutoring program, not just the generic Department of Education website.
    *   **Phase 2 (Data Integration):** We will integrate publicly available education data (e.g., from NCES) to provide hyper-local benchmarks. The dashboard could show a chart comparing the user's potential progress against their actual school district's average (e.g., *"In the Los Angeles Unified School District, the average math proficiency is 38%. Our students consistently outperform this benchmark..."*).
*   **Implementation:** This is primarily a data-sourcing and backend project. Phase 1 is a medium-effort task involving data categorization. Phase 2 is a much larger undertaking requiring a data pipeline to ingest and serve public educational data.
*   **Effort:** **Medium (Phase 1) to Very High (Phase 2).** 