# Time Back Site Vision & Technical Strategy

## Joe's Core Vision

**"Incept tailors questions to students, why can't the site tailor content to parents?"**

### What Joe Mentioned During Sunday Lunch

Site should use **fast LLMs to generate site content hyper tailored to the site visitor**

**Factors to integrate:**
- User's location using IP address
- Which of the 5 ICP's they opt into
- User experience data

**Process:**
1. Generate a custom homepage using the user's location and ICP provided
2. Ask them another follow-up question so we can generate the next section of the site
3. We said site loading can take up to 30 seconds
4. **If we stream in tokens, I think we can start showing content in less than 3 seconds**

---

## Critical Questions

### Is it worth building out the technical complexity of something that's never been built before?

**Alternative approach:** Simply build out a bunch of different pages for different states and interests and then show the user the content from that page

### Pros of LLM Site:
- Might go viral
- Joe will be happy
- Good first impression
- Productizable in future

### Cons of LLM Site:
- **How will SEO work?**
- Load times might be long
- Lots of complexity since it's never been built before

---

## User Journey Design

### What does the user journey look like?

Joe said he wants **max one question presented to the user before loading in the site**

**What is that one question?**
*"Which of these 5 ICP's are you?"*

### What does Joe truly want at the core?

Seems to me that he wants **a site that makes TimeBack's technical sophistication look impressive**

### What's the simplest way we can implement this?

### What interactive elements on the site might accomplish the same goals w/ less complexity?

**Example interactive element:**
Place where people can put in their student's information and we pull in test scores of Alpha kids similar to their student and say:

*"Jamie joined us at 54th percentile and 2 years later was at 98th percentile" - here's her story*

**Note:** Joe explicitly doesn't want the UI to have chatbots

---

## Strategic Considerations

### What is the ideal user experience irrespective of technical sophistication?

**Core question:** What do users want and how can we help them in the best way possible?

- This could be LLM or it could be something else
- I really don't think Joe has a definitive view of this site, so **we have an opportunity to advise him on what we recommend**

---

## Implementation Strategy

### If we build this out, what's our strategy?

**Recommended approach:**

1. **Build a number of templates first** to get one that Joe likes
2. **Then, integrate in the LLM generation system** (if we decide this is the best plan)
3. **Show him the UX** and see if it's something he likes

**For this, use a super fast LLM provider like Groq**

---

## Technical Specifications

- **Target load time:** Start showing content in less than 3 seconds via token streaming
- **LLM Provider:** Groq (for speed)
- **IP-based location detection**
- **5 ICP targeting system**
- **Template-based foundation with LLM enhancement** 