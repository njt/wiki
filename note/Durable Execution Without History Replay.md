# Durable Execution Without History Replay

A short position piece from Trigora proposing Transparent Continuation Checkpointing (TCC): instead of recovering a failed durable program by replaying its retained execution history, capture the program's live continuation at each durable boundary, commit it, and restore it directly after a failure. The pitch is not faster recovery in absolute terms but a different dependency: recovery cost should track the state the program still needs, not the length of its past. A controlled benchmark (fixed ~4 KB live state, boundary depth 10→1,000) shows TCC recovery flat at 0.6–0.9 ms while a Temporal replay baseline grows from 61 ms to 1.7 s, with the author unusually upfront about the narrow conditions and the long road to production.

---

## Key Quotes

> "Load retained history → re-execute the prefix → reconstruct the current position" / "Load committed continuation → restore live execution state → resume"

The two-line contrast is the entire proposal. Replay treats the past as the recovery path; TCC treats the present as the artifact. The event-sourcing world — and Temporal, and every journal-based workflow engine — chose replay because it is conceptually simple, language-agnostic, and auditable. TCC bets you can serialize a continuation portably instead, which is precisely where decades of systems research (continuation-passing, process migration, checkpoint/restart) have ground their gears.

> "A program that has performed ten thousand operations but retains a small live continuation should not necessarily become harder to recover simply because its past is long."

The sharpest sentence in the piece, and the one that makes it an *agent* argument rather than a workflow-engine argument. Long-running agents have exactly the shape where history grows monotonically — thousands of tool calls, waits on external events, child executions, direction changes — while the live state that actually matters stays bounded. Charging recovery for the past you no longer need is a historical accident of the replay design, not a law.

> "TCC does not make recovery constant-time. Recovery remains sensitive to the size and structure of the live continuation."

The honest hedge that makes the rest credible. The claim is about what recovery *depends on*, not that recovery is O(1). A continuation holding a large context window, open child executions, and pending waits is still expensive to snapshot — it just scales with the present, not the past.

> "Unsupported language constructs fail during compilation rather than producing ambiguous runtime behaviour."

This is where "transparent" gets its asterisk. Continuation capture is transparent only inside a language the compiler fully controls: no arbitrary FFI, no raw threads, no unconstrained native calls. That is the same bargain every deterministic workflow engine makes — restrict the programming model so recovery stays well-defined — except TCC moves the enforcement from runtime rules to compile-time failure. Arguably better ergonomics, identical philosophy.

> "These results ... demonstrate a difference in recovery scaling under the tested conditions, not that every TCC workload will outperform every replay-based system."

Benchmark honesty worth quoting. Worker creation excluded, ~4 KB live state, one replay baseline, and an explicit refusal to generalize. Most infrastructure posts bury these caveats; this one leads with them.

## Key Themes

#concept #pattern #tool

---

## Critical Analysis

**The idea is old, and the author knows it.** Serializable continuations as a recovery primitive are one of the graveyards of PL and systems research — Scheme-style continuations, process migration, checkpoint/restart systems — and the "What remains difficult" list (portable continuation representation, versioning, durable commit protocols, cross-language compatibility) is essentially that entire research program restated as a todo list. The post's virtue is that it does not pretend otherwise: it frames TCC as a prototype and invites criticism from workflow-engine, compiler, checkpointing, and distributed-runtime people. That is the right posture, and it is rare.

**The benchmark is the weakest part, and the honesty about it does not fully redeem it.** A 4 KB live state is the friendliest possible case for TCC — the whole argument is that recovery scales with live state, so a tiny live state guarantees a flattering result. And the Temporal comparison may be a soft target: production Temporal users manage long histories with "continue-as-new", which is exactly the manual escape hatch TCC automates. The measured phenomenon (restore vs. re-execute at growing depth) is real, but 1.7 s at depth 1,000 is only damning for workflows that never use the escape hatch.

**The agent win is real but narrower than it sounds.** Both replay and TCC refuse to re-run completed durable work; both still need idempotent, deduplicated effects. What TCC changes is the *control-state* reconstruction, not the effect-dedup problem. So the honest claim is: the workflow-engine bookkeeping around an agent run gets cheap to recover — not that agents themselves become crash-proof. The hard part of recovering a half-finished agent task was never reconstructing its loop position; it is knowing which external effects landed.

**The folk version of TCC already exists in agent harnesses.** [[Sidekick — Persistent Worker Agent Skill]] recovers workers from STATE.md checkpoints — checkpoint the live state, don't replay the transcript — which is TCC's philosophy implemented by hand at the harness level, with none of the compiler guarantees. [[State System]] makes the opposite bet (deterministic replay as an organizational primitive), and [[Magnitude]] builds an event-sourced runtime where restart replays the log or a snapshot plus suffix. TCC is the compiler-enforced end of a spectrum those pages already occupy informally.

**Verification is table stakes so far.** 50,000 generated semantic cases with no observed failures is the right kind of exercise for a runtime claiming crash correctness, but "the evaluated subset" is doing quiet work — generated tests probe the semantics the generator knows about. The tests that will actually judge TCC are production chaos: corrupt checkpoints, version-skewed binaries restoring old continuations, partially-committed durable effects. None of that exists yet.

**Net:** a genuinely interesting primitive with an unusually honest write-up. The open question is whether portable continuation capture is a research problem or an engineering slog — history says research problem, and until Trigora ships the versioning and migration story, the burden of proof stays with the prototype. But the target workload is real: the same one [[Cloud Agent Lessons from Cursor]] and [[All Your Agents Are Going Async]] describe from the operator's side.

## Related

- [[SQLite is All You Need for Durable Workflows]] — same target (durable workflows for AI agents), different axis: Obelisk asks where durable state should live and answers "a local SQLite file"; Trigora asks how recovery should work and answers "restore the continuation, don't replay." If Obelisk-style engines rebuild state from a journal, this source argues that reconstruction step itself is the wrong primitive.
- [[Cloud Agent Lessons from Cursor]] — Cursor's production bet is Temporal, the exact replay-based baseline this post benchmarks against; it strengthens the shared insight that long-running agents need durable execution infrastructure while complicating the assumption that Temporal-style replay is the right substrate for it.
- [[All Your Agents Are Going Async]] — Knill splits agent infrastructure into durable state and durable transport; this source strengthens the durable-state half with a third axis neither piece names: recovery cost must scale with live state, not accumulated history.
- [[Magnitude]] — Magnitude's event-sourced runtime is replay recovery done well ("restart replays the log or a snapshot + suffix"); this source complicates its thesis by proposing the committed continuation itself as the recovery artifact, which would make even snapshot-plus-suffix look like an intermediate point on the way to TCC.

---
*Sources: [[raw/durable-execution-without-history-replay]], [[summary/durable-execution-without-history-replay]]*
*Last updated: 2026-09-13*
