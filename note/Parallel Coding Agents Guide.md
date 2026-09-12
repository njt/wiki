# Parallel Coding Agents Guide

Avi Peltz's practical field guide to running multiple AI coding agents simultaneously: the orchestration patterns that work, the review bottleneck that emerges at scale, and the resource constraints that bite. Published on the Superset blog — part engineering advice, part product pitch, but the operational wisdom stands independent of tool choice.

---

## Key Quotes

> "The bottleneck isn't the agent or the model. It's the human orchestrating one task at a time."

This is the thesis the whole piece hangs on. If you're running agents sequentially — wait for one to finish, read its output, start the next — you're the bottleneck. The agents aren't the slow part; your attention is.

> "Even when agents edit different files, a shared git index means their commits can include each other's uncommitted changes."

The isolation problem stated cleanly. Most engineers discover this the hard way after their first attempt at `tmux`-ing two agents side by side. Git worktrees solve it — each agent gets its own staging area — but you have to know to reach for them.

> "10 agents each producing a diff every 15 minutes creates 10 diffs to review per hour."

The review bottleneck quantified. This is the part nobody talks about in the "100 parallel agents" hype. Generation scales linearly with agents; review capacity doesn't. The five strategies offered (prioritize by risk, review diffs not files, let agents self-verify, structured task descriptions, batch similar tasks) are solid, but they're mitigations, not solutions.

> "If the agent doesn't run tests, you're reviewing blind."

The sharpest line in the piece. Include test execution in the agent's prompt. Without it, you're doing code review without a compiler — guessing at correctness.

> "The key insight: you don't have to choose one agent for everything."

The agent selection matrix (Claude Code for complex refactors, Codex for cost-sensitive autonomous work, OpenCode for model flexibility, Aider for iterative pair programming) is pragmatic. Each agent has a behavioral profile that suits different tasks. Mixing them isn't complexity — it's matching tool to job.

---

## Key Themes

- **#pattern** — **Worktree-per-agent isolation.** The foundational primitive for parallel agents. Each agent gets its own branch and working directory. Without this, parallel agents corrupt each other's state. [[Agent of Empires]] and [[acpx]] both implement this pattern; [[workgraph]] uses it as the default isolation strategy.

- **#tool** — **Agent selection matrix.** Claude Code (complex), Codex (autonomous/cost-sensitive), OpenCode (flexible/local), Aider (iterative/pair). This echoes the multi-model approach in [[Inside the AI Workflows of Every's Six Engineers]] and [[On a Year of Multi-Model Development]].

- **#concept** — **The review bottleneck.** Generation throughput scales with agent count; review throughput doesn't. This is the same tension [[Cognitive Debt]] identifies from the velocity side and [[Zero Alignment]] identifies from the coordination side. The article's five mitigation strategies are a starting point, but the real answer is in [[Compound Engineering]] and [[Guardrails and Feedback Loops]] — automated verification that reduces what humans need to review.

- **#pattern** — **Structured task descriptions.** Vague prompts produce unreviewable diffs; detailed prompts with explicit constraints produce scoped, auditable changes. This connects directly to [[How to Write a Good Spec for Agents]] and [[Specifications as the Product]].

- **#concept** — **Orchestration maturity tiers.** Manual worktrees → scripted orchestration → dedicated orchestrator. This is a useful taxonomy even if tier 3 is the Superset pitch. [[Agent Orchestration]] tracks the same progression across the broader ecosystem. Spotify's [[Xirp]] is tier-3 made real — and its Portal context layer (Backstage Catalog + Workspaces over MCP) adds an organizational-context axis these tiers don't capture.

---

## Critical Analysis

**The good:** The isolation problem and review bottleneck are real, under-discussed operational constraints on parallel agents. The worktree-per-agent pattern is the right primitive. The advice to start with 2-3 agents, batch similar tasks, and write structured prompts is actionable and tool-agnostic. The agent selection matrix, while high-level, is a useful starting framework.

**The product-shaped hole:** This is a Superset marketing piece. Tier 3 ("dedicated orchestrator") is their product. The article frames the progression as inevitable — manual doesn't scale, scripts lack persistence, therefore you need a platform — but skips the middle ground entirely. You can get very far with a shell script, worktrees, and discipline before needing a dedicated orchestrator. [[Ralph]] and the [[repoMirror]] while-loop agent prove that simple prompts and simple loops beat complex platforms surprisingly often.

**What's missing:** The article doesn't address merge conflict resolution at all beyond "ignoring branch conflicts" as a mistake. If two agents modify overlapping regions of the same file on different branches, you get a merge conflict — and no amount of orchestration prevents it. The real answer is task decomposition that avoids overlapping file access, which requires understanding the codebase's module boundaries before dispatching agents. This is the problem [[Cord]]'s spawn/fork/ask primitives and [[Scaling Long-Running Agents]]'s planner/worker/judge pattern are trying to solve.

**The review story is optimistic.** "Focus on what changed" and "let agents verify their own work" assume agents reliably run tests and produce scoped diffs. In practice, [[AI Coding Tools Create More Bugs Than They Fix]] and [[Benchmark Exploitation]] suggest agents are less reliable than this advice assumes. The real review workflow needs the verification pipeline from [[Compound Engineering]] and [[Harness Engineering]], not just faster human attention.

**The 5-7 concurrent agent claim needs context.** That number depends heavily on what the agents are doing. Seven agents running Claude Opus on complex refactors will burn through API rate limits and CPU differently than seven Codex agents running o4-mini on test generation. The resource discussion is honest about the constraints but the "5-7 comfortable" framing is marketing, not engineering.

---

*Sources: [[summary/parallel-coding-agents-guide]]*
*Last updated: 2026-05-15*
