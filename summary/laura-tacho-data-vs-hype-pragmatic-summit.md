---
url: https://gist.github.com/a394507d48e3135a19de8cb3a50a369f
title: "Data vs Hype: How Orgs Actually Win with AI"
author: Laura Tacho
date_fetched: 2026-07-04
date_published: 2026-02-24
source: The Pragmatic Summit (YouTube: The Pragmatic Engineer)
original_url: https://www.youtube.com/watch?v=LOHgRw43fFk
ytx_gist: https://gist.github.com/njt/a394507d48e3135a19de8cb3a50a369f
---

# Data vs Hype: How Orgs Actually Win with AI

Laura Tacho (CTO, DX) at The Pragmatic Summit, February 24, 2026. 29m 49s.

## 1. Key Points

- **Adoption is near-universal, but organizational transformation is rare.** 92.6% of developers use an AI coding assistant at least monthly, yet an MIT study of 152 orgs found "high adoption, low transformation." Using the tool does not equal impact.
- **AI is an accelerator, not a fixer.** It amplifies existing organizational health: functional teams get faster with higher quality; dysfunctional teams become "dysfunctional faster" and can see twice as many customer-facing incidents.
- **Time savings have plateaued around 10%.** Self-reported hours saved hover at roughly 4 hours per developer per week, a figure that has barely moved in recent quarters. The ceiling on individual coding-task productivity is low.
- **AI-authored code in production is rising fast.** Industry-wide, 26.9% of merged code was written by AI without significant human intervention, up from 22% the prior quarter. Daily AI users exceed 30%.
- **Onboarding time has been cut in half.** Time-to-10th-PR has dropped by 50% since Q1 2024, and Microsoft research shows that faster onboarding performance sticks with an engineer for their first two years.
- **Agentic workflows are the expanding frontier.** Over 50% of developers at instrumented companies use agentic workflows daily. The ceiling is much higher, but the organizational challenges are the same.
- **Winning orgs do three things:** they set concrete goals and measure progress against them; they treat developer experience (feedback loops, docs, CI) as critical infrastructure for AI; and they focus experimentation on real customer problems, not moonshots.

## 2. Pithy and Provocative Quotes

- "Adoption doesn't mean impact."
- "Organizations who were dysfunctional already — now they're more dysfunctional, they're dysfunctional faster."
- "Spray and pray does not work."
- "Just call it agent experience and you'll get money for it."
- "The risk is if we don't address the systems level problems, we will just take them to space with us."
- "Stay grounded, stay skeptical, stay human. Most of all, stay pragmatic."
- "Average does not mean typical."
- "AI does not solve organizational systems problems."
- "Transformation is really uncomfortable."

## 3. Tools, Practices, and Methodologies

- **AI Measurement Framework (co-authored with Abi Noda / DX):** Tracks AI usage/adoption and translates it into organizational impact (speed, DevEx, quality, innovation ratio, cost). Connects adoption to real outcomes instead of just counting licenses.
- **DORA AI Capabilities Model (dora.dev):** AI readiness model backed by DORA's research data. Identifies practices correlated with good AI outcomes — e.g., having a clear, communicated AI stance.
- **ThoughtWorks Forest Framework:** Another industry-backed AI readiness model for internal audits on whether the org is doing the right things to benefit from AI experimentation.
- **Time-to-10th-PR as an onboarding metric:** Industry-aligned milestone for developer onboarding. Halved with AI usage; Microsoft research links faster time-to-10th-PR to sustained productivity gains for two years.
- **Multi-agent consensus patterns (JPMorgan Chase's MAFA framework):** Specialized agents annotate interactions, a second set re-ranks and validates output, consensus algorithms resolve disagreements. "Consensus among agents will be a huge problem to solve in 2026."
- **RALPH loops for rapid prototyping (Haven Headache Center):** Agentic workflows that take Linear and Figma artifacts, convert them into a PRD and JSON, then run RALPH loops to generate high-quality prototypes with documentation and tests.
- **HIPAA-compliant model trained on symptom logs (Haven):** Training a compliant model on hundreds of thousands of patient symptom messages to route them to medication refills or follow-ups. 3× industry customer satisfaction, real clinical improvements.

## 4. Case Studies

- **Haven Headache & Migraine Center:** Small clinic using RALPH loops and HIPAA-compliant models. 3× industry customer satisfaction, 3× improvement in clinical outcomes over expected.
- **Cisco:** 18,000 engineers using Codex daily. 50% faster code review. Concrete productivity gains at enterprise scale.
- **JPMorgan Chase:** MAFA multi-agent framework with consensus algorithms. Using agentic workflows in a regulated environment.

## 5. Unanswered Questions and Omissions

- **What does the "low transformation" org actually do to change?** The talk identifies the problem — adoption without impact — and says orgs need change management, but doesn't outline what that change management looks like in practice. What specific steps move a company from the 92.6% adoption bucket into the "winning" bucket?
- **The cost question is raised and immediately dropped.** The AI Measurement Framework includes a cost dimension, and the speaker notes costs keep going up, but there's no data or guidance on how to evaluate ROI when model and inference costs are rising.
- **The "agent experience" funding hack is cynical but unexplored.** The speaker suggests rebranding DevEx initiatives as "agent experience" to unlock budget, but doesn't address the ethical or long-term consequences of funding human infrastructure only when it serves AI.
- **What happens to the developers who aren't daily AI users?** The data shows a gap between 92.6% monthly adoption and 75% weekly adoption. The talk doesn't explore the experience or productivity of non-power users — are they being left behind?
- **No discussion of non-engineering functions.** The entire dataset and argument focus on developers and code. How do these patterns apply to AI adoption in product, design, marketing, or operations?
- **The environmental impact is name-checked and ignored.** The speaker mentions "skepticism about the real economic impact of AI, given how expensive it is, given the environmental impact," but never returns to the environmental dimension.
- **What's the counterargument to "just experiment on customer problems"?** The talk says moonshot experimentation is fine but unsustainable, and real wins come from solving customer problems. But many breakthrough innovations came from exploration not directly tied to immediate customer problems. How do orgs balance the portfolio?

## Transcript

Full transcript available in the ytx gist: transcript.md (29m 49s talk, transcribed 2026-05-18). Opens with a Carl Sagan space race analogy: "somewhere something incredible is waiting to be known." Data from 121,000 developers across 450+ companies.
