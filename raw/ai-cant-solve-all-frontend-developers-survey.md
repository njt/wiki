---
url: https://www.telerik.com/blogs/ai-cant-solve-all-what-120-frontend-developers-say-they-still-hate-working
title: "AI Can't Solve It All: Here's What 120+ Frontend Developers Say They Still Hate Working On"
author: Kathryn Grayson Nanz
date_fetched: 2026-07-18
date_published: 2026-07-06
site: Telerik Blog (Progress)
tags: [AI, Developer Community, JavaScript, React, Web Development, Survey, Frontend]
---

# AI Can't Solve It All: Here's What 120+ Frontend Developers Say They Still Hate Working On

**Author:** Kathryn Grayson Nanz — Developer Advocate at Progress, with a background in graphic design and a focus on React, UI, and design.

**Published:** July 6, 2026 (4 min read)

---

## Methodology

Informal user research at two back-to-back conferences — JSNation and React Summit — using a whiteboard and sticky notes. Approximately 120+ developers participated over two days under two distinct prompts:

- **Day 1 (JSNation):** "What's the most annoying frontend task or component to build from scratch?"
- **Day 2 (React Summit):** "What does AI still get wrong when building UI in React?"

The response format was open-ended (not a formal survey), which the author noted yielded "surprisingly consistent" answers despite the lack of structure.

---

## Day 1: Most Annoying Components/Tasks

The split between named components and tasks was roughly even. Top categories by mentions:

| Theme | Mentions |
|---|---|
| Date pickers | 9 |
| AI-related work (prompt engineering, AI-generated designs, AI features, chatbots) | 8 |
| Accessibility (WCAG compliance, testing, specific components like calendars and comboboxes) | 7 |
| Requirements & development process (scope creep, non-technical coworkers, client management) | 6 |
| Data grids & tables | 5 |
| Rich text editors | 4 |
| Design systems & theming | 4 |
| Comboboxes | 4 |

As the author put it: "it sucked to build a date picker 10 years ago and it still sucks today." Specific complaints around date pickers included timezone handling, accessibility, and date ranges.

---

## Day 2: AI's Shortcomings in React

When the question turned specifically to AI-generated React code:

| Theme | Mentions |
|---|---|
| React architecture | 6 |
| CSS & styling | 4 |
| Effects & async logic | 2 |
| Animation | 2 |

### Specific AI-Generated Code Complaints

Developers reported that AI tools frequently produce code that:

- Violates the rules of hooks
- Creates unnecessary `useState` calls
- Places state outside components (incorrectly)
- Generates new components instead of reusing existing ones
- Produces code without understanding project requirements
- Includes poor quality CSS

---

## Recurring Pain Points Across Both Days

Several themes appeared on both day's whiteboards, indicating persistent struggles:

- Accessibility — mentioned both as a standalone complaint and as something AI has not meaningfully alleviated
- Design systems — coordination and implementation remain difficult
- Figma-to-code workflows — translating designs to code is still friction-filled
- Localization — building for global audiences continues to challenge teams
- Testing
- State management
- Performance
- CSS — specifically noted as poorly handled by AI

The author observed that these are "not new problems" and many stem from "very human aspects of software development" — cross-team coordination, inclusivity, global audience considerations, and user testing.

---

## Main Arguments and Conclusions

1. **Integration, not tools, is the hard part.** "The difficulty now (as it always has been) is in the integration of tools into cohesive systems." Tooling has evolved, but the fundamental challenge of assembling everything into a working whole has not.

2. **AI shifts rather than eliminates work.** "AI is clearly changing how developers build software, but it hasn't eliminated the need for thoughtful engineering." AI has shifted developer effort toward "reviewing, refining and improving generated code rather than writing every line themselves."

3. **Many pain points have existing solutions.** "Component-centric frustrations" — date pickers, data grids, accessibility, design systems — "all have something in common: they're problems with existing solutions." The author recommends modern component libraries as a remedy and specifically cites Progress Telerik and Kendo UI libraries.

4. **Uncertainty about net improvement.** "Whether that's an improvement or not is something only time will tell."

---

## Call to Action / Recommendation

"Investing in the right UI foundation can remove much of the friction developers told us they're still experiencing today." The post is framed as user research but functionally serves as thought leadership promoting Telerik/Kendo UI's commercial component libraries as a solution for the exact pain points the survey surfaced.

---

*Fetched: 2026-07-18*
