---
url: https://www.telerik.com/blogs/multi-agent-orchestration-software-delivery-patterns-multi-repository-workflows
title: "Multi-Agent Orchestration: Software Delivery Patterns for Multi-Repository Workflows"
author: Adam Bertram
date_fetched: 2026-09-29
date_published: 2026-09
topics:
  - agent-orchestration
  - agent-coding-workflow
---

Adam Bertram's Telerik essay on orchestrating coding-agent work across multiple agents *and* multiple repositories, opening with the canonical failure: 11 green pull requests across five repos in ninety minutes, and the authentication service is still broken — because no agent owned the space between its task and another's dependency. He insists multi-agent coordination and multi-repository coordination are two distinct problems that overlap without implying each other.

The essay's core arguments: (1) agents should *inherit* existing CI/CD controls — branch protection, code owners, versioning, migration ordering — rather than spawn parallel ones, and since every repo's "gate" differs (merge queue, reviewer group, nightly build), each needs its own integration; (2) isolation precedes parallelism, via Git worktrees from a fresh `origin/main`, with OS-level sandboxing as the only boundary an agent instruction can't cross and hotspot files (lockfiles, DI registries) single-edit-at-a-time; (3) handoffs between agents should be versioned artifacts with four fields — scope, interface requirement, base commit, and evidence — because research shows most multi-agent failures are workflow-design misalignments, not model-capability failures; (4) context must be durable (a dedicated Git branch for plans and run history, MCP-integrated tools that outlive any one session); (5) composition has an order — cross-repo dependencies create merge sequencing that per-repo CI can't see, requiring a deliberately designed integration run against combined state, with the merge decision staying human.

He closes with a prescription: replay last quarter's cross-repo change read-only and let the gaps surface — they'll be in interface requirements that were never written down. The prerequisite is a delivery workflow explicit enough to hand to something that won't ask what you meant. The piece concludes with a plug for Progress Forge (formerly Progress Agent Harness), Telerik's commercial orchestration product.
