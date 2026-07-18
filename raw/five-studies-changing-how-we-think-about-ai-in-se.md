---
url: https://newsletter.getdx.com/p/five-studies-that-are-changing-how
title: "Five studies changing how I think about AI in software engineering"
author: Brian Houck
date_fetched: 2026-07-18
date_published: 2026-07-10
---

# Five studies changing how I think about AI in software engineering

**Subtitle:** AI compressed the upstream work. What does that mean for everything downstream?

**Author:** Brian Houck

**Publication:** Engineering Enablement (Substack newsletter)

**Date:** July 10, 2026

---

## Context & Framing

Houck opens by describing a moment when multiple independent papers converge to tell a larger story. He states that five recent papers have "significantly influenced how I'm thinking about AI and software engineering." Despite different methodologies and research groups, he sees them converging on a shared narrative.

**Core thesis:** "AI is compressing the upstream work of software engineering." The question shifts from "Is AI making developers faster?" to what happens after code generation — whether value is actually being shipped, where new bottlenecks arise, and what costs emerge when understanding can't keep pace with generation.

**Overarching conclusion (paraphrased):** Code generation is outpacing the systems needed to safely understand, verify, and deliver that code.

**Disclosure:** Houck notes that three of the five papers come from people he knows and works with extensively, and that none are his own.

---

## Study 1: GitHub Copilot and Developer Productivity

- **Authors:** Heilman, A., Kyllo, A., Murphy-Hill, E.
- **Paper:** "GitHub Copilot and Developer Productivity: An Observational Dose-Response Analysis" (arXiv)
- **Link:** arxiv.org/abs/2606.00438

**Methodology:** Instead of comparing Copilot users to non-users, the authors controlled for Active Coding Time and examined productivity variation within the same engineer over 43 weeks across 16,223 developers. This self-comparison design is highlighted as particularly clever.

**Key finding:** The highest Copilot usage weeks were associated with roughly 40% more completed PRs per coding hour compared to zero-usage weeks. The relationship showed a "dose-response pattern" — more engagement meant more throughput, though gains leveled off at very high usage.

**Robustness:** Seven robustness and falsification tests ruled out alternative explanations (team effects, generic AI engagement, PR slicing, shifts toward easier work). The positive association held consistently.

**Notable detail:** The strongest effects appeared for larger PRs (7+ files), countering the theory that developers merely break work into smaller units. Houck summarizes that the findings show "we're not just coding more, we're increasing coding efficiency as well."

---

## Study 2: Writing Code vs. Shipping Code

- **Authors:** Demirer, M., Musolff, L., Yang, L.
- **Paper:** "Writing Code vs. Shipping Code: Productivity Effects Across Generations of AI Coding Tools" (National Bureau of Economic Research, Working Paper 35275)
- **Link:** www.nber.org/papers/w35275

**Focus:** Whether individual coding speed gains survive all the way to shipped software. The authors analyzed AI adoption across 100,000+ GitHub developers.

**Hierarchy examined:** lines of code → files → commits → pull requests → projects/repos → releases

**Findings on coding activity increases by tool generation:**
- Autocomplete tools: roughly +40% more commits
- Interactive coding agents: roughly +140% more commits
- Autonomous agents: roughly +180% more commits

**Attenuation through delivery:** These gains fall off dramatically as work progresses. The largest effects appear in code generation. Smaller effects appear in repos touched, smaller still in releases shipped, and ultimately software consumed by users. Even with large increases in coding activity, the effect on shipped software tops out at roughly +30% more releases.

**Elasticity of substitution:** They estimated a low elasticity (~0.25) between AI-generated output and human effort. Houck (an economics degree holder) notes this measures how replaceable human work is with AI output. The estimate suggests "AI and human work are still largely complements rather than substitutes," with substantial human effort still required for review, integration, validation, and shipping.

**Open question:** Whether this fall-off is a fundamental limit of software engineering or simply reflects that organizations haven't adapted processes to an agentic world. Houck frames this paper as asking a harder question than Study 1: "faster at what, exactly?"

---

## Study 3: AI Coding Assistants — A Longitudinal View

- **Authors:** Vella, A., Blincoe, K.
- **Paper:** "The Impact of AI Coding Assistants on Software Engineering: A Longitudinal Study" (arXiv)
- **Link:** arxiv.org/abs/2605.23135

**Methodology:** Six-month longitudinal study of 95 professional software engineers, using two questionnaires with mixed-methods analysis and reflexive thematic analysis on open-ended responses.

**Productivity perceptions:** Stable and strongly positive over time. At both time points, 84% of participants reported improvement — consistent with Studies 1 and 2.

**Striking finding — the "productivity-experience paradox":** Among the matched cohort, the share of engineers reporting worse DevEx on at least one dimension nearly doubled in six months (from 14% to 27%). Flow state was the most vulnerable dimension; cognitive load eroded modestly; feedback loops actually improved.

**Key insight on decoupling:** Cross-sectional correlations between DevEx and productivity were strong, but change scores didn't correlate. Houck summarizes that "productivity and developer experience appear to be decoupling over time in AI-assisted workflows." He notes this has significant implications for those working with SPACE and DevEx frameworks.

**Caveat acknowledged:** The study population was not particularly large, but findings were significant and rigorously validated. Houck praises the rare longitudinal design and calls the productivity-experience decoupling a finding "worth replicating in larger populations."

---

## Study 4: 22 AI Systems Developers Want Built

- **Authors:** Choudhuri, R., Badea, C., Bird, C., Butler, J., DeLine, R., Houck, B.
- **Paper:** "AI Where It Matters: Where, Why, and How Developers Want AI Support in Daily Work" (arXiv)
- **Link:** arxiv.org/abs/2510.00762
- **Interactive site:** aka.ms/ai-where-it-matters

**Background:** Houck co-authored the original paper. His co-authors produced a second paper based on survey responses from 860 Microsoft developers across roles, domains, and geographies.

**Core concept — "bounded delegation":** The paper outlines a roadmap of 22 AI tools developers want beyond code generation.

**Three major themes:**

1. **The "right-shift" burden:** AI speeding up code generation creates a massive downstream bottleneck. Developers face more code to review, more production incidents to debug, and documentation that falls behind faster.

2. **Shift to verification:** Developers want AI embedded in verification tasks — tools that auto-assemble log/trace "case files" for on-call incidents, PR reviewers catching complex business logic flaws before human review, and change-aware test generation that identifies which assertions matter.

3. **Bounded delegation boundaries:** Developers want AI to absorb tedious "assembly work" (updating docs, writing edge-case unit tests) but never core logic, architecture, or critical decision-making. Houck notes this line was drawn even for tasks developers acknowledged AI could plausibly handle — "suggesting it's not just about capability gaps."

**Four non-negotiable guardrails for AI tools:**
- Explicit authority scoping (no auto-approvals)
- Clear data provenance
- Explicit uncertainty signaling (AI must admit when it doesn't know something)
- Least-privilege security access

---

## Study 5: From Technical Debt to Cognitive and Intent Debt

- **Author:** Storey, M.
- **Paper:** "From Technical Debt to Cognitive and Intent Debt: Rethinking software health in the age of AI" (ACM Queue)
- **Link:** queue.acm.org/detail.cfm?id=3807966

**Central argument (Houck calls this the most important paper he's read in a long time):** The technical debt metaphor is no longer sufficient. AI is reducing technical debt (through refactoring, test generation, automated review) while quietly accelerating two other forms of debt.

**Three types of debt defined:**

- **Technical debt** — lives in code; accumulates when implementation decisions compromise future changeability. AI is genuinely helping here.
- **Cognitive debt** — lives in people; accumulates when a team's shared understanding of a system erodes faster than it's replenished. When AI generates code, developers may accept it without building the mental model they'd have built writing it themselves. Multiplied across a team over time, this becomes an "accumulation of not knowing."
- **Intent debt** — lives in artifacts; accumulates when goals, constraints, and rationale guiding a system are unclear, unwritten, or forgotten. As more development is AI-assisted, intent debt becomes a first-order constraint on what AI can do.

**How they compound:** The three debts interact — intent debt causes cognitive debt; cognitive debt causes technical debt; technical debt amplifies cognitive debt. Managing software health requires attention to all three layers.

**Practical implication:** "Treat understanding as a deliverable." Shared understanding should be a first-class product of development, not a side effect of writing code.

---

## Synthesis: Houck's Final Thoughts

Houck weaves the five papers together into a cohesive narrative:

- **Heilman:** Per-engineer efficiency gains are real
- **Demirer:** But those gains don't survive to shipped software at anywhere near the same magnitude
- **Choudhuri:** The bottleneck has moved downstream; developers explicitly want tools for review, integration, verification, and understanding while refusing to delegate what they consider craft
- **Vella:** The lived experience is more uneven than productivity numbers suggest — flow and cognitive load erode even as throughput holds
- **Storey:** The deepest cost may be unmeasured — the slow erosion of shared understanding that makes any system safe to change

**Closing statement (paraphrased):** The bottleneck has shifted, but our tools, metrics, and team designs haven't moved with it yet — that's where the next several years of work will happen.

---

## Comment Section Highlight

A commenter named Engincan Veske notes that "AI moves the bottleneck from typing code to reviewing it" and asks whether any of the five studies separated senior from junior reviewers, suggesting that gap "seems to widen the most." Brian Houck "liked" the comment.
