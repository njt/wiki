---
url: https://github.com/jwyjohn/acl26-silo-bench
title: "SILO-BENCH (GitHub repository)"
author: Yuzhe Zhang, Feiran Liu, Yi Shan, Xinyi Huang, Xin Yang, Yueqi Zhu, Xuxin Cheng, Cao Liu, Ke Zeng, Terry Jingchen Zhang, Wenyuan Jiang
date_fetched: 2026-09-25
date_published: 2026-03-01
topics:
  - agent-orchestration
  - coding-agents-and-frameworks
---

SILO-BENCH's GitHub repo is the reference implementation behind the arXiv paper (2603.01045): a role-free benchmark where every agent in a multi-agent LLM system holds only a private data shard and must discover its own communication strategy to converge on a correct global answer. No pre-assigned roles, no scripted coordination — the topology that emerges is part of the measurement.

The evaluation matrix is 30 tasks (10 per complexity level: Aggregation O(N), Mesh Network O(N), Global Shuffle O(N log N)–O(N²)) × 6 agent counts (2 to 100) × 3 communication protocols (P2P messaging, broadcast, shared file system) = 54 configurations, all with round-based synchronization: writes in round *t* become visible in round *t+1*. Four metrics capture both outcome and process: Success Rate (S, proportion of agents correct), Partial Correctness (P, continuous per-task quality), Token Consumption (C), and Communication Density (D, messages per directed pair).

The headline finding is the **Communication-Reasoning Gap**: agents spontaneously form task-appropriate topologies and exchange information actively, yet systematically fail to synthesize distributed state into correct answers — the failure is localized to the integration stage, and coordination overhead compounds with scale until it eliminates parallelization gains entirely.

The codebase itself is a small (~5,400-line) Python harness: a shared execution engine (`src/engine.py`) parameterized by protocol plugins, Pydantic state models, XML-based tool calling (no native function calling), JSON-file persistence for every round and every agent, and a benchmark generator that produces provably-correct task instances with executable verification logic.
