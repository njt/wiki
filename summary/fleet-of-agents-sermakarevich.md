---
url: https://github.com/sermakarevich/claude/tree/main/usage_pattern/fleet_of_agents
title: "From one Claude agent to a fleet — in five small steps"
author: sermakarevich
date_fetched: 2026-05-22
date_published: unknown
topics:
  - agent-orchestration
---

# Fleet of Agents (sermakarevich/claude)

A tutorial directory in the `sermakarevich/claude` repo that builds up to running a fleet of parallel Claude Code agents in five self-contained steps. Each step adds one capability and is independently usable. The progression moves from a single manually-driven agent that persists work to disk, through an on-disk TODO queue, a bash loop for unattended execution, parallel agents with atomic task claiming via Gas Town's `beads` tool, and finally agents that can stop and ask questions when blocked.

The tutorial lives at `usage_pattern/fleet_of_agents/` with subdirectories for each step (`step_1_materialize_output` through `step_5_qa_blocking`) plus a README.md and article.md.

## Step 1 — Materialize Output

The agent writes task context to `.claude/tasks/<task_id>/` containing `TASK.md`, `PLAN_AND_PROGRESS.md`, and `FINDINGS.md`. A CLAUDE.md rule instructs the agent to create or resume from these files. The benefit: "you stop being the agent's memory."

## Step 2 — TODO/DONE Queue

Two files on disk: `.claude/tasks/TODO.md` (append bullets) and `.claude/tasks/DONE.md` (agent prepends summaries). The CLAUDE.md rule instructs the agent to take the top line from TODO.md, do the work, log completion, and remove the entry. This allows accumulating requests without waiting.

## Step 3 — loop.sh

A ~15-line bash script that repeatedly calls `claude -p "take next task from TODO.md per CLAUDE.md"` for each pending task. The key insight: "nothing in the agent changed" — the loop is a dumb re-invocation mechanism. This is the first point where you can walk away and return to completed work.

## Step 4 — Beads, and a Fleet

Replaces TODO.md/DONE.md with [`beads`](https://github.com/gastownhall/beads) (the `bd` CLI), a SQLite-backed task database with atomic claiming. Tasks move from `bd create` to `bd ready` to `bd update --claim`. Each loop exports a unique `BEADS_ACTOR` identifier so beads guarantees only one loop wins the claim. Beads also manages dependencies, preventing tasks with unfinished parents from being picked up. Multiple `./loop.sh` instances can run in parallel.

## Step 5 — Q&A Blocking

Since `claude -p` cannot use the `AskUserQuestion` tool, the agent writes questions to a per-task `Q&A.md` file (with `Context`, `Tried`, `Need` sections), then marks the task as blocked via `bd update --status blocked`. The blocked task disappears from `bd ready`, so "one blocked task doesn't stall the fleet." The human answers by appending an `## A:` block and flipping status back to `open`.

Critically: "loop.sh is byte-identical to step 4's" — no new flags or parsers. The blocking and resume mechanics rely entirely on beads controlling visibility and the agent writing questions, while "the loop stays dumb."

## Source Content

Full README.md and article.md text fetched from raw.githubusercontent.com.
