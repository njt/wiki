---
url: https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs
title: "grill-with-docs (Matt Pocock Skills)"
author: Matt Pocock
date_fetched: 2026-11-03
date_published: 2026 (undated)
topics:
  - claude-code
  - agent-coding-workflow
---

The `grill-with-docs` skill page from Matt Pocock's skills repository, captured with its two constituent skills and the repo's own documentation. It composes two primitives: `grilling`, an interview skill that stress-tests a plan, and `domain-modeling`, which maintains a `GLOSSARY.md` and ADRs while you design.

`grilling` maps the subject as a **design tree** — decisions with decisions hanging off them — and interviews in **rounds**: each round asks the whole **frontier** (every question whose prerequisites are settled), numbered, with the agent's recommended answer attached so the user can reply by number. Facts are the agent's job (dispatch a sub-agent to look them up rather than asking); decisions are the user's, and the agent "has broken the skill" if it answers its own. The session ends when the frontier is empty, and the agent must not act until the user confirms shared understanding.

`domain-modeling` is the active discipline of sharpening the domain model during design: challenging terms that conflict with the glossary, splitting overloaded words, inventing edge-case scenarios, cross-referencing claims against the code, and writing terms into `GLOSSARY.md` inline. ADRs are offered only when a decision is hard to reverse, surprising without context, and the result of a real trade-off. The docs include known failure modes: models writing implementation detail into the glossary (the most-reported problem), skipping the confirmation gate, and skills not reliably loading each other.

The skill sits in a family: `grill-me` (no working directory), `grill-with-docs` (writes glossary + ADRs), `wayfinder` and `triage` reuse grilling as a primitive.
