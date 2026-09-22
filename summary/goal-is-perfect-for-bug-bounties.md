---
url: https://xcancel.com/forefy/status/2094337553683878204
title: "/goal is perfect for bug bounties"
author: forefy (@forefy)
date_fetched: 2026-09-13
date_published: 2026-08-31
topics:
  - claude-code
  - guardrails-and-feedback-loops
---

forefy's tweet on Claude Code's `/goal` command, pitched at bug bounties. The
mechanism: `/goal` is a wrapper around a session-scoped, prompt-based Stop
hook. When a session hits Stop, the harness evaluates the condition and one of
three things happens — the goal is clearly not met (keep working, take the
reason as guidance), the goal is met (clear the Stop hook, done), or the goal
is impossible (clear the Stop hook, failed).

Authoring a goal is "practically playing with balancing forces": under-specify
a hard goal and the model concludes it's impossible; define a poor stop
condition and the model may waste tokens for hours or days; omit guardrails
and the model can break something like scope adherence. The tip that carries
the advice: "if it takes you less than 20 minutes to write a goal you're doing
it wrong." Delegating goal authorship to Claude itself is dismissed as
"like doubling down the milk to improve your coffee"; forefy instead links an
authoring helper (forefy.com/asr/goals/new) and a schema definition for those
who insist on one-shotting it with their agent.

The replies sharpen the mechanism. Federico (@DevCalledFede) observes that
whatever judges the condition reads the same session that produced the work,
so a goal phrased as a judgment call "gets graded from inside" — "the ones
that hold end on a check it can't author." forefy agrees the end condition is
the grading mechanics, points to dynamic workflows' fine-grained schemas
("the agents can't defy the schemas you set"), and closes that goals bring
back the "O.G. development mindset — make the function(prompt) perfect,
engineer every corner, worry about your function(prompt) looking beautiful to
the guy after you."
