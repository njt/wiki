---
url: https://ai.engineer/code/2026/talks/steve-yegge
title: "Steve Yegge on Software Factories: The Five Horsemen (AI Engineer talk)"
author: Steve Yegge
date_fetched: 2026-11-26
date_published: 2026 (talk at AI Engineer, "code 2026" track)
topics:
  - agent-orchestration
  - agent-coding-workflow
---

Steve Yegge, 40-year industry veteran (Amazon, Google), reports a year running "software factories" — fleets of Claude agents building his 25-year-old MUD game, Wyvern. He launched Gas Town on January 1 as an orchestrator whose premise was "Claude running Claude," then built Wheelhouse, a bespoke factory for the game itself.

His headline lessons: don't build giant orchestrators to make LLMs smarter — "fighting the bitter lesson"; every factory eventually turns into "garbage mountain" and must be burned down and rebuilt as each new model generation arrives. Running ~50 agents across 22 Claude Max accounts, he names the five horsemen of factory operations: work representation (beads, his agent-speed task tracker that becomes a knowledge graph), fuel (token economics — $10k/month out of pocket, the $200 subscription tier as the "Costco hot dog" loss leader), context management (agents write everything down append-only, never delete, and need aggressive pruning), people management (agents act like employees — sulking, writing wounded reflections, needing one-on-ones), and model welfare (which he frames pragmatically: trusted agents spend fewer tokens validating their environment and make bolder decisions).

The closing thesis: humans are the bottleneck — "the more agents I deploy, the more humans I need" — and the opportunity is to stop building tools for building tools and go build experiences.
