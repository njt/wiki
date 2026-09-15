# Agent Designer

A Pi skill that designs and deploys expert ensembles — casts of domain-expert personas run through a fixed, versioned orchestration methodology. Its real subject is not personas but **process as artifact**: the project treats a multi-agent analysis methodology the way software treats a library — semver-versioned, inherited whole, changed only by reviewed release — and ships its own refutation loop (a self-critique session whose findings became the v1.2.0 calibration). The repo is unusual among agent projects: the implementation is almost entirely markdown doctrine plus three small scripts, and the honesty constraints on evaluation (traceable contribution vs. causal lift) are more careful than what most agent frameworks bother to state.

#tool #project #agents #multi-agent #prompting

---

## What it is

`dbmcco/agent-designer` (v0.3.0, MIT) is a **skill bundle** for the Pi coding agent: the repo root is the skill, discovered by symlinking `~/.agents/skills/agent-designer` to it. `SKILL.md` is the dispatcher — its first and mandatory step is purpose triage between two protocols:

- **Panel protocol** — a standing analysis panel (moderator + 5–8 specialists + exactly one adversarial/audit seat) created inside the repo where the work happens. Deep, repeatable, re-entered over time.
- **Runtime protocol** — 2–5 experts cast live for one question, executed through Pi's subagent tool in a single `workflowScript` call, transcripts left on disk, cast promotable into a panel later.

Both inherit the same persona doctrine, roster heuristics, and methodology kernel. The provenance is eight hand-built panels in the author's `experiments/ai-simulations` (Shell scenario panel, terrain, root-cause, vc, pmf, and others); the skill canonizes their shared machinery and treats them as read-only reference.

## Architecture

The structure is **doctrine files + thin deterministic scripts + a bash test suite**. No runtime of its own — the host agent *is* the runtime, and the skill is methodology loaded into context.

- `SKILL.md` — triage, protocol steps, operating rules (locality, promotion path, calibration gate).
- `doctrine/persona.md` — the eight mandatory persona sections (name, title/signature, CV in the field's language, formative experience, proclivities, blind spots, communication texture, query formulation) and the two-pass completeness gate.
- `methodologies/kernel.md` — 11 orchestration rules, versioned 1.2.0, inherited whole by every ensemble.
- `methodologies/overlays/` — scenario-planning (nine phases from worldview elicitation to strategy testing), terrain-mapping, root-cause, and the pilot `assignment-decomposition` (v0.1.0, opt-in).
- `reference/roster-heuristics.md` — casting: sizing ranges, the single adversarial seat, eight coverage axes, "conventions are defaults, not laws".
- `reference/contribution-review.md` — the post-run evaluation protocol.
- `scripts/` — `validate-persona.sh` (structure), `validate-synthesis.sh` (dissent check), `diversity.py` (divergence smoke detector), `verify-bundle.sh` (SKILL.md references resolve, kernel version pinned everywhere, full suite green).
- `tests/` — bash structural tests with pass/fail fixtures, including a full example persona (`good-halvorsen.md`, Dr. Ingrid Halvorsen, pharmacometrician) and negative fixtures (`bad-jobposting.md`, `synthesis-no-dissent.md`).

Panels scaffold into the *target* repo as `panels/<slug>/` with a `panel.json` manifest (status lifecycle `calibrating → active → dormant → archived`, inherited kernel version, overlay set, roster paths), a README with the calibration record, and one "Standing panels" line added to the repo's `AGENTS.md`. Discovery is `find . -name panel.json` — the manifest convention replaces a registry.

## Key techniques

**Persona-as-retrieval-key, held as a hypothesis.** Each persona element is a pointer into expert-text territory of the training distribution; a complete coherent set lands the model where it continues convincingly. After the self-critique, the doctrine now explicitly separates four outcomes that do not co-vary — expert-like expression, useful attention, sound reasoning, actual correctness — and warns that an authoritative name and signature can buy the first without any of the rest. Declared blind spots are "instruments, not guarantees": they may make an agent more correctable or teach it to enact a predictable error, and both deserve testing.

**Semantic versioning for prompt doctrine.** The kernel is released like software: 1.0.0 extracted from the proven panels, 1.1.0 adding the script-checked dissent rule and contribution review, 1.2.0 restating rule 1 as a *visibility constraint* (the facilitator's clustering, contradiction-finding, and significance calls are analytical acts that must be challengeable in the synthesis output — what's forbidden is silently substituting its own conclusions for missing specialist work). Panels record the version they inherited; adopting 1.2.0 is a deliberate panel refresh, not auto-inheritance. New sequencing is a new overlay designed once against the kernel — never an ad hoc mutation inside one panel.

**The determinism boundary (kernel rule 9).** Deterministic scripts verify only structure — sections present, word floors (300 for personas, 100 for syntheses), dissent non-empty. Anything about quality, voice, or significance stays model-mediated. `validate-synthesis.sh` makes consensus collapse structurally impossible to pass silently: a synthesis without a substantive `## Dissent` section is invalid and gets re-run, and an empty dissent on a genuinely contested question is a convergence-collapse *warning*, not success.

**diversity.py as a cheap smoke detector.** Pure-stdlib lexical cosine distance across the seats' isolated outputs, with advisory thresholds: closest pair under 0.15 means two seats may secretly be the same seat; mean under 0.25 suggests premature convergence. It deliberately avoids embeddings so it always runs — the contract (distances in, advisory report out) would let embedding-based checks layer on later.

**Contribution review with an exposure-boundary subtlety.** Every run closes with a per-seat record: produced, kept, discarded-or-changed, unique contribution, misses. The counterfactual check re-runs *only the synthesis step* with one seat's input withheld — and the doc is unusually honest about what that measures: only direct marginal contribution at that stage, because if other seats already read the seat's work, its influence propagated and the cut must be made at the relevant **exposure boundary** instead; and a changed synthesis may be sampling variation, so repeat before reading signal. The honesty rule: no causal-lift claims without a counterfactual — traceable contribution only.

**Assignment decomposition (pilot).** Instead of casting "which expert should we spawn?", the overlay decomposes the question into reasoning needs along six dimensions — object of attention, reasoning operation, expert lens, stance, information exposure, required contribution — then groups rows into seats by a *why-chain* boundary test (if later steps need to know *why* earlier steps concluded, keep the chain in one instance). Sizing becomes an output of the grouping, not an input; casts are marked `dimension-derived` in the manifest so a roster-vs-dimension comparison can be cut later. The persona drops to being the "delivery vehicle" for an assignment.

**Anti-anchoring process design.** Specialists work in isolation before any exposure (early exposure anchors every later voice to the first confident output); cross-reading happens only in convergence rounds; summaries before full transcripts; the adversarial seat runs last, after positions have hardened enough to be worth attacking. Checkpoints after each synthesis triage user feedback into minor (fold in) vs. major (offer a re-round), with round ceilings forcing convergence.

## Design decisions

- **Prose over code.** The methodology is markdown, portable across Pi, Claude Code, and Codex by design. Trade: nothing executes except three scripts, so every guarantee beyond structure rests on the host model reading and obeying the doctrine. The bundle tests (`verify-bundle.sh`, fixture-driven bash tests) are the counterweight — they pin the *documentation*, not behavior.
- **A deliberately tiny deterministic surface.** Agent-studio's deterministic evidence gate was explicitly rejected — its corpus studies thin label-personas, not doctrine-built ones — as was famous-exemplar casting. The bet: regex quality gates would either fail good personas or pass bad ones, so judge voice with a model and only count structure with code.
- **Locality and no registry.** Panels live where the work happens; discovery by manifest convention plus an `AGENTS.md` line. Trade: no index to rot, but discovery depends on convention discipline holding across repos.
- **Calibration as a hard gate.** No persona is drafted before the user confirms goal and roster, recorded on disk via the `calibrating → active` status flip. Cost: one mandatory exchange; benefit: nobody is ever handed a finished cast they didn't ask for.
- **Self-critique as a workflow.** The 2026-09-09 `impact-on-intent` session (a Pi agent prompted to review the author's own writing "very critically") produced seven foundational issues: the retrieval-key claim outran its evidence; "the moderator never analyzes" is literally unachievable; terminology preservation is not insight preservation; the counterfactual was narrower than its ambition; fixed conventions were posing as laws of cognition; the persona is not the right unit of assignment; and decomposition needs a boundary principle. The repo converted these into kernel 1.2.0, an updated contribution-review doc, and the pilot overlay — treating every canon claim as a hypothesis, resolved by versioned experiment under an "abundance premise" (tokens and agents are cheap, so alternative arrangements are affordable to test).
- **My take.** The epistemic hygiene is genuinely ahead of the field — most agent frameworks never distinguish "the seat's words survived" from "the seat's reasoning mattered". But the evaluation machinery is thin where it matters: lexical cosine is a crude proxy for divergence (shared vocabulary can hide real disagreement and vice versa), the counterfactual protocol has a caution about sampling variation but no statistics, and the contribution reviews are written by the same model that did the synthesizing. The retrieval-key theory has no measurement plan beyond "record what you notice" in reviews. The skill's own rules would call all of this traceable contribution, not causal lift — it mostly does, which is to its credit.

## Comparison notes

- [[Pi Subagents]] — the runtime protocol executes through Pi's subagent tool's `workflowScript` call, i.e. this skill is a methodology layer running on top of pi-subagents' delegation substrate. pi-subagents moves data between child sessions; agent-designer decides what the seats *are* and how they must relate.
- [[Swarm Skill]] — the other obvious "orchestration shipped as a skill". Swarm's 14 rules are failure-derived operational hardening (rate limits, cost routing, void detection); agent-designer's rules are a priori methodology canon, versioned like an API and shipped with its own refutation cycle. Failure-driven vs. theory-driven, and they would strengthen each other.
- [[What I learned building an opinionated and minimal coding agent]] — the host-harness tension. Pi's stated philosophy is minimalism (no subagents, no plan mode); this skill's elaborate multi-persona cast only exists because the ecosystem grew the missing pieces (pi-subagents) as extensions. The minimal core stayed minimal; the orchestration arrived around it.
- [[Guardrails and Feedback Loops]] — agent-designer is a worked example of this topic's central split: deterministic checks for structure, model-mediated judgment for quality, plus review gates (dissent validation, contribution review) that make quality self-correcting. Its honesty rule — traceable contribution vs. counterfactual-backed causal claims — is a stricter standard than most evaluation writeups in the wiki.

---
*Sources: [[raw/agent-designer]], [[summary/agent-designer]]*
*Last updated: 2026-09-15*
