# The Problem is not the AI Code, but Nobody Knows Anything Anymore

Simon Späti's brain-note argues that the damage done by agentic coding is not the code itself — AI writes "probably average code" — but the disappearance of human understanding: teams no longer know the architecture or the intent behind their own decisions, and everyone just asks Claude. He imports a vivid first-hand account of a big-company team where everything (specs, tests, PRDs, tickets) is Claude-generated and nobody reads anything, and closes on maintainability as the "final boss" that punishes exactly this kind of hollow team.

---

## What it argues

The frame is a deliberate inversion. Most AI-coding discourse argues about code quality; Späti concedes quality is roughly average and moves the real question to knowledge. If AI pulls a below-average codebase up to average, the code is fine. What is missing is the plan, the intent, the architectural reasoning — and, crucially, the humans who hold them.

## Key quotes

> "To me, the problem is not the AI code, but that nobody knows anything, and everyone just asks Claude. You end up with no plan whatsoever."

The thesis in one line. The unit of loss is not correctness but comprehension — the team as a collective mind goes blank.

> "Nobody on my team likes this. They are being forced to ship as much as they can... People are working 12 to 13 hours a day just to press enter. Nobody is reading anything."

The imported account (from "Voxium") is the empirical centrepiece: AI doesn't save labour here, it converts it into unread press-enter hours. That detail matters — the failure mode isn't laziness, it's management treating "pushing code is not a bottleneck" as a licence to stop thinking.

> "AI makes this obsolete, or **seemingly obsolete**."

On data engineers who grew up pre-AI and had to know everything. The hedge "seemingly" carries the whole essay: knowledge looks redundant right up until the moment you must judge whether the generated system is right.

> "The final boss is, and always will be, maintainability. The easier it is to generate a quick pipeline, app, or BI dashboard, the more you have to maintain."

Generation gets cheaper; maintenance doesn't — it gets *worse*, because the maintainers are the people who never understood the system.

## Key themes

#concept #software-engineering-craft — knowledge and intent as the scarcest resource, not code throughput.

#pattern — the comprehension hollow-out: a team where the loop runs but no mental model of the system exists in anyone's head.

#agent-coding-workflow — the "ask Claude for everything" day-to-day habit as an organizational disease, not an individual tooling choice.

## Analysis

This is a small, honest note rather than a argued essay, but it lands one sharp move: refusing to litigate code quality at all. That sidesteps the tired "is AI code good?" debate and names the actual scar — organizational amnesia. The weakest part is that the essay gestures at causes ("if we still hired juniors... but it's not as easy") without developing them; the note admits the hiring-pipeline argument exists but can't resolve it. Still, its diagnosis dovetails with the empirical 2026 literature: coding time compressed, everything downstream — review, understanding, ownership — starved. Where stronger treatments build taxonomies (cognitive debt, intent debt), this one is valuable precisely as a practitioner's gut report of the same phenomenon in one paragraph.

## Related pages

- [[Cognitive Debt]] — this source is the practitioner's anecdotal ground for the same claim that not-reading accrues a debt billable later; it strengthens that page's thesis with the 12-hours-of-pressing-enter texture.
- [[Nobody Knows How Large Software Projects Work]] — complicates it: that page says nobody ever knew the whole system even pre-AI; Späti's point is the sharper collapse from partial knowledge to none.
- [[Five Studies That Are Changing How I Think About AI in Software Engineering]] — supplies the research mirror of this note's gut feeling: upstream compressed, downstream understanding breaking.
- [[Who Owns the Code Claude Wrote]] — the ownership question is a downstream consequence of the no-plan team this note describes.

---
*Sources: [[raw/the-problem-is-not-the-ai-code-but-nobody-knows-anything-anymore]], [[summary/the-problem-is-not-the-ai-code-but-nobody-knows-anything-anymore]]*
*Last updated: 2026-09-29*
