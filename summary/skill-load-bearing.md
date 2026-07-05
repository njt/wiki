---
url: https://github.com/danshapiro/skill-load-bearing
title: skill-load-bearing — Load-Bearing Assumptions Skill for AI Coding Agents
author: Dan Shapiro (danshapiro)
date_fetched: 2026-06-03
date_published: 2026-05
---

# skill-load-bearing — Load-Bearing Assumptions

A Claude Code / Codex skill for surfacing and verifying the falsifiable, currently-unproven claims a code plan depends on. The repo is a pure specification — 295 lines of markdown across 3 files. No runtime code.

## Repo Structure

```
skill-load-bearing/
├── README.md                                       (30 lines)
├── load-bearing-assumptions/
│   ├── SKILL.md                                    (114 lines — workflow + design)
│   └── references/
│       └── subagent-prompts.md                     (151 lines — copy-paste prompts)
```

## Architecture Pattern

**Multi-agent verification workflow** with human-in-the-loop checkpoints. Five sequential phases:

0. **Frame** — Collect the plan, restate the goal
1. **Discover** (stateful finder subagent, maximum capability) — Enumerate EVERY falsifiable load-bearing assumption. Bias toward exhaustive discovery.
2. **Strategize** (separate stateful strategist subagent, maximum capability) — Assign each assumption the cheapest reliable validation method, with fallback chains.
3. **Validate** (parallel stateless validator subagents, maximum capability) — One per validation target. Run code > inspect code > official docs > broader internet.
4. **Evaluate & loop** — Judge evidence skeptically. Inconclusive? Send back down the fallback chain. New assumptions surfaced? Add to ledger. Falsified? Flag immediately.
5. **Report** — Verified facts, falsified assumptions requiring plan changes, accepted residual risks.

Three checkpoints are reserved for the human operator (not delegated): reviewing the finder's list for missed assumptions, inspecting the strategist's method assignments for weak choices, and evaluating validator evidence.

## Key Techniques

### Late-Falsification Cost Matrix

A four-tier cost classification that determines whether an assumption is worth verifying:

| Cost | Meaning | Treatment |
|------|---------|-----------|
| `low` | Small local detail — field name, function call, flag | Usually don't validate. Let implementation discover it. |
| `medium` | Local helper/test/module design changes, plan survives | Validate only if cheap, risky, or naturally grouped |
| `high` | Subsystem boundaries, data model, concurrency, API surface must be rethought | Validate early |
| `critical` | Production data loss, downtime, security exposure, failed rollback | Validate early, with strong evidence |

This is the framework's core innovation: it's not "verify everything" — it's "verify what would be expensive to get wrong."

### Three Gates for Assumption Qualification

An assumption only counts if it passes all three:
1. **Decision-controlling** — if false, a planning-level decision changes
2. **Currently unproven** — not already established by context/evidence
3. **Falsifiable** — a concrete claim evidence can prove true or false

"`config.load()` reads `$HOME/.app/config.yaml`" qualifies; "the config system is well-designed" does not.

### Validation Method Preference Tree

Default preference, most to least reliable for code behavior:
1. Run code, observe output
2. Inspect code + static analysis
3. Official documentation
4. Broader internet

Override by weighing feasibility, safety, and question type. The tree itself is the default, not a reflex.

### Fallback Chains

When uncertain a single method will resolve a question, specify an ordered chain: `run (foo.sh --help) → inspect (arg parsing) → docs`. Validator walks the chain; one reliable method suffices when no fallback is needed.

### Stateful Finder + Strategist, Stateless Validators

Finder and strategist are stateful subagents — you send feedback and they refine, not re-derive. Validators are stateless and parallel — fresh context each, independent, no contamination. This is a deliberate two-speed design: slow iteration on analysis, fast parallel execution on verification.

### Assumption Ledger

Single source of truth tracking all assumptions across phases:

| ID | Assumption (falsifiable claim) | Decision controlled | Current evidence / uncertainty | Late-falsification cost | Method (+ fallback chain) | Status | Evidence / finding |

Status: `unverified` → `verifying` → `verified` / `falsified` / `accepted-risk`.

## Innovation Points

1. **"Load-bearing" framing is the key insight.** Most verification approaches are indiscriminate — they chase every unknown. This skill only verifies what would change the plan if wrong. The gate system (decision-controlling + unproven + falsifiable) is the filter.

2. **Late-falsification cost as prioritization.** The cost matrix doesn't just say "this matters" — it says HOW it matters and exactly when to invest. Low-cost assumptions are explicitly deferred. This is an economic argument, not a completeness argument.

3. **Validation method tree defaults to running code over reading docs.** The obvious approach is "check the docs first." This skill argues the opposite for code behavior: code IS the ground truth, documentation is a secondary source. It's a falsification mindset applied to the verification process itself.

4. **Stateful/stateless subagent split.** Finder and strategist need to incorporate feedback without re-deriving → stateful. Validators need independence and parallelism → stateless. This is a sophisticated understanding of subagent lifecycle that most orchestration frameworks don't distinguish.

5. **Assumptions as a first-class data type.** The ledger format forces every assumption to be a specific, testable claim with explicit stakes. This is specification-by-constraint: the shape of the output forces the quality of the thinking.

## Design Trade-offs

**Optimized for correctness over speed.** Five phases, three human checkpoints, iterative loops — this is not fast. It's designed for plans where getting an assumption wrong is expensive enough to justify the overhead.

**Human-in-the-loop by design, not by limitation.** The three checkpoints aren't "we couldn't automate this" — they're "automation would be worse." The finder is biased toward false positives (completeness over precision). The strategist uses default preferences that may not fit. The validators may return weak evidence. Each checkpoint is a quality gate that requires judgment the current generation of models doesn't reliably supply.

**Grouping over maximizing parallelism.** Validators group assumptions when one safe action can validate several. This trades raw parallelism for efficiency — fewer subagents, less duplicated work, lower token cost. But it risks making validator tasks ambiguous. The skill explicitly warns against this.

**Read-only, non-destructive verification only.** Validators can run commands freely but must not delete, overwrite, mutate, install, or make external/production writes without explicit user approval. This is a safety constraint that may occasionally force a fallback to less reliable methods (inspection instead of running).

**Markdown-as-code.** The entire skill is markdown with no executable code. It relies on the agent harness to interpret the workflow. This is both a strength (portable across Claude Code, Codex, and other platforms) and a limitation (no tool-enforced constraints, relies on the LLM following instructions).

## Prompt Engineering

The subagent prompts (references/subagent-prompts.md) are mini masterpieces of LLM instruction design:

- **Role priming:** "You are finding the load-bearing assumptions in a plan. Think as hard as you can."
- **Structured output:** Each role has a clearly specified output shape (findings list, method assignments, verdicts)
- **Bias calibration:** The finder is told "completeness matters more than precision" and given explicit anti-bias instructions ("probe the easily-missed categories")
- **Fallback behavior:** Validators are told "on failure, ambiguity, or inconclusive output, proceed to the next link"
- **Safety constraints:** Validators are told exactly what they cannot do, with the alternative ("report it as a blocked step")

## Comparison to Related Projects

**vs. [[Components of a Coding Agent]] (Raschka):** Raschka's taxonomy identifies "context quality" as the key differentiator; this skill operationalizes it for planning — it's a specific mechanism for improving the quality of what goes into context before the plan executes.

**vs. [[Orchestrator - Worker Skill]]:** That skill merges orchestrator and worker into one file. This skill separates them into three distinct subagent roles with different statefulness properties. The merge approach is simpler; the separate approach is more rigorous at the cost of more moving parts.

**vs. [[StrongDM Factory Techniques]] (Pyramid Summaries, Shift Work):** StrongDM's catalog is about building software without reading code. This skill is about verifying assumptions before building software. They're complementary: StrongDM's techniques operate during execution; this skill operates before execution.

**vs. [[Guardrails and Feedback Loops]]:** The guardrails philosophy is "deterministic enforcement, not instructions." This skill is pure instructions — there's no tool-enforced constraint, only prompt engineering. It trusts the LLM to follow the workflow. Whether that's a weakness depends on how well current models follow structured multi-step instructions.

**vs. [[Agent Orchestration]] and [[Agent Coding Workflow]]:** This skill adds a specific phase to the agent coding workflow: pre-planning assumption verification. It's a new stage in the maturity spectrum — between "vibes" and "compound engineering" sits "verified assumptions."

## Critical Assessment

The skill is elegant and well-specified. The three gates are the right ones. The late-falsification cost matrix is genuinely novel. The stateful/stateless split is sophisticated.

The main weakness is that it relies entirely on LLM instruction-following for enforcement. There's no tool that validates the ledger format, no mechanism that prevents a validator from skipping its fallback chain, no check that the finder actually enumerated every assumption. Compare with [[Structural Backpressure Beats Smarter Agents]] which argues for type-system enforcement over prompt-level pleading. This skill is all pleading.

The second weakness is cost. "Spawn a max-capability subagent" × 3 roles × N validators × potentially multiple loop iterations — this could burn serious tokens. The skill acknowledges this implicitly through its emphasis on grouping and deferring low-cost assumptions, but doesn't provide explicit budget guidance.

The third weakness is that it assumes the plan exists in a form the finder can reason about. For informal plans ("let's add auth to the app"), the finder may struggle to enumerate assumptions because there's not enough structure to anchor on. The skill doesn't address this upfront.

Still: this is the first skill I've seen that treats assumption verification as a first-class engineering discipline rather than a byproduct of "being careful." The fact that it's 295 lines of markdown with no code is both its greatest strength (any agent can run it) and its greatest limitation (no mechanical enforcement). For plans where getting an assumption wrong would be genuinely costly — database migrations, auth systems, payment flows, deployment to production — the overhead is justified.
