---
url: https://everything.intellectronica.net/p/five-mistakes-people-make-with-agentic
title: "Five Mistakes People Make with Agentic"
author: intellectronica (Everything newsletter)
date_fetched: 2026-10-14
date_published: unknown (2026)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A "Factory Life" talk turned essay: five recurring mistakes that stop people shifting from supervising agents step-by-step to delegating outcomes. "Factory mode" is defined up front as giving an agent an outcome, the context it needs, and a way to check its work, then letting it carry the task through — including problems along the way — and returning a result you can inspect.

**Mistake 1 — Micromanaging.** Two variants: falling back into chat mode where the plan stays in your head, and writing instructions so detailed the agent can't adapt. The author's own "always write tests" standing rule produced a test for a README. The practical test for delegation: could you hand over the task and come back an hour later to the result you meant?

**Mistake 2 — Using the best, most expensive model for everything.** Background jobs silently burn quota; reasoning effort, latency and quality bar should be matched per task. The right comparison is the total cost of getting an acceptable result, including retries and repairs — a cheap attempt that keeps failing can cost more than an expensive one that lands.

**Mistake 3 — No guidance, taste, or way to check work.** Agents fill gaps with generic patterns, producing polished output that misses the point. The fix is giving context a home: standing instructions (AGENTS.md), examples with explanations of why they work, and skills/tools. Crucially, when the same mistake recurs, diagnose *where* the process failed rather than reflexively adding another emphatic sentence.

**Mistake 4 — The puppet show.** Inventing a cast of agent personas (CEO, strategist, critic) before working out which functions actually need separation. Start with one capable agent; only split when there's a distinct function — parallelism, different tools, narrower permissions — and remove splits that only add handoffs.

**Mistake 5 — Treating rules, skills, and prompts as deterministic infrastructure.** A sentence like "do not publish without approval" enforces nothing if the agent retains the capability to publish. Instructions are interpreted, and behaviour varies with model version, context, and retrieved material. Real enforcement lives in the system around the agent: file permissions, sandboxes, identity without publish rights, and pre-action checks that block on failure — and the whole route must be checked, not just one UI surface.

The essay closes with five diagnostic questions for any task that needed more supervision than expected, plus a plug for the author's Maven course with Hugo Bowne-Anderson.
