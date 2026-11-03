# Bondsmith — Hard Bonds for Soft Claims

A workflow engine (Rust, pre-open-source) built around one rule: no claim an LLM makes about its own work counts without a deterministic proof bonded to it. Workers do the code, Gates check it, a Driver state machine is the only writer of state, and every step's Postconditions are evaluated in plain Rust against facts gathered from disk, git, and Gate results.

---

The author opens with the sentence every agent-session summary ends with — "All tests pass" — and notes that more than once it was false: not run, run before the last edit, or never written. Bondsmith is the systematic answer: the LLM stays free inside a step, but everything it says afterwards is a soft claim that needs a hard bond.

The architecture enforces a strict division of labour:

- **Worker**: a disposable headless LLM session for exactly one step. It edits files and commits — that's all. "Nobody asks it whether it passed or failed."
- **Gate**: a deterministic check (lint, type-check, tests) whose verdict comes from the tool, not the model. Repos configure their own gate pool.
- **Driver**: a state machine that spawns Workers, runs Gates, and is the only thing that writes state.

Postconditions use a closed verb vocabulary — `exit_zero`, `sha_is_ancestor`, `non_empty: spec.md`, `gate_pool_pass` — and fail closed: a missing fact is a failure. A step either passes or hard-blocks; there is nothing in between.

Flows are plain data (JSON contracts + Markdown prompts on disk), nothing is compiled in, and no Flow is built in. Harness independence is architectural: contracts and prompts carry no harness-specific text; Workers launch through one port (Claude Code today, Pi next). Observability is total by construction — state file, append-only event log, per-Worker session transcripts — with a read-only UI over the same files.

The worked example is the best part: a test-first implement step where four of five Postconditions held (the red-test commit exists, is an ancestor of HEAD, the suite grew 1708 → 1711, the red test was re-run at that SHA) — and the step was still held because one test still failed. That is what fail-closed verification actually looks like when it bites.

Beyond gates, the review pipeline produces structured Findings, and Triage rules (per-repo, first-match-wins) decide which get auto-fixed — with the Driver re-deriving the Worker's triage decisions from the rules in code, and a floor no rule can cross: security Findings and anything above medium severity always go to the human. A learn-from-reports Phase mines repeated rulings into proposed Triage rules (as PRs) and Knowledge-base entries.

---

## Key quotes

> "Whatever it tells me about that code afterwards, I check with something which is not an LLM."

The whole thesis in one sentence. Verification must be a different epistemic system from generation — the same separation of church and state as determinism-vs-model.

> "There is no arrow from the Worker to state."

The sharpest architectural claim in the piece. The agent's authority ends at the filesystem; the only path into the record of truth runs through deterministic code. Most "agent harnesses" quietly violate this by letting the model narrate its own progress.

> "So four checks out of five hold, and it is still not enough. The step is held until the failing test passes."

An unusually honest worked example — not a success story, but a success story *of the failure detection*. The 1708 → 1711 test count is a nice touch: even "tests were added" is proven numerically, not asserted.

> "A rule may never decide a security Finding, or anything above medium severity. Those always come to me."

The learn-from-reports loop is interesting precisely because it's bounded: automation may absorb the rulings the human keeps repeating, but the severity floor is a structural guarantee, not a prompt.

> "Pick one action your agent reports on, and replace the report with a check a program runs."

The takeaway deliberately survives without the tool: one check is enough to feel the difference. Good practical advice and good marketing in one.

## Themes

#concept — postconditions, fail-closed verification, closed verb vocabularies
#pattern — Worker/Gate/Driver separation; state written only by deterministic code
#tool — Bondsmith itself, harness-agnostic launcher port

## Analysis

This is the most concrete engineering realisation yet of a thesis the wiki keeps converging on from different directions: the agent is a great generator and an untrustworthy witness. Where [[Agentic Verification]] argues for cheap checking loops and [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] gives a tool-choice heuristic, Bondsmith is the build — a typed contract system where "the tests were run" is literally `exit_zero` evaluated by the Driver.

What's most valuable is the discipline of what the Worker *cannot* do. Most agent frameworks treat the model's self-report as a convenience; Bondsmith treats it as untrusted input, and even the Worker's *triage decisions* get re-derived from rules in code before being accepted. That's a stronger stance than most harnesses take, and the cost is honest: five gates on a nine-step task, plus the engineering of a Rust workspace and a UI around them.

Two caveats. First, the fail-closed posture plus a closed Postcondition vocabulary will bump against the long tail of "facts worth proving" — the piece doesn't discuss what happens when the proof you need isn't expressible as a verb. Second, the human still rules on every non-trivial Finding; the Triage-rule learning loop is the interesting bet, but it's only "fully there" as a mechanism, not yet proven as a flywheel. Still, as a pre-open-source design essay it's unusually specific: real numbers, real failed runs, a severity floor.

## Relations

- [[Agentic Verification]] — Osmani argues agents make checking cheap; Bondsmith is the fully-committed version, where the check is not merely cheap but structurally mandatory, run by a Driver that cannot be talked out of it.
- [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] — this source strengthens the deterministic-first tier with a concrete implementation: gates, exit codes and severity floors as the load-bearing layer, models confined to generation.
- [[Deterministic Spine, Agentic Leaves]] — Bondsmith is essentially that topology given a type system: the Spine is the Driver's Postcondition evaluation, the Leaves are Workers free inside one step.
- [[Aviator Verify]] — a commercial product with the same intent-verification intuition (check what was agreed, with evidence); Bondsmith differs by grounding every claim in exit codes and git SHAs rather than a captured criteria list.

---
*Sources: [[raw/bondsmith-hard-bonds-for-soft-claims]], [[summary/bondsmith-hard-bonds-for-soft-claims]]*
*Last updated: 2026-11-03*
