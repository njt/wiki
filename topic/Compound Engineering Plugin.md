# Compound Engineering Plugin

The shipped implementation of the [[Compound Engineering]] philosophy: a 33-skill plugin for AI coding agents that runs the brainstorm → plan → work → simplify → review → **compound** loop across 14 agent hosts, with a TypeScript converter that translates one Claude-native skill format into each host's plugin format. Where the Every.to essay described the methodology, this repo is the machinery — and the machinery is more interesting than the essay implied.

---

## Architecture

Two halves, cleanly separated:

**The skills** (`skills/<skill>/SKILL.md` + `references/` + `scripts/`). Each skill is a directory; `SKILL.md` is the entrypoint with YAML frontmatter (`name`, `description`, `argument-hint`, plus custom fields like `ce_platforms`). The body follows a strict pattern: an **Outcome** (what it produces and who consumes it), a **Done** bar (the completion condition), and a phase list where each step names its `references/` file as a *required read at the acting step*. This is the "phase-loaded kernel" pattern from `CONCEPTS.md`: the always-loaded body is reduced to what must fire without any reference read, and everything else is pulled in lazily — an explicit defense against host prompt-budget truncation.

**The converter** (`src/`, ~8K lines of TypeScript on Bun). `src/index.ts` is a `citty` CLI with `convert`, `install`, `cleanup`, `list`, `plugin-path`. `src/parsers/claude.ts` parses the Claude plugin manifest and skill files; per-target converters (`src/converters/claude-to-*.ts`) map tools, permissions, hooks, and model names *explicitly*; per-target writers (`src/targets/*.ts`) emit the bundle to the host's expected paths. The design vocabulary (`Converter`, `Writer`, `Bundle`, `Target`, `Install manifest`) is defined in `CONCEPTS.md`, which reads like a formal spec for a plugin-federation problem most projects solve with ad-hoc scripts.

Two architecture decisions stand out:

- **Skills seed subagents; there are no standalone agents.** `CONCEPTS.md` is explicit: most specialist behavior lives as "specialist prompt assets" — internal persona files owned by one skill — that get handed to a *generic* subagent at dispatch time. This inverts the original Every design (the essay described 26 standalone agents like `security-sentinel`), and it's the load-bearing choice: the skill controls *when* the persona loads, *which* model serves it, and *how* its output merges.
- **The install-manifest ledger** (`src/utils/legacy-cleanup.ts`, 1,199 lines). Writers record exactly which paths they wrote, so reinstall never overwrites a path the user has replaced (a symlink to a personal fork, a hand-authored dir). The invariant: *a writer never claims a path it did not write* — unowned paths are preserved, and the ledger self-heals when an override is removed.

## Key techniques

- **Progressive disclosure as a survival strategy.** Skill bodies are kept under per-host prompt budgets by moving load-bearing content into `references/` files behind "load stubs" — instructions that name what the reference contains and the failure mode of skipping it, deliberately omitting anything the agent could improvise. `CONCEPTS.md` documents the reasoning: every known host truncation keeps the *beginning* of a body and silently drops the tail, so ordering is load-bearing.
- **The compound loop's memory is a repo directory, not a database.** `ce-compound` writes one "learning" (a solution doc with structured frontmatter) to `docs/solutions/` per run; `ce-brainstorm` and `ce-plan` read those learnings as grounding. A "durable bar" gate (`<!-- ce-durable-bar -->`) uses a counterfactual — *if this doc disappeared, would a future engineer redo the investigation?* — to decide whether anything gets written at all.
- **Cross-model corroboration with receipts.** Review/planning skills can dispatch a "cross-model pass" to a different provider, but the run only counts it as independent when it can verify the *serving model family*, not merely the requested one. `CONCEPTS.md` calls this the "model identity receipt": outputs without one are labeled requested-but-unverified.
- **The converter's frontmatter round-trip guard** (`src/utils/frontmatter.ts`): when emitting YAML, any bare string that js-yaml would read back as a non-string (a date, a bool, `null`) is quoted — re-parsing with the *same loader* keeps the rule in lockstep with the parser's grammar rather than a hand-maintained token list.
- **Deterministic scaffolding around LLM judgment.** The skills lean on bundled scripts for anything mechanical: `ce-sweep/scripts/sweep-state.py` (feedback-source cursors), `ce-resolve-pr-feedback/scripts/get-pr-comments`, `ce-compound/scripts/validate-frontmatter.py`. The [[Smart Models Dumb Pipes]] pattern: the model decides, code enforces.

## Design decisions

- **Claude Code is the source of truth; everything else is a compilation target.** The repo is authored once in Claude's skill format, then converted. This is a bet that the Claude skill format is the most expressive common denominator — and it means the 14 hosts are second-class by construction, each inheriting whatever the converter can faithfully map (Devin, for instance, drops some `allowed-tools` names to permission prompts).
- **Report-only review, local apply is explicit.** `ce-code-review` does not fix anything by default; it produces findings. Autonomy is opt-in (`/lfg` applies fixes), not the default. This is the safety design the philosophy page's "skip permissions" concern was pointing at.
- **Host prompt budgets as a first-class constraint.** The plugin ships to hosts with wildly different body-size ceilings, so the skills are engineered around truncation — a discipline most skills packages ignore because they only test one host. This is the quiet reason the project survives being "run on 14 hosts" rather than just "installable on 14 hosts."
- **Cost is controlled by model tiers and evidence dossiers, not by trimming capability.** Cheap "extraction" subagents gather verbatim quotes into scratch files; the orchestrator carries a gist and downstream agents read the dossier. When a host can't pick per-agent models, cost control falls back to read budgets and output caps.

## Comparison notes

[[Compound Engineering]] (the philosophy page) describes the *idea*; this is the *implementation*, and it has moved past the essay's snapshot. The essay counted "26 agents, 23 commands, 13 skills" and standalone review agents; the shipped plugin is 33 skills with specialist personas seeded into generic subagents, and it runs on 14 hosts, not just Claude Code. The plugin is also the concrete answer to the essay's open question — "what's the measured payback?" — because the `docs/solutions/` learnings are the compounding asset made visible.

[[Claude Code Skills System]] is the substrate: `SKILL.md` directories, frontmatter, subagent delegation, lazy loading. This plugin is arguably the most sophisticated skills package built on that substrate — it pushes the "progressive disclosure as architecture" idea far past the official docs, engineering around prompt-budget truncation the reference never mentions.

[[Loop Engineering]] is the meta-skill; this plugin is a full five-component loop shipped as a product, with the distinctive addition of a *return arrow* — the compound step that feeds each cycle's learning into the next. Most loops Osmani surveys generate and review; very few write the learning back.

[[Agent-Native Architectures (Every)]] is the same company's design guide (files as the universal interface, improvement over time), and the plugin is that guide applied to the engineering process itself: learnings and plans are files, capability improves without redeploying code, and the "improvement over time" principle is literal — the toolset gets smarter each run.

#tool #project #agents #compound-engineering #skills #code-review

---
*Sources: [[raw/compound-engineering-plugin]], [[summary/compound-engineering-plugin]]*
*Last updated: 2026-09-08*
