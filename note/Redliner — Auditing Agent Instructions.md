# Redliner — Auditing Agent Instructions

Matt Galligan's `redliner` skill (`galligan/skills`, v0.1.0) audits the instruction layer — `AGENTS.md`, `CLAUDE.md`, `SKILL.md` files, `.cursorrules`, scoped rules, tool descriptions — for conflicts, unnecessary work, and unintended consequences, and returns an evidence-backed redline: ranked findings, exact quotations, proposed diffs, one consolidated JSON. It never executes or edits what it audits. This page ingests the *full* fetch of a skill the wiki first saw on 2026-09-13 as [[Minority Report (Instruction Audit Skill)]], since renamed upstream — and the newly visible material (a writing style guide, two instruction-design guides, a calibrated review rubric, and the entire Python pipeline with tests) moves the center of gravity from "a skill with discipline" to "a doctrine of instruction engineering with a tool attached."

---

## What Changed Since the Minority Report Ingest

**The name.** `minority-report` → `redliner`. The old note read the Philip K. Dick reference as the pitch: precogs for instruction failures, predicting what instructions *will* do before the failure teaches you. "Redliner" is the duller and more accurate word — a redline is a marked-up draft someone else must accept or reject, which is precisely the propose-never-apply contract. The rename trades a metaphor for a job description, and the discipline language shifted with it: "A clean report is valid; do not manufacture dissent" became "A clean review is valid; do not manufacture criticism." Same principle, less poetry.

**The fetch.** The 2026-09-13 ingest saw the SKILL.md plus a file listing; this one contains everything — ten reference guides, two JSON schemas, seven scripts, three test files. The earlier note's "deterministic floor" thesis can now be checked against source, and it holds: `diffcheck.py` validates hunk positions, counts, and source context entirely in memory ("Check proposed unified diffs in memory; never write audited files" is the module docstring, not an aspiration), and `consolidate.py` refuses to write output over any source, map, review, or adjudication file.

**The version.** Still 0.1.0, through a rename and a visible revision. For a tool whose whole business is instruction hygiene — versioned artifacts, provenance, review dates — its own identity management is a small irony worth recording.

## How the Pipeline Actually Works

`discover.py` walks a bounded scope for entry files (`AGENTS.md`, `CLAUDE.md`, `SKILL.md`, `GEMINI.md`, `.cursorrules`, `.windsurfrules`, Copilot and Cursor rule files), follows markdown links, Obsidian links, `@` imports, and backtick paths, and records SHA-256 fingerprints plus candidate signals — regex hits for broad triggers, universal reads, required checks, and confirmation gates. Those signals are demoted by design: "Candidate passages are leads, not findings; inspect full relevant sections and account for files without keyword matches." Managed content (skills.sh installations, installed Claude/ChatGPT/Codex plugins) is excluded with reasons preserved in the map, and GitHub remote-affiliation checks are advisory only — "unknown provenance is not proof of authorship."

Reviewers then apply the bundled rubric against `review.schema.json`; substantial scopes fan out to subagents with explicit file assignments and separately owned JSON outputs, and a coordinator adjudicates through a recorded file, so rejected candidates survive with reasons rather than vanishing. Consolidation enforces the floor — schema, fingerprints, exact quotations, diff applicability, full mapped-file coverage — and `findings.json` is the mandatory deliverable; the markdown report is decoration that may never replace or truncate it.

## Key Quotes

> "Agent-facing instructions are closer to configuration than prose. Their goal is not elegance; it is to communicate behavior with the smallest practical amount of ambiguity."

The thesis of Agentish, the reference guide that is this fetch's real prize. It adapts ASD-STE100 Simplified Technical English — aerospace documentation's controlled language — to agent-facing text: normative vocabulary matched to audience, one canonical term per concept, conditions before actions, atomic procedural steps, and a stated safe path for every prohibition. It is the most rigorous public treatment of steering-file prose style this wiki has recorded.

> "Words that describe quality are not decision criteria."

Agentish's banned-qualifier list reads like an autopsy of real CLAUDE.md files: *appropriate, clean, correct, efficient, excessive, important, maintainable, meaningful, minimal, necessary, obvious, proper, reasonable, relevant, significant, simple, substantial*. These are the words human code review runs on, and the guide demotes every one of them when it controls agent behavior — the replacement is always an observable condition. "Add tests for significant behavior changes" becomes "Add or update tests when the change modifies observable behavior."

> "Write consequential instructions so compliance is testable."

Agentish's "Evalability" section is the bridge between style and verification: a strong rule exposes condition, actor, action, target, exception, and expected outcome, such that you can construct a triggering case, a safe counterexample, and an unrelated case. Contrasting-pair examples are "natural seeds for behavioral eval cases." And the guide polices its own scope: "Language conformance is one layer, not the result. A sentence can follow this guide perfectly and encode a bad policy."

> "A rule with no evidence is **preventive/unproven** — record that status rather than treating it as validated."

Instruction Selection's question six — "Would removal cause a repeatable or high-consequence failure?" — is the strongest keep signal, and this clause is its honesty provision. Instructions justified by imagined failures get a status, not a pass: the rule stays flagged as unevidenced, which is exactly how instruction rot becomes visible before it becomes debt.

> "Asking the model whether it needs a rule is not a behavioral test."

`model-review.md`'s protocol for contested removals: compare original and proposed instruction on a triggering case, a safe counterexample, and an unrelated task, on the target model and harness; isolate the change so a regression can be attributed; and if target-model execution is unavailable, "report that limitation and do not claim measured improvement." Age, an imperative, or a model name "is a lead, not evidence that the instruction is obsolete."

## Key Themes

### #concept — The instruction estate as governed configuration

The skill's unseen premise is that steering files have become an *estate* — dozens of bodies across root files, scoped rules, skills, and tool descriptions, accreting for years, read by machines. Redliner imports code's review machinery wholesale: discovery, fingerprints, coverage accounting, adjudication, provenance. Agentish completes the frame — if instructions are configuration, they get configuration's standards: consistency over variety, observability over elegance.

### #pattern — Evalability: write rules their evals can prove

The demand that every consequential instruction expose a triggering case, a safe counterexample, and an unrelated case turns instruction-writing into eval authoring. This is the same move [[Benchmarking AGENTS.md Changes]] argues for empirically — instruction edits are behavior changes and need validation — here pushed upstream into the sentence structure itself.

### #pattern — An evidence floor with declared limits

The scripts enforce what is mechanically checkable and refuse to fake the rest: "A passing check verifies structure and evidence, not the reviewer's judgment." Provenance and ownership stay advisory; coverage proves files were accounted for, "not that judgment was correct." The honesty about what the floor does not verify is the design's most transferable property.

### #tool — Cross-harness by construction

Entry points include `GEMINI.md` and `.cursorrules`; exclusions cover ChatGPT and Codex plugin stores; and the skill ships an `agents/openai.yaml` interface manifest alongside its Claude-format SKILL.md. The doctrine is deliberately provider-agnostic — the Astra and Fable profiles are explicitly "named review targets, not permanent claims about which models are latest."

### #person — Matt Galligan

Two halves of one thesis: [[The Grid — Agent Identity Architecture]] tells agents *who to be*; Redliner audits *what agents are told*. Both treat the instruction layer as engineered configuration deserving production-grade engineering. The rename from Minority Report to Redliner reads as the author applying his own doctrine to his own artifact: drop the flourish, keep the contract.

## Critical Analysis

**The doctrine is now the product.** In the earlier, thinner fetch, the SKILL.md's review discipline was the story. With everything visible, the references outweigh the orchestrator: Agentish, Instruction Selection, and Instruction Placement are independently useful documents that will outlive the tooling around them. That is a strength and a tell — the skill is a delivery vehicle for a position paper on instruction engineering, and the pipeline exists to make the position enforceable.

**The styleguide/review tension is managed, not dissolved.** A rubric that routes wording checks through Agentish risks collapsing into style enforcement — every finding becomes "your instructions don't conform to Galligan's guide." The rubric guards against exactly this ("Style nonconformance alone is not a finding; identify the concrete behavioral consequence"), and the calibration examples repeatedly refuse flaggable-seeming patterns. The guard is right, but the pull is real and worth watching in any actual audit output.

**Still no sample audit.** The package demonstrates machinery, not results — no completed run, no corpus of found conflicts, no evidence the multi-reviewer adjudication beats one careful pass. That was the earlier note's open question and it stays open; the fuller fetch sharpens it, because now the machinery is known to be real, making the missing evidence the only missing thing.

## Related

[[Minority Report (Instruction Audit Skill)]] is this same project at its previous name and shallower fetch. The full source strengthens that note's deterministic-floor thesis with visible implementation — `diffcheck.py`'s in-memory validation and `provenance.py`'s reasoned exclusions are now checkable facts — and complicates its framing: the "dissenting voice against the accumulated majority" reading was the Minority Report metaphor doing work that the rename to Redliner quietly retires.

[[Writing a Good CLAUDE.md]] argues the instruction budget is ~150-200 followable directives and every line degrades the rest. Redliner's Instruction Selection operationalizes that budget as review criteria — "Can software enforce it?", "Does the model already know it?", unevidenced rules recorded as unproven — turning a sizing argument into an audit procedure.

[[Benchmarking AGENTS.md Changes]] shows instruction edits regressing on holdout tasks (AGENTS.md inversion) and demands empirical validation. Redliner's `model-review.md` mandates the same discipline for contested removals — triggering case, safe counterexample, unrelated task, isolated change, no claimed improvement without execution — as a review contract rather than a one-off experiment.

[[Steering Claude Code]] taxonomizes the seven channels that deliver instructions to Claude Code. Redliner's Instruction Placement is its review-grade sibling: a surface table of what each channel owns and refuses, plus the load-verification rule ("Moving a paragraph to a different file reduces persistent context only if the harness defers loading that file") and the claim that skill descriptions are compiled policy because harnesses route on them before bodies load.

---
*Sources: [[raw/redliner]], [[summary/redliner]]*
*Last updated: 2026-09-22*
