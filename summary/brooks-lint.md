---
url: https://github.com/hyhmrright/brooks-lint
title: "brooks-lint: AI code reviews grounded in twelve classic engineering books"
author: hyhmrright
date_fetched: 2026-06-15
date_published: 2026-03-26
topics:
  - guardrails-and-feedback-loops
---

# brooks-lint — Full Analysis

## Overview

brooks-lint is a Claude Code plugin (v1.3.0, MIT) that provides AI-powered code reviews grounded in 12 classic software engineering books. It operates as six independent skills (PR review, architecture audit, tech debt assessment, test quality review, health dashboard, full sweep) across platforms supporting Agent Skills (Claude Code, Gemini CLI, Codex CLI, Cursor, Windsurf, OpenCode, Antigravity, pi, Copilot, Kiro, Factory Droid).

At its core, it's a **pure prompt-engineering product** — the "source code" is markdown files that get concatenated into LLM system prompts. No AST parsing, no static analysis, no custom code logic beyond the scaffold scripts that assemble prompts and call the Anthropic API.

## Architecture

### File structure

```
brooks-lint/
├── .claude-plugin/plugin.json    # Claude Code marketplace metadata
├── .codex-plugin/plugin.json     # Codex CLI plugin metadata
├── gemini-extension.json         # Gemini CLI extension metadata
├── hooks/
│   ├── session-start             # Shell script that invokes session-start.mjs
│   └── session-start.mjs         # Auto-installs slash commands, injects context
├── commands/                     # Short-form slash command wrappers (brooks-review.md, etc.)
│   ├── brooks-review.md
│   ├── brooks-audit.md
│   ├── brooks-debt.md
│   ├── brooks-test.md
│   ├── brooks-health.md
│   └── brooks-sweep.md
├── skills/
│   ├── _shared/                  # Framework loaded into every mode's system prompt
│   │   ├── common.md             # Iron Law, config, report template, health score, history
│   │   ├── decay-risks.md        # R1-R6: symptoms, sources, severity guides, false-positive guards
│   │   ├── test-decay-risks.md   # T1-T6: test decay risk definitions
│   │   ├── source-coverage.md    # 12-book coverage matrix with "do not over-flag" rules
│   │   ├── remedy-guide.md       # --fix mode enhancement rules
│   │   └── custom-risks-guide.md # Template for project-specific Cx risk codes
│   ├── brooks-review/            # Mode 1: PR review
│   ├── brooks-audit/             # Mode 2: Architecture audit
│   ├── brooks-debt/              # Mode 3: Tech debt assessment
│   ├── brooks-test/              # Mode 4: Test quality review
│   ├── brooks-health/            # Mode 5: Health dashboard
│   └── brooks-sweep/             # Mode 6: Full sweep & auto-fix
├── scripts/
│   ├── assemble-prompt.mjs       # Concatenates framework files into system prompt per mode
│   ├── ci-review.mjs             # CI entry point — git diff → Anthropic SDK → JSON report
│   ├── run-evals-live.mjs        # Live eval runner against Claude API
│   ├── eval-utils.mjs            # Classification: Iron Law check, risk code extraction
│   ├── validate-repo.mjs         # Repo structure validator
│   ├── validate-repo.test.mjs    # Tests for validation
│   ├── history.mjs               # .brooks-lint-history.json trend tracking
│   ├── cli-utils.mjs             # CLI argument parsing
│   ├── frontmatter.mjs           # YAML frontmatter utilities
│   ├── bump-version.mjs          # Version bumper
│   └── install.sh                # Multi-platform installer (11 platforms)
├── evals/evals.json              # 47 eval scenarios covering all 12 risk dimensions
├── .claude/agents/               # Claude Code subagent definitions
│   ├── trigger-boundary-auditor.md
│   ├── release-manager.md
│   ├── consistency-qa.md
│   ├── eval-curator.md
│   └── skill-author.md
├── .claude/skills/               # Claude Code internal skills
│   ├── brooks-harness/SKILL.md
│   ├── release/SKILL.md
│   └── new-skill/SKILL.md
└── .github/
    ├── actions/brooks-lint/action.yml  # GitHub Action for CI integration
    └── workflows/
```

### Prompt assembly pipeline

`scripts/assemble-prompt.mjs` (50 lines) is the core mechanism. Given a mode string, it concatenates:

1. `skills/_shared/common.md` — always loaded (Iron Law, config, report template)
2. `skills/_shared/source-coverage.md` — always loaded (book coverage with false-positive guards)
3. Risk definitions based on mode:
   - "test" → `test-decay-risks.md` only
   - "health"/"sweep" → both `decay-risks.md` and `test-decay-risks.md`
   - everything else → `decay-risks.md` only
4. Mode-specific guide from the mode's directory

The assembled system prompt is fed to the Anthropic SDK by both `ci-review.mjs` (CI mode) and `run-evals-live.mjs` (eval mode). The session-start hook tells Claude to use the Skill tool, which loads the SKILL.md files, which in turn instruct Claude to read the shared framework files.

### The six decay risks (production code)

Defined in `skills/_shared/decay-risks.md` (~295 lines). Each risk has:
- Diagnostic question
- 8-10 observable symptoms with per-symptom book citations
- Severity guide (Critical/Warning/Suggestion with numeric thresholds)
- "What Not to Flag" section (false-positive guards)

| Risk | Diagnostic Question | Key Books |
|------|---------------------|-----------|
| R1: Cognitive Overload | How much mental effort to understand? | Code Complete, Refactoring, DDD, Philosophy of SD |
| R2: Change Propagation | How many unrelated things break on one change? | Refactoring, Clean Architecture, Pragmatic, SE@Google, Brooks |
| R3: Knowledge Duplication | Same decision in multiple places? | Pragmatic, Refactoring, DDD |
| R4: Accidental Complexity | Code more complex than the problem? | Refactoring, Code Complete, Brooks, Philosophy of SD |
| R5: Dependency Disorder | Dependencies flow consistently? | Clean Architecture, Brooks, Pragmatic, SE@Google |
| R6: Domain Model Distortion | Code faithfully represents the domain? | DDD, Refactoring |

### The six test decay risks

| Risk | Diagnostic Question | Key Books |
|------|---------------------|-----------|
| T1: Test Obscurity | How hard to understand what this test verifies? | xUnit Test Patterns, Art of Unit Testing |
| T2: Test Brittleness | Do tests break on behavior-preserving refactors? | xUnit Test Patterns, Art of Unit Testing, Pragmatic |
| T3: Test Duplication | Same test scenario in multiple places? | xUnit Test Patterns, Pragmatic |
| T4: Mock Abuse | Test more complex than the behavior it tests? | Art of Unit Testing, xUnit Test Patterns, WELC |
| T5: Coverage Illusion | Suite actually protects against failures that matter? | WELC, How Google Tests Software, Art of Unit Testing |
| T6: Architecture Mismatch | Suite structure reflects actual risk profile? | How Google Tests Software, WELC, xUnit Test Patterns |

### The Iron Law

Enforced in `common.md`:

```
NEVER suggest fixes before completing risk diagnosis.
EVERY finding must follow: Symptom → Source → Consequence → Remedy.
```

### Health Score algorithm

```
Base: 100
-15 per 🔴 Critical
-5 per 🟡 Warning
-1 per 🟢 Suggestion
Floor: 0
```

### Source coverage discipline

`source-coverage.md` (~249 lines) encodes "do not over-flag" rules per book. For example:
- "Large systems are not automatically second systems" (Brooks)
- "CRUD-heavy workflows may legitimately use transaction scripts" (Evans)
- "A stable public API is not a liability if it is intentionally supported" (Winters)
- "A composition root wiring concrete dependencies is not a DIP violation by itself" (Martin)

### Config system

`.brooks-lint.yaml` supports: `disable` (skip risk codes), `severity` (override tiers), `ignore` (glob patterns), `focus` (only these risks), `suppress` (dismissed findings with optional expiry), `custom_risks` (Cx codes for project-specific risks).

### Eval system

`evals/evals.json` contains 47 scenarios across all 12 risk dimensions. Each has per-language prompts (Python, TypeScript, Go, Java), expected risk codes, and classification rules. The `eval-utils.mjs` classifier checks for:
1. Iron Law format (Symptom/Source/Consequence/Remedy)
2. Correct risk code identification
3. Health Score presence
4. False-positive avoidance (`no_risk_codes` scenarios)

### Mode 6: Full sweep pipeline

The most ambitious feature (`sweep-guide.md`, ~265 lines):
1. Pre-flight consent gate
2. Sequential four-dimension scan (review → test → debt → audit)
3. Fix classification: Safe (single-file), Extended-Safe (multi-file + tests), Residual (needs human)
4. Post-fix test verification with rollback on failure
5. Iteration loop: re-scan touched files + dependents, cap at 3 rounds for non-critical
6. 3-retry retirement: findings that fail fix 3 times go to `unresolvable`

## Key Techniques

1. **Prompt-as-code architecture**: The entire product is markdown files that become LLM system prompts. No parsing, no AST, no static analysis — the LLM does all the reasoning. This is both the biggest strength (works with any language, any codebase) and weakness (entirely dependent on LLM quality).

2. **Symptom-to-source mapping**: Each code smell is explicitly mapped to a specific book and principle. "Long Method" → Fowler Refactoring, "Magic numbers" → McConnell Code Complete Ch.12, etc. This makes findings traceable rather than vibes-based.

3. **False-positive guard clauses**: Every risk definition includes "What Not to Flag." A switch statement over a closed protocol is not missing polymorphism. A composition root is not a DIP violation. High fan-out in orchestration layers is not disorder. These prevent the common LLM failure mode of over-flagging.

4. **Severity calibration with numeric thresholds**: Instead of "bad code is bad," each risk has specific thresholds: function >50 lines = Critical, 20-50 = Warning. Nesting >5 = Critical, 4-5 = Warning. This provides consistency across runs.

5. **Health Score as a trend metric**: The 0-100 score with history tracking makes decay visible over time. Trend display (e.g., "85 → 82 (−3) over last 3 runs") shows direction, not just state.

6. **Multi-platform via standardized Agent Skills format**: One set of skill files works across 11 platforms because they all implement the Agent Skills spec. The `install.sh` script just copies files flat to the right directory.

7. **Eval-first development**: 47 scenarios with positive (should find this) and negative (should NOT flag this) cases, including clean code scenarios that test false-positive avoidance. The "no_risk_codes" flag in evals is particularly important — it tests that brooks-lint doesn't just flag everything.

8. **Triage workflow**: Interactive post-report triage with accept/dismiss/defer per finding. Dismissed findings are suppressed in `.brooks-lint.yaml` with optional expiry, so they resurface later.

## Design Decisions

**Optimized for: consistency and traceability.** The Iron Law ensures every finding has evidence (Symptom), authority (Source), impact (Consequence), and action (Remedy). The structured format means every output is comparable.

**Optimized for: false-positive avoidance.** The "What Not to Flag" sections and `source-coverage.md` "do not over-flag" rules are explicitly designed to prevent the LLM from being too eager. Clean code scenarios in the eval suite enforce this.

**Sacrificed: deep static analysis.** No AST parsing means it can't detect things linters find (unused variables, type errors). It's designed to complement linters, not replace them.

**Sacrificed: code-level fixes.** The --fix mode produces enhanced remedy descriptions, not code diffs. The sweep mode changes files but the core review modes are diagnosis-only.

**Pure prompt product**: The entire value is in the prompt engineering — which books to cite, how to structure findings, what symptoms to look for, what NOT to flag. This is both elegant (works with any LLM that reads markdown) and fragile (prompt quality IS product quality).

## Comparison Notes

**vs. ESLint/Pylint**: Linters catch syntax, style, and type errors deterministically. brooks-lint catches architectural drift, knowledge duplication, and domain model distortion — things linters can't detect because they require understanding intent.

**vs. OpenCodeReview**: OCR is a Go binary with deterministic file filtering, per-file goroutines, and a comment filter pass. brooks-lint is pure prompt engineering with no compiled code logic. OCR optimizes for coverage; brooks-lint optimizes for depth and traceability.

**vs. plain Claude**: The benchmark claims 94% pass rate for brooks-lint vs 16% for plain Claude on structured finding format, book citations, severity labels, and health scores. The gap is consistency, not capability.

**vs. CodeRabbit/AI PR Reviewer**: SaaS with dashboards. brooks-lint is self-hosted, CLI/skill-local, with project-level config in `.brooks-lint.yaml`.

## Dependencies

- `@anthropic-ai/sdk` (^0.52.0) — only devDependency, for CI and eval runners
- No runtime dependencies on npm packages, language-specific tooling, or external services
- The plugin itself needs no installation beyond copying markdown files
