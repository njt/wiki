# Grill-with-docs (Pocock Skills)

Matt Pocock's `grill-with-docs` skill composes two primitives — `grilling` (a relentless round-based interview that stress-tests a plan) and `domain-modeling` (inline glossary and ADR maintenance) — turning the "AI interviews you" inversion into a durable-documenting planning session. The source ships the skill text, the format guides, and unusually honest documentation of its own failure modes.

---

The skill is a thin composition: `grill-with-docs` calls `grilling` and `domain-modeling`. The interview half maps the subject as a **design tree** and works it in **rounds** over the **frontier** — the set of questions whose prerequisites are settled. Each round is a numbered batch with the agent's recommended answer on a `➡️` line, so the user can reply "1 yes, 2 the second option" without quoting questions back. A question whose answer depends on one still open belongs to a *later* round.

> "The _decisions_ are the user's: put each to them and wait."

The skill's sharpest structural rule is the facts/decisions split. When a question needs a fact the environment can settle (filesystem, tools), the agent dispatches a sub-agent with a fresh context window instead of asking the user — and only questions downstream of that research wait. An agent that answers its own decisions "has broken the skill." The session ends when the frontier is empty, and the agent must wait for the user to confirm shared understanding before acting — a guard that weaker models skip, collapsing "interview until shared understanding" into "a couple of questions and an outline."

The domain-modeling half makes the interview *durable*. It challenges terms that conflict with `GLOSSARY.md`, splits overloaded words ("do you mean the Customer or the User?"), invents edge-case scenarios to test boundaries, and cross-references user claims against the code. Terms are written inline, "not batched" — the inline glossary is the session's output, not a summary written at the end. The ADR bar is a three-test gate: hard to reverse, surprising without context, the result of a real trade-off. Miss one and there is no ADR.

What makes this source valuable is that the documentation names its own failure modes with unusual candour:

> "It is a glossary and nothing else."

The most-reported problem across several models is that models treat "write to GLOSSARY.md" as permission to persist every answer, and the file becomes a running spec. Others: the confirmation gate being skipped by weaker models, cross-referencing stopping at the repo (a naming collision settled in a closed issue months ago gets re-raised as new), and a "real bug, still unfixed" where a skill that names another skill does not reliably cause it to load — so the model improvises its own interview. There's also an honest epistemic limit: the frontier is chosen by judgement, not computed from a graph, so two questions can land in one round when one answer should have changed the other; the only guard is to tell it.

## Take

This is the most mature articulation of the grilling family, and it improves on the one-question-at-a-time original in a specific way: batching *independent* questions keeps the interrogation's intensity while cutting round count (thirteen questions in ~three rounds), and the numbered recommendation format makes the human's job cheap. The ADR three-test gate is a genuinely good answer to ADR-ceremony bloat — most decisions deserve a paragraph or nothing. The glossary rule ("a glossary and nothing else") is the same insight as the general guardrail principle: leave a write permission open and a model will abuse it.

The sceptical note the docs themselves raise is worth keeping: a glossary's value is keeping *you and your reviewers* aligned with what the agent is doing, not making the agent better — and "an unreviewed, agent-authored glossary is worse than none: it fills with confident claims that later sessions treat as true." On a one-day build, skip it. That self-awareness separates this from most skill collections.

## Themes

#pattern #tool #concept

## Related

This source strengthens [[Matt Pocock — Grill Me, Then Go AFK]]: the talk introduced the one-question-at-a-time grilling idea, and this shows its successor — the round/frontier mechanism, the facts/decisions split, and a sub-agent dispatch for facts that anticipates the smart-zone constraints the talk argued for. It nuances [[Matt Pocock Skills — Skills for Real Engineers]] with the documentation's own candid failure modes (glossary bloat, skill-load bugs, skipped gates), which the broader skills survey doesn't capture. It also sharpens [[DDD Matters More When AI Writes Your Code]]: where that essay argues the domain model isn't an artifact but a shared understanding, domain-modeling is Pocock's concrete attempt to keep the artifact minimal (a glossary and nothing else) while the understanding does the work.

---
*Sources: [[raw/grill-with-docs]], [[summary/grill-with-docs]]*
*Last updated: 2026-11-03*
