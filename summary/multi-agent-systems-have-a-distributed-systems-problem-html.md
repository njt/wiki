---
url: https://christophermeiklejohn.com/ai/agents/distributed/zabriskie/2026/03/30/multi-agent-systems-have-a-distributed-systems-problem.html
title: "Multi-Agent Systems Have a Distributed Systems Problem"
author: Christopher Meiklejohn
date_fetched: 2026-09-25
date_published: 2026-03-30
topics:
  - agent-orchestration
  - distributed-systems
---

Christopher Meiklejohn — CRDT researcher (Lasp, Partisan, Filibuster, Basho/Riak alum) — argues that multi-agent systems inherit, by construction, the exact failure categories distributed systems spent fifty years solving. The essay is grounded in his own experience running multiple Claude Code instances on one codebase (Zabriskie), where two worktrees both created migration 267 and one silently overwrote the other: a textbook lost update.

He surveys the coordination layer of ChatDev, MetaGPT, and AutoGen and finds the same gap in each: shared mutable state with no concurrency control, no causal ordering across agent interactions, no fault model, and no recovery path. MetaGPT's pub-sub message pool is an improvement over ChatDev's dialogue-only coordination, but it still doesn't tell an agent whether it's reading a stale version or about to write a conflicting change.

The core claim is inevitability: conflicts and stale reads, crash recovery, ordering without a clock (Lamport's happened-before), partition tolerance, and Byzantine faults (every hallucinating agent is a Byzantine actor) are properties of the architecture itself — multiple writers, no shared clock, partial failure — and emerge regardless of prompt engineering. The interesting open question he poses is not whether the parallels hold but how much of the CRDT / version-vector / fault-injection toolkit actually transfers when the replicas hallucinate and lose context instead of merely crashing.
