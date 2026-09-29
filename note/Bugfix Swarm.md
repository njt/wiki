# Bugfix Swarm

bugfix-swarm is a Pi package that runs a human-gated, read-only static-analysis bug hunt as a nine-phase funnel: a short human kickoff, waves of hunter and adversarial-verifier subagents that each write one report file and return only an envelope, a dedicated dedupe agent that merges everything into one findings report, a human picker per repair bucket, and executor agents that fix accepted rows and verify by live execution. It matters because it is one of the most fully worked-out documents of multi-agent *process* engineering in the wild — ~3,100 lines of doctrine distilled from real runs (a 90k-LOC census: 28 hunters, 21 verifiers, 133 findings), with every rule annotated by the failure that produced it.

---

## Architecture

The code footprint is tiny; the doctrine is the system.

- `extensions/index.ts` (22 lines) — the single Pi extension: registers the seven-skill corpus under `skills/` via a `resources_discover` hook and installs the `checkbox_picker` tool from the npm dependency `@taylursatula/pi-checkbox-picker`. Nothing injects content; "USER-GATED" doctrine travels in the skill descriptions themselves.
- `skills/bugfix-swarm/SKILL.md` (799 lines) — the runbook: bootstrap → decompose → hunt → verify → dedupe → slice → decide → fix → close, with hard gates between phases and a pacing contract ("never idle, never poll", two channels to the human never mixed in one turn).
- `skills/subagent-orchestration/SKILL.md` (285 lines) — pass design: the blind/primed/adjudicator/tiebreak/recheck taxonomy, the preamble template, dispatch and dispute mechanics.
- `skills/bugfix-swarm-handoff/SKILL.md` (151 lines) — compaction continuity: the five volatile facts, FLUSH blocks after every decision point, the ordered resume gate.
- `skills/writing-probes/` (460 lines) and `skills/plan-investigation/` (241 lines) — live-verification doctrine and fix-wave claim-checking.
- `agents/*.md` — four global subagent roles with frontmatter-pinned models and thinking budgets: `swarm-investigator` (glm-5.3, thinking high), `swarm-verifier` (deepseek-4.1-flash, medium), `swarm-dedupe` (flash, low), `swarm-executor` (glm-5.3, medium). All use `prompt_mode: replace` and skills off.
- `examples/census-hunt.example.js` (461 lines) — a filled `SubagentWorkflow` script from a real census: `pipeline(areas, hunt, verify)` with no barrier, schema-validated envelopes, fail-loud preamble sanity checks, `gate: '<command>'` on fix agents.

Topology: a strict hierarchy with a flat orchestration core. The orchestrator holds the map, the ledger, and the standard; agents are interchangeable pass-executors; the dedupe agent is a dedicated context so merging 28 reports never enters the orchestrator's window. Context tiers are explicit — each layer reads, writes, and hands up exactly one thing (envelope, one-line report path, briefs and pickers).

## Key techniques

- **Envelope by schema, payload by file.** Every pass writes `scratch/<run>/reports/<pass-id>.md` and returns only counts (findings by tier, areas clean, disputes). Report paths derive from pass IDs, so the dedupe agent can enumerate the corpus from paths alone and the orchestrator never reads raw evidence. This is what keeps orchestrator context flat as agent count grows.
- **Information states as an independence mechanism.** "Independence is an information-design property, not an agent property": a primed verifier checks faithfully *inside* the claim's frame and inherits its arithmetic slips; only a blind pass can catch a wrong framing. The convergence bar — two passes agreeing, at least one frame-independent — gates every finding before a human sees it. A seam finding is frame-disjoint from an area finding at its endpoints, so cross-channel agreement satisfies the bar directly.
- **Enumeration closes classes; reading closes sites.** One composed shell command (`rg`/`ast-grep`/`awk`) answers "is the hazard class closed" and publishes its own bound; agents handed a pattern return pattern-bounded reports. Corollary doctrine: "a zero from a structural search is not a clean result" — confirm the rule matches a site you know exists before believing a zero.
- **The registry as externalized state.** Conclusions live in the kata tracker, written at the moment of decision, never batched: one ticket per finding with typed evidence (`commit:<sha>`, `test:<cmd>`, `reviewed-paths:<path>`), a run-level ticket holding the coverage summary and decay reading, and a permanent, monotone refuted set that later bootstrap passes read as the do-not-re-chase list. Verified-clean surfaces are kept separate from refuted claims — both make "clean" auditable, only the first is a no-re-chase entry.
- **Pinned severity scale in a shared preamble file.** Without one, 28 agents produce 28 scales and dedupe compares noise. A human correction edits the preamble file once and rides in every later prompt — agents carry nothing between invocations.
- **Fail-loud workflow scripts.** Bake correctness-critical text (the preamble) into the script as literals after an args-propagation failure silently handed every child the string `undefined`; throw at startup if sanity strings don't byte-match; spot-check child session files for prompt delivery before trusting a running wave.
- **Honesty-forcing output contracts.** Null-result permission plus mandatory self-refutation as the anti-fabrication pair; "frequency is demonstrated, not asserted" — an ungroundable frequency is latent by definition; "checked and clean" sections required so coverage is auditable; verifiers return corrections, not verdicts ("a report of bare CONFIRMEDs added a sample, not information").

## Design decisions

- **Read/write as a privilege boundary, not a phase.** Investigation agents change nothing; a human decision moves a site from read to write — never an agent's judgment, never the orchestrator's. Write authority is bounded by directive rather than allowlist ("you may write exactly one file"), and the verifier is explicitly forbidden from fixing what it judges, to protect verdict independence.
- **Human attention, not tokens, is the scarce resource.** Runs are overnight on unmetered endpoints; the pacing contract pipelines the human's attention (repairs dispatch the moment a picker returns, the next picker opens while they run) and forbids two pickers in flight. This inverts the usual cost-optimization frame entirely.
- **Ambiguity never settles in-agent.** Genuine ambiguity is reported and moved on; reasoning that stays in a transcript dies with it. Disputes are settled by identifying the decisive artifact, not by asking who is right — and a dispute reducible to arithmetic or a 30-line read is the orchestrator's own read, not a delegation.
- **The skill is USER-GATED by construction.** The description forbids autonomous loading even when a task would benefit; the human opens every run. Combined with "agents never recommend fixes" and commit-only-when-asked, the whole funnel is built so no agent decision reaches the tree unratified.
- **Vocabulary is treated as an input.** The runbook notes that "hunt, swarm, hunter, kill" elicits an aggressive frame that shapes what gets reported, and treats the anti-noise rules as the deliberate counterweight — prompt engineering applied to its own prompt engineering.
- **Weaknesses.** The doctrine is self-admittedly tuned to one harness (Pi, `SubagentWorkflow`, Lunaroute routes, kata) and one operator's calibration history; portability is a manual edit. And it is expensive in the dimension it optimizes for: the human decides every row across every bucket, which is only survivable because runs are rare.

## Comparison notes

- [[Swarm Skill]] shares the multi-agent bug-hunt shape (hard rules from failures, adversarial verification) but optimizes Claude Code sessions; bugfix-swarm goes much further on the funnel's human-decision plumbing and on compaction continuity across multi-window runs.
- [[Agent Swarm Model Economics]] (Cursor) scales planner/worker trees to thousands of agents and optimizes token economics; bugfix-swarm runs 30–50 agents a few times a quarter and optimizes the ledger's correctness and the human's attention — near-opposite points on the same orchestration spectrum.
- [[Pi Subagents]] covers the Pi subagent mechanism bugfix-swarm is built on; this repo is the largest worked application of it, including the workaround that Pi packages can't register subagent definitions (agents ship as files to copy into `~/.pi/agent/agents/`).
- [[Kata]] is the registry this runbook is written against — one ticket per finding, typed evidence to close, refuted set as a monotone do-not-re-chase list — a concrete instantiation of kata's agent-ergonomics thesis.
- [[Structural Backpressure Beats Smarter Agents]] argues for deterministic verification over more intelligence; bugfix-swarm's `gate:` command on fix agents and EXECUTED-or-UNVERIFIED contract are that thesis applied to repair verification, while its verify layer shows what backpressure looks like when the check itself must be an agent.

---
*Sources: [[raw/bugfix-swarm]], [[summary/bugfix-swarm]]*
*Last updated: 2026-09-29*
