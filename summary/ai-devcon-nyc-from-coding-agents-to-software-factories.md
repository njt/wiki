---
url: https://tessl.io/blog/ai-devcon-nyc-from-coding-agents-to-software-factories
title: "AI DevCon NYC: From Coding Agents to Software Factories"
author: Patrick Debois
date_fetched: 2026-11-26
date_published: 2026-11-26
topics:
  - agent-coding-workflow
  - agent-orchestration
---

Patrick Debois (the DevOps coinage) previews the AI DevCon NYC programme (Nov 2–4, Industry City, Brooklyn) and uses it to lay out his thesis that the industry's organising question has shifted: not "how capable is a coding agent?" but "what system makes agentic development repeatable, trustworthy and scalable?"

He decomposes a working "software factory" into five mutually reinforcing layers: **context and skills** (Brandon Waselnuk on knowledge "emitted, not authored" — the decisions made during work that never reach a README or skills file), **harnesses and tools** (Ryan Lopopolo's harness-engineering framing; Dru Knox following a failure → guardrail → improved-harness reliability loop), **orchestration** (James Moss on what breaks going from one agent to ten), **verification and evaluation** (Nandini Singhal on Datadog combining telemetry, agent-session traces, task evals and delivery outcomes), and **feedback and improvement** (Shivam Jindal on SpaceXAI's use of Cursor with long-running agents).

The layers compound: better context raises throughput, which pressures orchestration and review; better verification permits autonomy, which raises governance stakes. The developer's role changes accordingly — the scalable fix to a repeated agent mistake is to improve the harness, context or verification, not to correct the instance.

On the enterprise layer, Debois argues single-developer wins don't transfer: organisations need to decide what belongs in a central platform, how skills are distributed, which models touch which systems, and whether any of it improves delivery. He highlights Bartosz Ocytko's Zalando session (agentic engineering across 2,000+ engineers: platform investment, governance, risk-based review, knowledge sharing) and names the surrounding practice "agent enablement" — with the software factory as the technical system that emerges around it.
