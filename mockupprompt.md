Prompt for Lovable / v0
```
Project name: TimeBack “Living Story” Site

Goal:
Design a 3-screen responsive web experience that feels hyper-personalised for parents exploring TimeBack, an AI-powered education platform that lets kids “crush academics in 2 hours a day.”

Brand guard-rails
• Tailwind palette: primary #1E40AF (indigo-900), accent #F59E0B (amber-500), gray-surface #F9FAFB  
• Font scale: Inter / system-UI, 14 – 48 px  
• Motion: 200 ms ease-out fade / slide

Screen 1 – Landing “Hero”
• Full-width Hero component
    – Headline: “Your child can crush academics in just 2 hours per day.”  
    – Sub-headline: “See how Brownsville students jumped from 31st→84th percentile in 12 months.”  
    – Primary CTA button “Get Early Access”  
• Above-the-fold stat strip (3 StatBlock cards)  
    1. “2× Learning Velocity”  
    2. “1410 Avg SAT at Alpha Schools”  
    3. “26 000 Alpha Anywhere sign-ups”  
• Email entry modal pops after CTA click (just design flow state)

Screen 2 – Proof Section (scroll target)
• TwoColLayout  
    Left: 20 s video thumbnail (StoryVideo) – caption “Brownsville Parent Story”  
    Right: large StatBlock with “31st → 84th percentile” + short paragraph  
• Below: interactive Chart (placeholder) titled “Your school vs. TimeBack” with Compare button

Screen 3 – Dynamic Quiz / Personalisation
• Centered Diagram section that illustrates “Questionnaire → AI → Personalised Lesson” (simple 3-node flowchart)  
• Mini-quiz card (radio buttons) – “Which best describes you?” 5 options (Parent, Educator, Entrepreneur, Government, Philanthropist)  
• Secondary CTA “Show my personalised plan”  
• Footer with FAQ accordion (3 items) and slim e-mail capture bar

General component notes
• Use only these atoms: Hero, StatBlock, StoryVideo, Chart, Diagram, CTAButton, TwoColLayout, FAQ, EmailModal.  
• No chatbots on the UI.  
• Design should feel warm & story-driven, not overly “tech.”  
• Mobile first: cards stack, video becomes 16 × 9, quiz radio buttons full-width.

Deliverables
• 3 responsive frames (1440 px desktop & 390 px mobile)  
• Layer naming: ComponentName_variant (e.g. StatBlock_primary)  
• Export as Figma frames 