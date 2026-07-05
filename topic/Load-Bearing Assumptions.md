# Load-Bearing Assumptions

A Claude Code/Codex skill that surfaces and verifies the falsifiable, unproven claims a code plan depends on — the assumptions that, if wrong, would force you to redo architecture rather than fix a local detail. Unlike generic "be thorough" advice, it provides a specific multi-agent workflow (finder → strategist → parallel validators) with a cost-based prioritization framework. 295 lines of markdown, no executable code.

---

## Key Quotes

> "Plans don't fail on the things you know are risky. They fail on things you were sure of but never checked."

> "The cheapest place to catch a wrong assumption is before the first line of code."

> "Completeness matters more than precision — a missed critical assumption is costlier than a few extra checks."

On the validation preference hierarchy:
> "Run code, observe output... Inspect code + static analysis... Official documentation... Broader internet. Override this default by weighing feasibility, safety, and question type."

On the stateful/stateless split:
> "Finder and strategist are stateful — you send feedback and they refine, not re-derive. Validators are stateless and parallel — fresh context each, independent, no contamination."

## Architecture

The skill defines a five-phase verification workflow with three human-in-the-loop checkpoints. All logic lives in two files: `load-bearing-assumptions/SKILL.md` (workflow + when to use) and `references/subagent-prompts.md` (copy-paste prompts for the three subagent roles).

**Phase structure:**

| Phase | Role | Agent Type | Model |
|-------|------|-----------|-------|
| 0. Frame | Human | — | — |
| 1. Discover | Finder | Stateful, max capability | Most capable available |
| 2. Strategize | Strategist | Stateful, max capability | Most capable available |
| 3. Validate | Validators | Parallel, stateless | Most capable available |
| 4. Evaluate | Human | — | — |
| 5. Report | Human | — | — |

Three checkpoints are reserved for the human operator — these are quality gates, not automation gaps:
1. Review the finder's list for missed assumptions (implicit environmental, version, data-shape, and concurrency assumptions are commonly overlooked)
2. Inspect the strategist's method assignments for weak choices
3. Judge validator evidence skeptically — does the evidence actually establish the claim?

**The stateful/stateless split** is deliberate. Finder and strategist are stateful so they incorporate feedback without re-deriving. Validators are stateless and parallel so they stay independent, fast, and uncontaminated by each other's context.

## Key Techniques

### Late-Falsification Cost Matrix

The framework's core innovation. Not "how important is this?" but "what happens if I learn this is wrong during implementation?"

- **`low`**: A field name, function call, or flag changes. Plan survives. → Usually don't validate.
- **`medium`**: Local module design changes, plan survives. → Validate only if cheap or risky.
- **`high`**: Subsystem boundaries, data model, concurrency, API surface must be rethought. → Validate early.
- **`critical`**: Production data loss, downtime, security exposure, failed rollback. → Validate early, with strong evidence.

### Three Qualification Gates

An assumption only counts if it passes all three:
1. **Decision-controlling** — if false, a planning-level decision changes
2. **Currently unproven** — not already established by context or trusted evidence
3. **Falsifiable** — a concrete claim evidence can prove true or false

"`config.load()` reads `$HOME/.app/config.yaml`" qualifies; "the config system is well-designed" does not.

### Validation Method Preference Tree

Defaults to most-reliable-first for code behavior:
1. **Run code, observe output** — direct evidence of actual behavior
2. **Inspect code + static analysis** — when running is infeasible or unsafe
3. **Official documentation** — for documented contracts, supported versions
4. **Broader internet** — for undocumented behavior; weakest, must corroborate

This is a default, not a reflex. Override by weighing feasibility, safety, and question type. When uncertain, specify a **fallback chain**: `run (foo.sh --help) → inspect (arg parsing) → docs`.

### Assumption Ledger

Single source of truth across all phases:

| ID | Assumption | Decision controlled | Evidence/uncertainty | Cost | Method + fallback | Status | Finding |

Status: `unverified` → `verifying` → `verified` / `falsified` / `accepted-risk`.

### Grouping Over Maximizing Parallelism

When one safe command, code inspection, or source can validate several assumptions, group them — unless the combined task would become ambiguous or weaken the evidence. This trades raw parallelism for efficiency.

## Design Decisions

**Speed sacrificed for correctness.** Five phases, three human checkpoints, iterative loops — this is not fast. It's designed for plans where getting an assumption wrong is expensive enough to justify the overhead.

**Human-in-the-loop by choice, not limitation.** The checkpoints aren't "we couldn't automate this" — they're places where current models don't reliably supply the required judgment. The finder is biased toward false positives (completeness over precision). The strategist uses defaults that may not fit. Validators may return weak evidence.

**Read-only, non-destructive verification only.** Validators can run commands freely but must not delete, overwrite, mutate, install, or write to production without explicit user approval. This safety constraint sometimes forces a fallback to less reliable methods.

**Markdown-as-code.** No executable code, no tool-enforced constraints. The skill relies entirely on the LLM following instructions. This makes it portable across Claude Code, Codex, and other platforms — but means there's no mechanical enforcement of the ledger format, the fallback chain, or the non-destructive constraint.

## Comparison Notes

**vs. [[Orchestrator - Worker Skill]]:** That skill merges orchestrator and worker into one file. This skill splits into three distinct subagent roles with different statefulness properties. The merged approach is simpler for small workflows; the separated approach handles complexity better at the cost of more moving parts.

**vs. [[StrongDM Factory Techniques]]:** StrongDM operates during execution (Shift Work, Pyramid Summaries, Gene Transfusion). This skill operates before execution — it verifies the plan's foundations. They're complementary stages, not alternatives.

**vs. [[Guardrails and Feedback Loops]]:** The guardrails philosophy is "deterministic enforcement, not instructions." This skill is all instructions — no tool-enforced constraint. [[Structural Backpressure Beats Smarter Agents]] would argue this is the skill's fundamental weakness: capability and certainty are different things, and prompt-level pleading delivers capability without certainty.

**vs. [[Components of a Coding Agent]] (Raschka):** Raschka identifies context quality as the key differentiator; this skill operationalizes it for planning — it improves the quality of what goes into context before execution begins.

**vs. [[Agent Coding Workflow]]:** This skill adds a specific phase to the workflow: pre-planning assumption verification. It sits between "vibes" (no verification) and "compound engineering" (full harness validation) as a lightweight middle ground.

## Critical Assessment

The skill is elegant: the three gates are the right ones, the late-falsification cost matrix is genuinely novel, and the stateful/stateless split shows sophisticated understanding of subagent lifecycle.

Three weaknesses:

1. **No mechanical enforcement.** Everything relies on LLM instruction-following. There's no tool that validates the ledger format, prevents a validator from skipping its fallback chain, or checks that the finder enumerated all assumptions. This is prompt engineering, not engineering.

2. **Token cost at scale.** Max-capability subagents × 3 roles × N validators × loop iterations adds up fast. The grouping and deferral mechanisms help but don't eliminate the concern.

3. **Assumes structured plans.** The finder needs something to reason about. For informal plans ("add auth to the app"), it may struggle — not enough structure to anchor assumption enumeration.

Still: this is the first skill that treats assumption verification as a first-class engineering discipline rather than a byproduct of being careful. For plans where getting an assumption wrong would be genuinely costly — database migrations, auth systems, payment flows, deployment to production — the overhead is justified.

---
*Sources: [[summary/skill-load-bearing]]*
*Related: [[Agent Coding Workflow]], [[Components of a Coding Agent]], [[Orchestrator - Worker Skill]], [[StrongDM Factory Techniques]], [[Guardrails and Feedback Loops]], [[Structural Backpressure Beats Smarter Agents]], [[Agent Orchestration]]*
*Tags: #tool #project #agents #skills #verification*
*Last updated: 2026-06-03*
