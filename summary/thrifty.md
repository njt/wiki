---
url: https://skills.2389.ai/plugins/thrifty/
title: thrifty — A Tiered-Delegation Execution System for Claude Code
author: 2389 Research Inc
date_fetched: 2026-07-03
date_published: 2026
topics:
  - agent-architecture
---

# thrifty — Agent Systems Plugin for Claude Code

**Source:** 2389 Research Inc marketplace (2389.ai)
**Version:** v0.5.0
**Tagline:** A tiered-delegation execution system for Claude Code.

## Summary

thrifty implements a multi-model orchestration approach where a stronger model (Sonnet) plans work into sprints, a cheaper model (Haiku) builds and self-verifies against defined gates, and Sonnet steps in only when failures occur. The project is benchmarked as "~64% cheaper than Opus at equal quality."

**Categories:** orchestration, delegation, tiered-build, multi-agent, codegen

## Install Methods

- **npx (any agent):** `npx skills add 2389-research/thrifty`
- **Claude Code plugin:** `/plugin install thrifty@2389-research`
- **Local symlinking:** `for d in skills/*/; do ln -s "$PWD/$d" ~/.claude/skills/; done`

## Core Concept

The guiding principle is to "offload each sprint of work to the weakest model that can do it correctly." The architecture treats gates (tests/checklists) as the trust contract — independently re-run rather than self-reported.

### Benchmarked Architecture

A spec goes to **Sonnet**, which writes a contract (pinning cross-sprint and genuinely ambiguous decisions) plus `sprints.jsonl`. **Haiku** then runs as a single cached agent building every sprint, executing the gate, and self-fixing. **Sonnet** applies a scoped patch only if a specific failure remains (which reportedly never happened in testing — Haiku self-fixed to green on all 7 tasks).

The result: "~64% cheaper than Opus building the same spec, at equal gate quality" on large, multi-unit builds. On trivial builds, planning overhead makes thrifty more expensive.

## Skills (6 total)

| Skill | Role | Model |
|---|---|---|
| `thrifty` | Orchestrator (subagent flow) | Current session |
| `thrifty-dispatch` | Orchestrator (lean dispatch — **default**) | Current session |
| `thrifty-plan` | Director's planning | Sonnet |
| `thrifty-brief` | Expand unit spec into brief (split tier) | Sonnet |
| `thrifty-execute` | Execute one unit | Haiku |
| `thrifty-check` | Verify + fix one unit | Sonnet |

## Planning Tiers

Three approaches for who writes briefs:

- **Direct:** The architect writes both contract and every brief. Best for "few sprints / subtle, correctness-critical work."
- **Split:** The architect writes contract + terse unit specs; parallel Sonnet brief-writers expand them. Best for "many sprints (≳ 6) / mechanical briefs / scale."
- **Hybrid:** The architect writes the 1–2 subtle sprints' briefs, delegates the rest.

Rough rule: direct below ~5 sprints, split above.

## Decomposition Modes

Three modes for structuring work:

- **Partition:** Sprints own separate regions/files, run in parallel, architect merges. Best for separable outputs.
- **Relay:** One shared artifact extended segment by segment, sequentially. Best for flowing prose.
- **Layered:** Role-specialized passes over the whole artifact (draft → continuity edit → polish), sequentially. Best for "one seamless voice via multiple lenses."

A single artifact with no cross-sprint seams skips the contract entirely.

## Verification

Verification is adaptive: runnable criteria (tests, build, CLI exit codes) just run the gate — no model cost on pass. If a gate fails, a Sonnet checker reads and surgically fixes it. Assertional criteria (prose, citations, design quality) require a Sonnet checker to read and judge. So for code with tests, "the common path costs no checker tokens."

## Fix Loop (Bounded)

1. **Surgical** (Sonnet, in place) — ≤ 2 attempts
2. **Executor redo** (fresh Haiku + notes) — ≤ 1
3. **Architect replan** (revise contract/brief) — ≤ 1
4. Otherwise **surface to the human**

A regression guard rolls back any fix breaking a previously-passing criterion.

## Two Implementations: Dispatch vs. Subagent

**Recommended: `thrifty-dispatch` (lean dispatch flow).** This is the benchmarked architecture underpinning the ~64% cost claim. It uses bare `claude -p` calls hard-pinned to Haiku, writing outputs to disk; the orchestrator reads only a tiny manifest.

**Subagent substrate:** Richer but pricier. Every subagent spawn carries ~40k of Claude Code harness context, and each report re-enters the orchestrator's context, compounding cost with unit count. The dispatch flow avoids both. A caveat: the subagent path "cannot force the executor model in code" — a runtime may silently fall back to Sonnet, so the ledger's savings line is "an estimate, not a measurement."

## Artifacts

Outputs land in `docs/thrifty/<task-slug>/` (CONTRACT.md, briefs/, LEDGER.md) — auditable and resumable.

## Lineage

thrifty generalizes three prior systems:

1. **speed-run** — offloads first-pass generation to a fast model; the strong model does architecture and surgical fixes only.
2. **pipelines/local_code_gen** — a strong architect pins every cross-sprint decision so the executor never makes system-level choices.
3. **Noospheric Orrery** — proof of philosophy where Sonnet writes/refines the spec until a cheap model reliably passes.

## Trigger Phrases

Say **"thrifty"**, **"delegate this"**, or **"tiered build"** on a multi-part task. The agent tries dispatch first and falls back to subagent flow when dispatch runtime is unavailable.

## Related Plugins (from same publisher)

- **building-multiagent-systems** (v1.0.0) — patterns for multi-agent orchestration and coordination
- **deliberation** (v1.0.0) — decision-making through discernment rather than debate
- **review-squad** (v1.0.0) — dispatch panels of specialized subagents for reviews and audits

## Publisher Info

**2389 Research Inc** — "Building tools for how we actually work." All plugins are open source, hosted at github.com/2389-research/claude-plugins. Contact: hello@2389.ai. © 2026.

## Key Links

- GitHub source: https://github.com/2389-research/thrifty
- Benchmark results: https://github.com/2389-research/thrifty/blob/main/eval/RESULTS.md
- Reproducible experiments: https://github.com/2389-research/thrifty/blob/main/experiments/README.md
