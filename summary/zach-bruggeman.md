---
url: https://ai.engineer/code/2026/talks/zach-bruggeman
title: "Culture Lessons from Building Inspect at Ramp"
author: Zach Bruggeman
date_fetched: 2026-11-26
date_published: 2026 (AI Engineer Code conference talk)
topics:
  - agent-coding-workflow
  - coding-agents-and-frameworks
---

Zach Bruggeman, an engineer at Ramp, gives a talk about the culture that grew
around Inspect, Ramp's in-house collaborative coding agent, now shipping ~75%
of all Ramp code. He deliberately skips the plumbing (sandboxes in under 5
seconds, multiplayer sessions, integrations with Slack/Linear/Data Dog) and
focuses on three cultural lessons.

**Lesson 1: kill information silos.** One Inspect for the whole company acts
as a solid baseline, so nobody has to maintain a private 15-minute agent setup
that goes stale as best practices move. Improvements found by anyone propagate
to everyone, mostly invisibly — so no formal "how to use AI" training is
needed.

**Lesson 2: build trust in public.** Every Inspect session is public inside
the company by default. A PR review shows not just the diff but the full
conversation, tool calls, screenshots, and a video of the verification agent
clicking through the app. He tells the story of a bug where Inspect (correctly,
initially disbelieved) diagnosed its own telemetry instrumentation as the
cause; the whole debugging happened in a shared thread and turned a skeptical
bug reporter into an engaged colleague within an hour.

**Lesson 3: think bigger together.** He pushes back on the "lone wolf
operator" ideal — individual AI leverage without outward collaboration
"will collapse in on itself." Two engineers built Ramp's code review agent
(Review Buddy) on Inspect's automation triggers without asking permission,
which pushed the Inspect team to see the product as more than a coding agent.
His closing challenge: ask your team "what have we been waiting to do that we
haven't done yet?" — everyone has access to the same models, so the excuse is
gone.
