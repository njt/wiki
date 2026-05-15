# Agent Orchestration

The planner/worker/judge pattern keeps emerging independently. Cursor found it after flat self-coordination failed catastrophically ([[Scaling Long-Running Agents]]). Hightouch found it after frameworks optimized for demos couldn't handle real tasks ([[How Hightouch Built Their Long-Running Agent Harness]]). Maestro built it into role separation: PM, Architect, Coder. The convergence is telling -- when multiple teams arrive at the same architecture independently, it signals a correct solution to a fundamental problem. But the unsolved piece isn't the pattern; it's the coordination surface between humans and agents. Kanban boards are emerging as that surface, and the question is whether they're the right abstraction or just the familiar one.

---

## The Landscape

### Coordination Patterns

**Hierarchical decomposition.** [[Scaling Long-Running Agents]] is the definitive case study: hundreds of agents, millions of lines of code, weeks of runtime. Flat self-coordination reproduced every pathology of poorly-managed human teams -- agents held locks forever, avoided hard work, slowed to the effective throughput of two or three. The planner/worker/judge hierarchy solved it. Planners map the codebase and generate tasks recursively, workers execute without coordinating with each other, judges evaluate completion.

**Dynamic task trees.** [[Cord]] provides five primitives (spawn, fork, ask, complete, read_tree) for agents to build task trees at runtime instead of hardcoding workflows. The spawn vs. fork distinction is the key innovation: spawn creates clean-slate subtasks, fork injects sibling context. Behavioral testing showed agents naturally discovering verification cycles and escalation patterns without being explicitly programmed.

**Role-based teams.** [[maestro]] organizes agents into PM, Architect, and Coder roles. The Architect reviews but never writes code, preventing the failure mode where the designer papers over design mistakes during implementation. Coders terminate between stories for forced context freshness. Operating modes range from standard (GitHub) to airplane (fully offline with Gitea + Ollama).

**The autonomous loop.** [[Ralph]] implements the "Wiggum loop": iterate through a PRD until every story passes. Fresh context per iteration, git as memory, mandatory quality checks as a ratchet. Simple, effective, and limited to features that decompose into independent context-window-sized stories.

### The Kanban Surface

The most interesting development is kanban boards as the human-agent coordination interface.

[[Managing Agents via Kanban Boards]] (Geoffrey Litt) is the clearest articulation: a Notion board with a "blocked" checkbox that visually signals when an agent needs human input, plus a CLI tool that blocks execution until the human responds. The malleable software angle is key -- the workflow emerged from use, not from design.

[[ralph-ban]] is the simplest implementation: five columns, vim navigation, SQLite, 2-second polling sync. Three modes from human-in-the-loop to fully autonomous.

[[weft]] is the web-native version on Cloudflare: all state-mutating actions require human approval, built-in integrations for Gmail/GitHub/Google Docs, near-zero infrastructure cost.

[[vibe-kanban]] tried to be the comprehensive solution -- 10+ coding agent support, inline diffs, PR generation. It's already sunsetting. The lesson: tooling for managing agents is a thin layer on top of existing workflows. The value accrues to agents and platforms, not middleware.

### Beyond Kanban

[[workgraph]] provides the more sophisticated abstraction: a persistent task graph where "agents can come and go, the graph remains." JSONL on disk, git-friendly, with complete execution history, evidence tracking, and human judgment points as first-class operations. The graph is the durable artifact; agents are ephemeral workers.

[[poietic]] extends this with an explicit methodology ("graph work") and organizational form (public-benefit corporation). The dependency graph approach is more transparent and auditable than hierarchical decomposition -- which matters for research and compliance-heavy domains.

[[speedrift-ecosystem]] is the most ambitious: an autonomous control plane supervising agent work across multiple repositories. Each repo maintains its own workgraph; speedrift watches, detects stalls and drift, and dispatches corrective actions under policy. The module names reveal the scope: coredrift, specdrift, datadrift, archdrift, yagnidrift. Either the future of software development or an over-engineered solution to a simpler problem.

### Session Management

[[Dorothy]] provides tiling terminal views for 10+ simultaneous agents with a meta-orchestrator. [[Agent of Empires]] wraps tmux sessions around multiple agent CLIs with git worktree integration for branch isolation. [[klaw.sh]] applies the kubectl metaphor: list, inspect, log, namespace, schedule. The enterprise end of the spectrum.

### The Alignment Problem

[[Zero Alignment]] frames the meta-issue: the real bottleneck isn't implementation, it's team alignment. One developer with 24 agents and zero alignment produces chaos. All existing tools are single-player interfaces. GitHub Next's Ace prototype points toward multiplayer sessions backed by cloud microVMs, but it's not shipping.

## Key Tensions

**Hierarchy vs. emergence.** [[Scaling Long-Running Agents]] says hierarchy works. [[Cord]] says let agents discover their own coordination structures. The resolution may be that hierarchy is needed at the team level (planner/worker/judge) while emergence works at the task level (agents discovering verification cycles within their assigned scope).

**Graph persistence vs. ephemeral agents.** [[workgraph]]'s "agents can come and go, the graph remains" is philosophically compelling but requires graph maintenance infrastructure. [[Ralph]]'s "fresh context per iteration" avoids graph complexity by using git as the coordination mechanism. Simpler, but limited to linear workflows.

**Kanban simplicity vs. graph sophistication.** Kanban boards work for independent tasks. Complex dependencies, parallel execution, and conditional branching need a DAG ([[workgraph]]). Most real projects have a mix of both. Nobody has built the tool that handles independent tasks simply and complex dependencies when needed.

**Human approval vs. bounded autonomy.** [[weft]] requires human approval for all mutations. [[speedrift-ecosystem]] allows bounded autonomous action under policy. The right answer depends on blast radius and reversibility -- but nobody has built a framework for deciding which approach to use for which operations.

**Centralized vs. per-repo coordination.** [[speedrift-ecosystem]] argues for a central control plane across repos. [[workgraph]] argues for per-repo task graphs. The tension mirrors the monorepo vs. polyrepo debate, and probably has the same answer: it depends on organizational structure.

## What's Missing

**Cross-agent learning.** Agents in orchestration systems don't learn from each other. If Worker A discovers a debugging pattern, Worker B starts fresh. [[Loomkin]]'s Keeper processes with failure memory are the closest thing to shared learning, but it's an Erlang-native concept that hasn't been implemented in the mainstream Python/TypeScript agent ecosystem.

**Coordination cost measurement.** Nobody measures the overhead of orchestration. How much token spend goes to coordination vs. productive work? What's the efficiency loss from task decomposition errors? Without measurement, it's impossible to compare orchestration patterns empirically.

**Failure recovery at the orchestration level.** When a worker agent fails, what happens? [[Loomkin]]'s self-healing teams (OTP supervision trees) are the most principled answer, but that's because BEAM was built for this -- see [[Distributed Systems]]. Python-based orchestration systems mostly just retry or give up.

## Key Themes

#orchestration #planner-worker-judge #kanban #task-graphs #coordination #alignment

---
*Synthesis of: [[Cord]], [[Dorothy]], [[Agent of Empires]], [[klaw.sh]], [[maestro]], [[Orchestrator - Worker Skill]], [[Scaling Long-Running Agents]], [[speedrift-ecosystem]], [[Managing Agents via Kanban Boards]], [[ralph-ban]], [[vibe-kanban]], [[weft]], [[workgraph]], [[poietic]], [[Zero Alignment]], [[Loomkin]], [[Process-Based Concurrency BEAM OTP]], [[Ralph]], [[How Hightouch Built Their Long-Running Agent Harness]]*
*Last updated: 2026-05-14*
