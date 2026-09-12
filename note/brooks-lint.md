# brooks-lint

An AI code review tool delivered as a Claude Code plugin (v1.3.0, MIT) that diagnoses code against 12 decay risk dimensions synthesized from 12 classic software engineering books — Brooks, McConnell, Fowler, Martin, Hunt & Thomas, Evans, Ousterhout, Winters et al., Meszaros, Osherove, Feathers, and Google Testing. Every finding follows an Iron Law (Symptom → Source → Consequence → Remedy) with book citations and severity labels. The architecture is pure prompt engineering: the product is markdown files that become LLM system prompts, with no parsing, no AST, no static analysis. Six analysis modes (PR review, architecture audit, tech debt, test quality, health dashboard, full sweep) work across 11 platforms via standardized Agent Skills format.

---

## Architecture

brooks-lint is a **pure prompt product**. The "source code" is a hierarchy of markdown files organized into a shared framework + per-mode guides. A lightweight Node.js scaffold (`scripts/assemble-prompt.mjs`, 50 lines) concatenates these into a system prompt for the chosen mode, which is then fed to the Anthropic SDK by either the CI runner or eval runner.

### File structure

```
skills/
├── _shared/                  # Loaded into every mode's system prompt
│   ├── common.md             # Iron Law, config, report template, Health Score
│   ├── decay-risks.md        # R1-R6 definitions with symptoms, sources, severity, guards
│   ├── test-decay-risks.md   # T1-T6 test decay definitions
│   ├── source-coverage.md    # Per-book "do not over-flag" rules (false-positive discipline)
│   └── remedy-guide.md       # --fix mode: enhanced remedy rules
├── brooks-review/            # Mode 1: PR Review
│   ├── SKILL.md              # Triggers the Skill tool
│   └── pr-review-guide.md    # 7-step analysis process
├── brooks-audit/             # Mode 2: Architecture Audit (+ Mermaid dep graph)
├── brooks-debt/              # Mode 3: Tech Debt (Pain × Spread scoring)
├── brooks-test/              # Mode 4: Test Quality Review
├── brooks-health/            # Mode 5: Composite Health Dashboard
└── brooks-sweep/             # Mode 6: Full sweep with auto-fix pipeline
```

### Prompt assembly pipeline

```
assemble-prompt.mjs:
  1. common.md          → always
  2. source-coverage.md → always
  3. decay-risks.md and/or test-decay-risks.md → based on mode
  4. Mode-specific guide → from mode directory
  → System prompt for Anthropic SDK
```

The session-start hook (`hooks/session-start.mjs`) auto-installs slash commands into `~/.claude/commands/` and injects context telling Claude which skills are available. Claude then uses the Skill tool to load the SKILL.md files, which instruct it to read the shared framework.

### The twelve decay risks

**Production code** (`decay-risks.md`, 295 lines):

| Risk | Diagnostic Question | Key Sources |
|------|---------------------|-------------|
| R1: Cognitive Overload | How much mental effort to understand? | Code Complete, Refactoring, DDD, Philosophy of SD |
| R2: Change Propagation | How many unrelated things break on one change? | Refactoring, Clean Architecture, Pragmatic, SE@Google, Brooks |
| R3: Knowledge Duplication | Same decision in multiple places? | Pragmatic, Refactoring, DDD |
| R4: Accidental Complexity | Code more complex than the problem? | Refactoring, Code Complete, Brooks, Philosophy of SD |
| R5: Dependency Disorder | Dependencies flow consistently? | Clean Architecture, Brooks, Pragmatic, SE@Google |
| R6: Domain Model Distortion | Code faithfully represents the domain? | DDD, Refactoring |

**Test suite** (`test-decay-risks.md`, 247 lines):

| Risk | Diagnostic Question |
|------|---------------------|
| T1: Test Obscurity | How hard to understand what this test verifies? |
| T2: Test Brittleness | Do tests break on behavior-preserving refactors? |
| T3: Test Duplication | Same test scenario in multiple places? |
| T4: Mock Abuse | Test more complex than behavior it tests? |
| T5: Coverage Illusion | Suite protects against failures that matter? |
| T6: Architecture Mismatch | Suite structure reflects actual risk profile? |

Each risk has 8–10 observable symptoms, per-symptom book citations, a severity guide with numeric thresholds, and a "What Not to Flag" section for false-positive guards.

### Health Score

```
Base: 100
-15 per 🔴 Critical, -5 per 🟡 Warning, -1 per 🟢 Suggestion
Floor: 0
```

History is tracked in `.brooks-lint-history.json` with trend display across runs.

### The Iron Law

```
NEVER suggest fixes before completing risk diagnosis.
EVERY finding must follow: Symptom → Source → Consequence → Remedy.
```

Enforced by the eval classifier (`eval-utils.mjs`) which checks for all four fields.

---

## Key Techniques

### Symptom-to-source matrix

Every observable code smell is explicitly mapped to a specific book and principle: "Long Method" → Fowler Refactoring, "Magic numbers" → McConnell Code Complete Ch.12, "Hyrum's Law violations" → Winters et al. SE@Google Ch.1. This makes findings traceable rather than vibes-based — each claim carries an authority that a human reviewer can verify.

### "What Not to Flag" guard clauses

The most important innovation. Each risk definition ends with explicit false-positive guards: a switch over a closed protocol is not missing polymorphism, a composition root is not a DIP violation, high fan-out in orchestration layers is not disorder, CRUD workflows may legitimately use transaction scripts. The eval suite tests these with `no_risk_codes: true` scenarios — code that looks dubious but shouldn't be flagged.

### Source coverage discipline

`source-coverage.md` (249 lines) encodes per-book "do not over-flag" rules with three categories: "Encoded today" (what's active), "Do not ignore" (subtle signals), "Do not over-flag" (acceptable tradeoffs). This prevents the common LLM failure mode of citing books for everything.

### Eval-first development with false-positive testing

47 eval scenarios in `evals/evals.json` test both positive detection (should find R1 Critical in deeply nested code) and negative detection (should NOT flag clean code as R5 just because it has a composition root). The `classify()` function in `eval-utils.mjs` distinguishes "pass" from "false-positive-pass" — critical for a tool whose failure mode is being too eager.

### Fix classification in sweep mode

The full sweep (`brooks-sweep`) classifies every finding: Safe (single-file mechanical fix), Extended-Safe (multi-file with test coverage), Residual (needs human judgment). Safe fixes are applied directly; Extended-Safe are applied then verified with the project's test command; Residual findings become a human-readable report. A 3-retry budget retirees findings that fail verification, and non-critical rounds cap at 3 iterations.

### Project-level config

`.brooks-lint.yaml` supports disabling risk codes, overriding severity tiers, ignoring file globs, focusing on specific risks, suppressing dismissed findings (with optional expiry), and defining custom Cx risk codes. Config errors are reported in the review output rather than crashing.

### Trend tracking

`.brooks-lint-history.json` accumulates `{date, mode, score, findings, scope}` records. After each run, the report shows trend: "85 → 82 (−3) over last 3 runs." First-run shows "First run — no trend data." This turns code quality from a snapshot into a trajectory.

---

## Design Decisions

**Optimized for: consistency and traceability.** The Iron Law guarantees every finding has evidence, authority, impact, and action. The benchmark claims 94% pass rate for brooks-lint vs 16% for plain Claude on structured finding format — the gap is consistency, not capability.

**Optimized for: false-positive avoidance.** The "What Not to Flag" sections and eval false-positive tests are the differentiator. The tool knows when to shut up, which is harder than knowing when to flag.

**Sacrificed: static analysis.** No AST parsing, no type checking, no control flow analysis. Designed to complement linters, not replace them. The README is explicit: "brooks-lint doesn't replace your linter. It catches what linters can't."

**Sacrificed: code-level fixes.** The --fix mode produces enhanced remedy descriptions with target/action/rationale, not code diffs. The sweep mode edits files but even then, most findings are Residual (human-needed).

**Pure prompt product.** The entire value is in prompt engineering — which books, how to structure findings, what symptoms, what guards. This is both elegant (works with any LLM, any language) and fragile (prompt quality IS product quality; model behavior changes break the tool).

**Multi-platform bet.** One set of skill files works across 11 platforms because they all implement the Agent Skills spec. The install script just copies files flat to the right directory. This is a bet on the Agent Skills standard being durable.

---

## Comparison Notes

**vs. [[OpenCodeReview]]**: OCR is a Go binary with deterministic file filtering, per-file goroutines, and a comment filter pass. brooks-lint is pure prompt engineering with no compiled code logic. OCR optimizes for coverage across many files; brooks-lint optimizes for depth and traceability within fewer files.

**vs. ESLint/Pylint**: Linters catch syntax, style, and type errors deterministically. brooks-lint catches architectural drift, knowledge duplication, and domain model distortion — problems linters can't detect because they require understanding intent.

**vs. plain Claude Code review**: brooks-lint is a structured prompt wrapped in a skill. Without it, Claude reviews are unstructured and inconsistent. The value is the framework, not the model.

**vs. [[Guardrails and Feedback Loops]]**: brooks-lint embodies the "linters beat prompts" principle but inverts it — it uses prompts to simulate what a linter for architectural concerns would look like. The Iron Law and severity guides are deterministic constraints on LLM output.

**vs. [[StrongDM Factory Techniques]]**: SMF validates code by harness (does it build/pass/deploy?). brooks-lint validates by deep analysis (does the logic make sense?). Complementary approaches — SMF for correctness, brooks-lint for maintainability.

**vs. [[FrontierCode]]**: FrontierCode measures whether a PR would be accepted by a human tech lead. brooks-lint diagnoses why it might not be, with book citations. FrontierCode is the benchmark; brooks-lint is the feedback.

**vs. [[A New Era for Software Testing]]**: antirez's agentic QA uses markdown checklists and commit inspection for testing. brooks-lint's test mode (T1-T6) is a more structured, book-grounded alternative to ad-hoc test review.

**vs. CodeRabbit / AI PR Reviewer**: SaaS with dashboards and team management. brooks-lint is self-hosted, CLI/skill-local, with project config in `.brooks-lint.yaml`. No dashboard, no team features, no SaaS — just the analysis.

---

Tags: #tool #project #code-review #quality #agents

*Sources: [[summary/brooks-lint]]*
*Last updated: 2026-06-15*
