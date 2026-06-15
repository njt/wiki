# Fleet of Agents (sermakarevich)

A five-step tutorial that builds from a single Claude Code agent writing work to disk all the way to a fleet of parallel agents sharing a queue and asking questions when stuck. The tutorial's concepts were later productionized as [[Fleet Supervisor (sermakarevich)]], a full Python supervisor with web UI, Telegram integration, and MCP-based human-in-the-loop. Each step is self-contained and independently useful — you can stop at any point with a working setup. The progression is the best explicit walkthrough of the Ralph Wiggum loop graduation path I've seen: it makes teachable what other practitioners describe anecdotally.

---

## Architecture

The tutorial constructs a progressively sophisticated agent harness in five layers, each building on the last:

**Step 1 — Filesystem as memory.** Agent writes `TASK.md`, `PLAN_AND_PROGRESS.md`, and `FINDINGS.md` into `.claude/tasks/<task_id>/`. One agent, manually driven, but every task is resumable. This is [[Planning With Files]] applied at the per-task level.

**Step 2 — On-disk queue.** `TODO.md` and `DONE.md` replace the ad-hoc "what should I work on next?" problem. Agents take the top entry, work, log, and remove. You manage a backlog but still drive each iteration manually. This is the queue primitive that [[Ralph]] and [[RepoMirror]] both depend on.

**Step 3 — Bash loop.** A ~15-line `./loop.sh` invokes `claude -p` repeatedly until the queue empties. Nothing in the agent changes — the loop is a pure re-invocation mechanism. This is the [[Ralph]] pattern at its most minimal: a dumb loop around a smart agent.

**Step 4 — Parallel agents with atomic claiming.** Replaces markdown queues with Gas Town's [`beads`](https://github.com/gastownhall/beads) (`bd` CLI), a SQLite-backed task database. Atomic `bd update --claim` with unique `BEADS_ACTOR` identifiers lets multiple `./loop.sh` instances run in parallel without race conditions. Beads also manages dependencies, blocking tasks with unfinished parents. This is the coordination layer that [[What Ralph Wiggum Loops Are Missing]] argues freeform markdown can't provide.

**Step 5 — Q&A blocking.** Since `claude -p` can't use `AskUserQuestion`, agents write questions to per-task `Q&A.md` files and mark tasks blocked. The loop skips blocked tasks, so one stuck agent doesn't stall the fleet. The human answers, flips status back to open, and the agent resumes. The critical design decision: `loop.sh` is **byte-identical** to step 4's. The loop stays dumb; all intelligence lives in the agent and the queue.

## Key Quotes

> "you stop being the agent's memory"

The thesis of step 1, compressed to six words. When the agent writes its context to disk, the human is freed from carrying task state across sessions. This is the [[Planning With Files]] insight applied to the individual task level.

> "nothing in the agent changed"

The loop is a dumb re-invocation mechanism. The agent doesn't know it's in a loop — it just receives the same prompt (`claude -p "take next task from TODO.md per CLAUDE.md"`) each iteration. This separation of concerns (smart agent, dumb loop) is the architectural insight that makes the whole thing work.

> "one blocked task doesn't stall the fleet"

The Q&A blocking mechanism's essential property. Because blocked tasks disappear from `bd ready`, other agents keep working on other tasks. This is the difference between a stopped agent and a stopped fleet — and it's achieved without any new loop logic.

> "the loop stays dumb"

The refrain that runs through steps 3-5. `loop.sh` is byte-identical from step 3 onward. All new capabilities (parallel claiming, dependency management, Q&A blocking) are implemented in beads or in the agent's behavior, never in the loop itself. This is the opposite of [[Gas Town's Agent Patterns]], where the orchestrator grows increasingly complex.

## Key Themes

- **#pattern** — **Dumb loop, smart agent.** The loop is a pure re-invocation mechanism. All intelligence lives in the agent's CLAUDE.md rules and the queue system. This inverts the instinct to build coordination logic into the orchestrator — instead, push it into the tools the agent uses. Contrast with [[Gas Town's Agent Patterns]] where the orchestrator (the "mayor") is the intelligent component.

- **#pattern** — **Progressive disclosure of autonomy.** Five steps, each self-contained and useful on its own. You can stop at step 1 (resumable tasks), step 2 (backlog management), step 3 (unattended agent), step 4 (parallel fleet), or step 5 (blocking-aware fleet). This is the best pedagogical structure for teaching agent orchestration I've seen — it maps cleanly onto the graduation path [[What Ralph Wiggum Loops Are Missing]] describes.

- **#tool** — **beads (bd).** Gas Town's SQLite-backed task database with atomic claiming and dependency tracking. The critical dependency for steps 4-5. Without atomic claiming, parallel loops would race on TODO.md edits. Without dependency tracking, agents would pick up tasks whose prerequisites aren't done. [[Gas Town After 10,000 Hours of Claude Code]] criticizes beads for polluting git history, but sermakarevich's tutorial doesn't address that trade-off.

- **#concept** — **Filesystem as agent memory.** Step 1's `TASK.md` / `PLAN_AND_PROGRESS.md` / `FINDINGS.md` pattern is [[Planning With Files]] instantiated as a per-task convention. The human stops being the agent's memory; the filesystem becomes the persistence layer. This is the foundation that makes steps 2-5 possible — without persistent task context, each loop iteration starts blind.

- **#concept** — **Q&A as blocking primitive.** The agent can't use interactive tools in `-p` mode, so it writes questions to a file and blocks itself. This is a workaround for a platform limitation (no `AskUserQuestion` in headless mode), but it's also a genuinely useful pattern: structured questions with `Context`/`Tried`/`Need` sections make the human's answering job faster than reading agent logs.

## Critical Analysis

**This tutorial is the best introductory material on agent fleets I've seen.** Most writing on the topic is either hype ("100 agents overnight!") or skepticism ("it doesn't work yet"). This tutorial is neither — it's a workshop manual. Each step produces a working system. The progression is logical and each increment is small enough to understand. The "stop at any step" framing is honest about the trade-offs at each level of autonomy.

**The "dumb loop" principle is the deepest insight here**, and it's easy to miss because it's stated so matter-of-factly. Most agent orchestration systems grow complexity in the orchestrator — Gas Town's mayor, Cord's spawn/fork/ask, Taskmaster's 39 MCP tools. This tutorial does the opposite: the loop never changes, and all new capabilities are pushed into the tools the agent uses (beads for queue management) or the agent's own behavior (writing Q&A files). This is [[Smart Models Dumb Pipes]] applied to agent orchestration — the agent is the smart component, the loop is the dumb pipe.

**The dependency on beads is both a strength and a weakness.** Beads solves the atomic claiming problem elegantly, and its dependency tracking is genuinely useful. But it introduces an external dependency (a Go binary from Gas Town) and, per [[Gas Town After 10,000 Hours of Claude Code]], pollutes git history with agent bookkeeping. The tutorial doesn't acknowledge this trade-off. An alternative using git worktrees and branch-per-task isolation (as [[Parallel Coding Agents Guide]] recommends) would avoid the dependency but add complexity elsewhere.

**The Q&A blocking is clever but fragile.** It depends on the agent recognizing when it's stuck and writing a question — which requires the agent to have good metacognitive awareness of its own uncertainty. Less capable models will either not write questions when they should (silently producing garbage) or write questions when they shouldn't (stalling on easy tasks). The `Context`/`Tried`/`Need` structure helps, but it's a prompt-level solution to what might be a capability-level problem.

**What's missing:** The tutorial doesn't address merge conflicts when multiple agents touch overlapping files. [[Parallel Coding Agents Guide]] identifies this as the hardest operational problem in parallel agent fleets, and the answer (task decomposition that avoids overlapping file access) requires understanding module boundaries before dispatching agents. Also missing: any discussion of review throughput. Ten agents producing diffs means ten diffs to review — generation scales, attention doesn't. The fleet creates work faster than a human can validate it.

**Compared to Ralph:** This is [[Ralph]] made explicit and teachable. Huntley's bash loop and snarktank/ralph are both single-agent patterns. This tutorial walks the same path (step 3 is essentially Huntley's technique) but extends it to multi-agent parallelism (step 4) and blocking-aware coordination (step 5). It's the missing manual for the graduation path [[What Ralph Wiggum Loops Are Missing]] describes.

**Compared to Gas Town:** This is Gas Town's beads tool applied in a simpler, more focused setting. Gas Town is a full orchestrator with a mayor, workers, and complex coordination. This tutorial strips it down to a bash loop + beads + agent rules. It's Gas Town for people who want to understand the pieces before assembling the whole machine.

---

*Sources: [[raw/fleet-of-agents-sermakarevich]]*
*Last updated: 2026-05-22*
