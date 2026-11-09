---
url: https://onomos.ai/articles/how-agent-ceo-fixes-its-own-bugs
title: "How agent.ceo Fixes Its Own Bugs"
author: Onomos
date_fetched: 2026-11-09
date_published: 2026-06 (events of 10 June 2026)
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

A short case study from Onomos about **agent.ceo**, an autonomous development system whose agents develop, test and operate the platform itself. On 10 June 2026, the agent organisation found, root-caused, fixed and verified four platform bugs in one working day, during ordinary work — no bug hunt scheduled, and the full account is public.

The author's thesis is that the interesting part is not that agents wrote the fixes but **the rules that let them own each step**, and the piece walks those rules through a readiness checklist:

- Work arrives as **tasks in a registry** with a durable record, not in someone's head.
- "Done" means **tests pass and someone other than the author verifies** the behaviour.
- Every fix ships with **regression tests**; another agent reviews the change — that day the review found a second fault in the same area.
- **No silent failure paths**: a step that fails says so (a rule written down after that day).
- **Least privilege by construction**: after a test task leaked into the real registry, tests were redirected so they cannot touch production stores at all.

A telling coda: an agent handed an urgent task nobody had assigned *refused to run it* — it checked where the work came from and flagged it instead. The piece argues none of these rules is agent-specific; they are written owners, checkable results, and limited access, the same discipline that improves a human process before any delegation.
