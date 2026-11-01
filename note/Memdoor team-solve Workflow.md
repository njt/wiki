# Memdoor team-solve Workflow

Memdoor's `team-solve` example rebuilds the GVS5H multi-agent method (Lee, 2026) as eight YAML files and zero code: a planner, two parallel ideators, a solver, a manager, and a repair worker, each with a fresh context, talking only through files on disk, with a deterministic script — not a model — deciding success. On ten LiveCodeBench Hard problems (5 passes, Qwen3.8-27B) it achieves parity with the paper's harness (9.8 vs 9.6 mean) and beats single-call Fable 5 (8.0), at a median of 40 minutes and $0.39 per run.

---

## What it is

- **Topology:** `plan ──┬── ideate-a ──┬── solve ── test ── review ── repair ── verify` — a DAG where `requires:` is the only coordination mechanism.
- **Two of seven steps are commands, not agents.** `test` runs the samples and never fails the run; `verify` runs the hidden tests and its nonzero exit does. The agent steps write proposals and instructions into `notes.md`; the machine owns the verdict.
- **No model named.** Steps only declare reads/writes; whatever `/model` is pinned answers, including a local model via `memdoor connect local` (Ollama, llama.cpp, vLLM).
- **Costs published:** $0.39 median per run, $35.79 for all 50 — all 50 runs' requests, replies, programs, grader output and cost are in the repo.

## Honesty as a feature

The README is unusually careful about what its numbers show: ten problems, not the authors' hundred ("9.8 and 9.6 are the same number"); possible training-data contamination for all columns; Fable 5 was not run in this harness; and the single lost problem is dissected — the manager invented a wrong counterexample and the team rewrote a correct program.

---

> "Every agent prompt says 'fresh context, read the files, do not rely on chat'. The paper credits that ledger discipline for the gain."

File-mediated state with fresh contexts per step is the whole design. Nothing is remembered that isn't written down; nothing is trusted that isn't re-read. This is memory-as-filesystem rather than memory-as-context-window.

> "The one lost problem: the manager invented a wrong counterexample and the team rewrote a correct program. Twice elsewhere a correct program was overwritten and then recovered. Both gaps are the authors' loop, kept as is."

The most interesting failure mode: a *reviewer* role hallucinating a counterexample that destroys a correct solution. Review steps are not infallible — they can be a source of regression, not just correction. That the workflow kept the authors' loop as-is, defect included, is proper parity methodology but means the harness inherits rather than fixes its weaknesses.

## Analysis

The striking thing is the economics: five small-model agents at $0.39 a run beat a frontier single-call at $~? per run. But the honest caveats matter more than the headline — ten problems cannot support a parity claim, and the authors themselves say so. What the page actually demonstrates is not a benchmark result but a *workflow format*: multi-agent orchestration expressed as a DAG of YAML files where coordination is `requires:`, state is files, and verification is a shell exit code.

This is a useful counterweight to heavyweight orchestration frameworks. The inter-agent communication protocol here is literally markdown files — `plan.md`, `notes.md`, proposals — which makes every handoff inspectable and the whole run replayable from the saved artifacts. Compare with JSON-message or shared-memory designs: files on disk are slower but free of schema debt, and they double as the audit log.

The weakness worth dwelling on is the review-repair loop's capacity to damage: `review` reads test output and writes repair instructions, and a hallucinated counterexample cascades into overwriting a correct solution. The system has no notion of "don't regress" beyond the final verify gate. A cheap addition would be re-running verify on the pre-repair program.

**Tags:** #tool #pattern #concept

## Related pages

This source strengthens [[Dynamic Workflows in Claude Code]]'s claim that orchestrated multi-agent topologies beat single-context windows, but nuances it: here the harness is static YAML rather than agent-authored JavaScript, and the verification gate is a script exit code rather than a model judge. It complicates [[Decompiling MW2 with Four Claude Agents]]' picture of long-running multi-agent work — team-solve is the minimal, 40-minute, single-problem end of the spectrum, showing that fresh-context-plus-files discipline can work without compaction machinery at all. It reinforces [[Poor Man's Loop Engineering]]'s thesis that the minimum viable loop is self-testing plus independent review — with the twist that the reviewer can itself inject errors ([[Silently Resolved Ambiguity Is Comprehension Debt of Intent]]'s decision-hygiene problem in miniature).

---
*Sources: [[raw/team-solve]], [[summary/team-solve]]*
*Last updated: 2026-11-02*
