---
url: https://www.autoencoders.com/papers/reinvent-software-tickets-for-the-ai-era/
title: "It's Time to Reinvent Software Tickets for the AI Era"
author: AskFutures (autoencoders.com)
date_fetched: 2026-11-17
date_published: unknown
topics:
  - agent-coding-workflow
  - agent-orchestration
---

AskFutures argues ticketing systems were designed around "who's working on what," a question that stops being the most interesting one when coding agents do the implementation. Their three jobs for Linear tickets: record decisions, orchestrate parallel agents, and define what ships together.

Tickets become the durable memory of *why* — "code remembers what changed; tickets remember why" — created only for decisions and meaningful deliverables, not for every internal step an agent takes. A parent ticket is divided into child tickets with encoded dependencies: independent children run in parallel, blocked ones wait. The unit of work is redefined around agent independence: a good child ticket is not a small task but one an agent can complete with minimal coordination and minimal conflict with others. Integration is batched — several child tickets feed one pull request for the parent, amortising slow merge gates across parallel work instead of paying the full validation cost per small task.

For discoverability, a small text file acts as a decision index: short, searchable descriptions with pointers (not copies) into Linear, exploiting three levels of progressive disclosure (map → ticket → code/PR). Descriptions are written for agent search ("search layout, pagination, chart height") rather than taxonomy. Linear remains the source of truth; the index is only the map. The closing line: "We don't use tickets to count how busy agents are. We use them to keep parallel work coherent."
