---
url: https://www.npmjs.com/package/pi-multiloop
title: "pi-multiloop"
author: lhl
date_fetched: 2026-09-22
date_published: unknown (npm page shows v0.4.0, no publish date; repo paths reference May 2026)
topics:
  - coding-agents-and-frameworks
  - guardrails-and-feedback-loops
---

# pi-multiloop

`pi-multiloop` (v0.4.0, MIT, by lhl) is an autoloop/autoresearch extension for the Pi coding agent. Its distinguishing feature is **multi-lane isolation**: several loops can run in the same worktree, each with its own independent state directory, so tuning a CUDA kernel and sweeping quantization parameters at the same time does not require extra worktrees or branches. The implicit critique of prior loop tools is that they conflate state isolation with filesystem isolation — when the experiments touch different files, what needs separating is the loop state, not the tree.

It offers two entry points. `/goal <objective>` is the zero-setup path: no metric, no verify command, no questions — the lane, mode, and scope are derived from the objective's wording and work starts immediately; the run finishes when the agent's own completion audit passes, not when a number crosses a threshold. `/multiloop <goal>` is the measured path: a guided setup scans the repo, proposes the whole configuration (metric, verify command, guards) in one message, and starts the loop on a single approval. Both create ordinary runs under `.multiloop/`, shareable commands (`status`, `resume`, `pause`, `stop`, `archive`).

Four loop modes cover the space: **Optimize** (the classic edit → measure → keep/revert cycle), **Research** (log ablations and sweeps without keep/revert), **Dev** (implement, test, commit with iteration tracking), and **Punchlist** (parse a markdown checklist and work through open items). All state lives in a single `.multiloop/` directory: a `registry.json` index, per-lane per-run directories holding an atomic `state.json` resume snapshot, an append-only `results.jsonl` iteration log (never overwritten, human-readable and diff-friendly), and an optional `lessons.md` where strategy pivots are recorded and carried forward to bias future hypotheses.

The most distinctive mechanism is the **compound verifier**. A measurement can carry mechanical checks (`npm test`) and prompt-based correctness checks alongside the metric; keep is recommended only when the metric improves *and* every check passes. If a loop was configured with a guard or prompt verifier and the agent omits the corresponding verdict, the missing verifier is recorded as a failed check, and `multiloop_decide` rejects mismatched decisions — "a faster-but-incorrect output is mechanically forced to `revert` unless the agent reruns verification and records a passing result." Noisy benchmarks are handled with Median Absolute Deviation confidence scoring.

The package is also a study in agent-facing telemetry discipline. Every run records elapsed time, turns, tool calls, and token totals, reported in status views and on pause/complete — but the counters "never reach the agent," because a running total delivered every turn "reads to a model like a context gauge — enough to make one wind down work that was not finished." Loop-owned turns auto-continue by queueing the next required action (a pending measurement forces decide/log before new work), and when Pi auto-compacts mid-loop, pi-multiloop injects a loop-aware resume prompt grounded in `.multiloop/` state after the interrupted turn. It does not auto-attach persisted loops in new sessions — only a passive "available to resume" notice. It composes with other Pi extensions (pi-boomerang for context compression, pi-supervisor for goal enforcement, pi-review-loop for quality gates), and sits in a lineage descending from karpathy/autoresearch through codex-autoresearch forks and a wider ecosystem of Pi loop extensions.

---
*Sources: [[raw/pi-multiloop]]*
*Last updated: 2026-09-22*
