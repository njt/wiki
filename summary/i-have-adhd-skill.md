---
url: https://github.com/ayghri/i-have-adhd/blob/main/skills/i-have-adhd/SKILL.md
title: "i-have-adhd (SKILL.md)"
author: ayghri
date_fetched: 2026-08-01
topics:
  - agent-architecture
---

A Claude Code skill that reformats LLM output for readers with ADHD. The rules
are grounded in five facts about ADHD cognition: small working memory means
anything not on screen is forgotten; the gap between understanding and doing is
where work dies; starting is the hardest step; time estimates feel uniform; and
dopamine is scarce so visible progress matters.

Ten output rules follow: lead with the next action, not context; number
multi-step tasks into bounded single actions; end with one concrete next step
under two minutes; suppress tangents until the current task is done; restate
progress state every turn; give time estimates in specific units; make completed
work visible without burying wins in recaps; use a matter-of-fact tone for
errors; cap lists at five items; and forbid preamble, recap, and closing
pleasantries.

The rules persist for the entire session unless the reader says "stop adhd mode"
or "normal mode." Six exceptions are listed where the rules should be
overridden, including when the user asks for an explanation, when a destructive
action needs confirmation, and when a debug spiral suggests the wrong assumption
rather than the wrong code. A pre-send checklist ensures the first and last
lines alone convey what to do next and what just happened.
