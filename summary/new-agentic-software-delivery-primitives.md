---
url: https://commandline.microsoft.com/new-agentic-software-delivery-primitives/
title: "New Agentic Software Delivery Primitives"
author: Microsoft (commandline.microsoft.com)
date_fetched: 2026-10-28
date_published: 2026
topics:
  - agent-coding-workflow
  - agent-memory-and-context
---

Microsoft's essay argues the agentic software delivery system is best understood stage by stage, not as an end-to-end black box. The classic SDLC stages (Plan, Build, Verify, Observe) survive; what changes is where human attention goes and what artifact each stage hands to the next. The unifying idea is a shared **knowledge layer**: goals, design decisions, validation evidence, release history, and production signals all connect so that a production finding can reshape planning.

Key claims:

- Transforming the whole SDLC at once fails because you can't tell what's working; decompose per stage — inputs, human/AI jobs, output artifact, success criteria.
- Each stage keeps a human job-to-be-done: choosing problems and resolving ambiguity (Plan), guiding consequential design decisions (Build), judging whether the thing built is what was intended (Verify), interpreting signals (Observe).
- Verification is where the shift is sharpest: agents test across dimensions and produce "discrete, auditable evidence," moving human energy to "is this what we meant to build."
- Memory must serve a purpose: data is only useful when it lets an agent answer specific questions — "similar to a database index or a materialized view, the goal of memory is to give an agent quick access to data from the past to improve future performance." The test: does the agent do the job better with the synthesized knowledge than without?
- Human judgment concentrates in validation, operation, and planning; "humans and a refined sense of taste are necessary to identify and produce lasting value."
- Practice at Microsoft: Copilot coding agents producing draft PRs, Copilot Code Review with agent skills and MCP, and harness-packaged reusable skills.
