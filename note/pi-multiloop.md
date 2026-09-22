# pi-multiloop

An npm-page README for pi-multiloop v0.4.0, an autoloop/autoresearch extension for the Pi coding agent by lhl. Its claim to novelty is **multi-lane isolation** — several loops running in one worktree, each with independent state under `.multiloop/` — plus a zero-setup `/goal` command for objectives that have no metric. Under the hood it is a careful piece of loop infrastructure: four modes (Optimize/Research/Dev/Punchlist), compound verifiers that force keep decisions to survive both a metric improvement and mechanical/prompt checks, append-only JSONL history, and compaction-aware resume.

---

## Key Quotes

> "Other loop extensions only support one loop per session or worktree. … pi-multiloop lets each loop have its own lane with independent state, so you don't need extra worktrees or branches."

The founding observation is that prior tools conflate two different kinds of isolation. Two experiments touching different files in one repo do not need separate filesystems — they need separate *loop state*: which iteration, which baseline, which results log. Lanes give you the second without paying for the first. This is the same realization that [[What Ralph Wiggum Loops Are Missing]] arrives at from the other direction: once more than one loop runs, freeform shared state is what breaks first.

> "a faster-but-incorrect output is mechanically forced to `revert` unless the agent reruns verification and records a passing result."

The compound verifier is the strongest idea in the package. Acceptance is not a suggestion the agent is asked to honor; `multiloop_decide` rejects decisions that contradict the recorded checks, and a verifier the agent was configured to run but silently skipped is recorded as a *failed* check. This is [[LLM-as-a-Verifier]]'s independence principle implemented as a state-machine rule rather than a prompt exhortation. The residual loophole is honest, though: "unless the agent reruns verification and records a passing result" — rerunning a flaky check until green is the classic Goodhart path, and the design leans on the agent not grinding it.

> "These counters never reach the agent. … a running total delivered on every turn reads to a model like a context gauge — enough to make one wind down work that was not finished."

The single sharpest sentence in the doc. Work accounting is for the human; showing it to the model would *change the model's behavior*, making it wind down unfinished work. That is unusual sophistication about agent UX: telemetry and context-pressure signals are different instruments, and conflating them corrupts the loop. It pairs directly with [[Self-Generated Prompt Injections in Compaction Summaries]] — both are about what the harness chooses to surface into the agent's context around state transitions, and how that choice steers behavior.

> "Compaction-aware resume — when pi auto-compacts during a loop … pi-multiloop injects a loop-aware resume prompt after the interrupted turn ends."

Long autonomous loops do not die of context exhaustion; they die of amnesia at the compaction boundary. Reconstructing in-memory state from `results.jsonl` + `state.json` and injecting a state-grounded resume prompt is the harness taking responsibility for the one moment the model is guaranteed to be disoriented.

## Themes

#tool #pattern #concept

## Analysis

The interesting engineering here is almost entirely about **state, not prompting**. An append-only `results.jsonl` that is "never overwritten," an atomically-replaced `state.json` snapshot, a registry, an archive with timestamp prefixes — the loop logic itself is the boring classic (edit, measure, keep/revert), and the package's value is that the state survives restarts, context resets, and compaction. Compare [[Ralph]]'s freeform markdown tracking: pi-multiloop is what that pattern looks like after it has been forced to grow durable, inspectable state because people ran more than one loop at a time and needed to resume yesterday's run.

The two-tier UX is a quiet admission about where the real cost lives. `/multiloop` needs a metric, and defining one is the expensive part — so the guided setup scans the repo, proposes everything at once, and asks a clarification round only when no metric-producing command exists, several plausible sources compete, or a command looks unsafe. `/goal` punts entirely: no metric, no verify command, termination by "completion audit." That trades rigor for friction, and the README is refreshingly honest that such runs finish "when the agent's completion audit passes, not when a number crosses a threshold" — i.e., the cheap tier trusts the agent's self-assessment, which is exactly the "grading its own homework" weakness [[Loop Engineering]] warns about. Offering both, labeled, is the right call; users should know which tier they are in.

Weakest point: the README is a spec, not an evaluation. There are no benchmarks, no traces, no claim about how often compound verifiers actually catch bad keeps in practice — the compound-verifier JSON example is illustrative. And multi-lane-in-one-worktree assumes the "experiments touch different files" premise; two loops that *do* collide on the same files get lane isolation of state but no filesystem protection, which the doc does not address. As lineage it is clear-eyed, though: it names karpathy/autoresearch as "the original" that "established the pattern" and positions itself as the Pi-native equivalent of the codex-autoresearch fork. In the Pi ecosystem it slots next to [[Oh My Pi (omp)]] (harness depth) and pi-boomerang/pi-supervisor/pi-review-loop, with which it explicitly composes.

## Relations

- [[Loop Engineering]] — strengthens: Osmani's taxonomy describes designing harnesses that prompt agents; pi-multiloop is a concrete, stateful implementation of that thesis, down to mechanical continuation and escalation after consecutive failures.
- [[Performance Optimization Loop]] — nuances: Gordon's human-driven monitor → baseline → change → re-benchmark discipline is the same shape as the Optimize mode, automated; MAD confidence scoring addresses the noisy-benchmark problem Gordon handles by careful manual methodology.
- [[What Ralph Wiggum Loops Are Missing]] — strengthens and extends: that piece argues loops need structured state before they scale past one agent; pi-multiloop's lanes, registry, and JSONL history are a working answer for multiple loops in one tree.
- [[Self-Generated Prompt Injections in Compaction Summaries]] — complicates usefully: pi-multiloop deliberately injects a curated, state-grounded prompt into post-compaction context, the mirror image of that piece's worry about uncontrolled compaction content steering the agent.

---
*Sources: [[raw/pi-multiloop]], [[summary/pi-multiloop]]*
*Last updated: 2026-09-22*
