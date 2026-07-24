---
url: https://github.com/ovid/paad
title: PAAD — Defense-in-Depth for AI-Assisted Development
author: Curtis "Ovid" Poe
date_fetched: 2026-07-25
date_published: 2026-03-14
---

# PAAD — Defense-in-Depth for AI-Assisted Development

**Repository:** https://github.com/ovid/paad
**Author:** Curtis "Ovid" Poe
**License:** MIT
**Version:** 1.11.0
**Total skill LOC:** ~2,237 lines of SKILL.md across 8 skills

## What It Is

PAAD (Pushback, Alignment, Architecture, Discipline) is a Claude Code plugin marketplace and multi-platform skill suite that adds defense-in-depth safeguards to AI-assisted development. It's a system of AI agent skills — each a document of prompts, procedures, and decision graphs — designed to catch the failure modes AI coding assistants don't reliably catch on their own: weak specs, plan-spec drift, architectural decay, and skipped quality gates.

PAAD supports Claude Code natively and ships compatible SKILL.md files for Cursor, Kiro, and Antigravity. A `convert_skills.py` script strips Claude Code-specific paths and `paad:` command references to produce platform-neutral versions.

## Architecture

### Plugin distribution model

PAAD is structured as a Claude Code **plugin marketplace** hosting a single plugin also called `paad`:

```
paad/
├── .claude-plugin/marketplace.json    ← marketplace catalog (lists plugins)
├── plugins/paad/                       ← the paad plugin
│   ├── .claude-plugin/plugin.json      ← plugin manifest (v1.11.0)
│   └── skills/                         ← 8 SKILL.md files
│       ├── pushback/SKILL.md           ← spec critique
│       ├── alignment/SKILL.md           ← requirements-to-tasks alignment
│       ├── agentic-architecture/SKILL.md ← multi-agent architecture analysis
│       ├── fix-architecture/SKILL.md    ← guided architecture fix loop
│       ├── agentic-review/SKILL.md      ← multi-agent bug-hunting review
│       ├── agentic-a11y/SKILL.md        ← multi-platform accessibility audit
│       ├── vibe/SKILL.md                ← TDD-guarded quick fixes
│       ├── makefile/SKILL.md            ← Makefile generation
│       └── help/SKILL.md                ← help display
├── kiro_and_antigravity/skills/        ← ported versions for Kiro/Cursor/Antigravity
│   ├── .kiro/skills/                   ← Kiro-compatible (paths neutralized)
│   └── .agent/skills/                  ← Antigravity wrappers referencing .kiro files
├── scripts/convert_skills.py           ← strips Claude Code references for portability
└── docs/plans/                         ← design documents for skill creation
```

Invocation model: `/paad:<skill-name>` in Claude Code. Outside Claude Code, the SKILL.md files are auto-discovered by the hosting platform (Cursor, Kiro, Antigravity) and triggered by natural language.

### The skill format

Each skill is a single `SKILL.md` file with YAML frontmatter (`name`, `description`) followed by Markdown prose containing:

1. **Pre-flight check digraph** — A Graphviz `dot` block visualizing decision points and stop conditions. Required for every skill except `help`. The digraph is a safety mechanism: it ensures the LLM doesn't skip gates (e.g., branch protection, context window warnings, test infrastructure checks).
2. **Phased workflow** — Numbered phases with explicit instructions for what to read, what to run, and what to check at each step.
3. **Agent prompt templates** — For multi-agent skills, exact prompt text to dispatch to specialist subagents, including caveats ("steering files may be stale").
4. **Report templates** — Exact markdown structures for output files, with sections for findings, metadata, and coverage checklists.

### The specialist + verifier pattern

The core architectural pattern across the three "agentic-" skills (architecture, review, a11y) is:

1. **Phase 1: Reconnaissance** — Collect repo metadata, build file manifests, scan for steering files
2. **Phase 2: Parallel specialists** — Dispatch 5+ agents simultaneously, each with a distinct lens (e.g., Logic & Correctness, Error Handling, Security). Each specialist receives the file manifest, repo overview, and their specific flaw/strength taxonomy.
3. **Phase 3: Verification** — Dispatch a single verifier agent with ALL specialist findings. The verifier reads actual code at each referenced file:line, confirms the bug/barrier exists, drops false positives, deduplicates cross-specialist findings, and assigns severity.
4. **Phase 4: Report** — Write verified findings to a structured template with coverage checklists.

This pattern addresses a fundamental reliability problem: LLMs are good at generating plausible-sounding findings but bad at verifying them. The verifier acts as an adversarial filter.

### Build system

The `Makefile` provides six validation checks:

| Target | What it verifies |
|--------|-----------------|
| `check-versions` | marketplace.json ↔ plugin.json version sync |
| `check-digraphs` | every skill (except help) has a Graphviz digraph |
| `check-help` | every skill documented in paad:help overview |
| `check-readme` | every skill documented in README.md |
| `check-frontmatter` | SKILL.md name matches folder name, description present |
| `validate` | `claude plugin validate` on marketplace and plugin |

## Skill-by-Skill Analysis

### pushback (202 lines)

A spec critic that reviews requirements before implementation begins. Unique design choices:

- **Source control reality check first** — Scans `git log --oneline -50 --since="2 weeks ago"` before any spec analysis. If commits conflict with what the spec assumes (renamed tables, removed APIs, changed infrastructure), these showstoppers are presented before anything else.
- **Scope shape check** — Before analyzing individual requirements, checks feature cohesion (unrelated features bundled together) and spec size (8+ distinct features = suggest split). Cohesion runs before size because splitting unrelated features may resolve the size problem.
- **One issue at a time** — Issues are ranked by severity and presented one at a time. The user can say "good enough" at any point. This prevents overwhelming the user with a wall of findings and respects attention as a scarce resource.
- **6 analysis categories**: Contradictions, Feasibility (given actual codebase), Scope imbalance, Omissions, Ambiguity, Security concerns.

### alignment (255 lines)

Checks that intent documents (requirements, specs) and action documents (plans, tasks) are aligned, then rewrites all tasks in TDD format.

- **Document classification** — Classifies files as "intent" (what we want) or "action" (what we'll do) based on content analysis, not file name heuristics.
- **Three alignment checks**: Requirements coverage (every requirement has tasks?), Scope compliance (every task maps to a requirement?), Design alignment (if design docs exist).
- **Dependency-ordered presentation** — Missing requirements first (root causes), then design gaps, then missing/orphaned tasks last (symptoms). Fixing upstream issues often resolves downstream ones.
- **Mandatory TDD rewrite** — Once aligned, rewrites action items in red/green/refactor format with specific RED step details: "Write a test that asserts X. Expected failure: Y. If it passes unexpectedly: Z." This is not optional — it produces better implementations because the RED step surfaces unknown codebase issues.

### agentic-architecture (278 lines)

Multi-agent architecture analysis spanning 34 flaw types and 14 strength categories. Diagnosis only — does not propose fixes.

- **5 specialist agents**: Structure & Boundaries, Coupling & Dependencies, Integration & Data, Error Handling & Observability, Security & Code Quality.
- **34 flaw types**: A comprehensive taxonomy from "global mutable state" and "god object" to "inconsistent error/logging conventions across services." Each agent is assigned specific flaw types from this catalog.
- **14 strength categories**: Explicitly balanced — strengths are equally important as flaws because they tell teams what to protect.
- **Refactor history instruction**: Every agent prompt includes: "Before flagging a candidate flaw, use `git log --oneline` on the relevant files/directories to check whether the current code is the result of recent intentional work." Prevents false positives on freshly refactored code.
- **Coverage checklist**: Every one of the 34 flaw types and 14 strength categories appears in a table with Observed/Not observed/Not applicable status. Prevents blind spots.
- **No fixes**: Explicitly refuses to propose solutions. The report is structured questions to guide investigation, not answers.

### fix-architecture (286 lines)

Guided, iterative fixing of flaws from an agentic-architecture report. The most procedurally complex skill.

- **Sequential, not parallel** — Architecture fixes happen one at a time because fixing one structural flaw often resolves others. The design doc explicitly states: "merging multiple structural refactors back together is a reliable way to introduce new bugs."
- **Safety-net phase before any fixes** — ALL safety-net tests must be written and committed BEFORE ANY fixes are applied. "No exceptions — one refactor can break code another flaw's tests would have caught."
- **Developer conversation** — One question per message (never combined): solo vs team, auto-commit vs manual, flaw triage, plan confirmation. This is deliberate UX design to avoid overwhelming the developer.
- **7 fix statuses**: Fixed, Won't fix, Partially fixed, Skipped, Fixed (pre-existing), Attempted/reverted. Status is written inline into the report so the skill can resume across sessions.
- **Flaw dependency detection** — After each fix, checks whether it resolved other flaws (not just in current batch). Reports: "Fixing F-03 appears to have also resolved F-07 (low cohesion)."
- **Context-aware stopping** — Recommends pausing when context approaches limits, designed for multi-session continuation.

### agentic-review (194 lines)

Multi-agent bug-hunting code review. Pre-merge quality gate.

- **5 specialist agents**: Logic & Correctness, Error Handling & Edge Cases, Contract & Integration, Concurrency & State, Security.
- **Conditional Plan Alignment agent** — Dispatched only if design docs are found in `docs/plans/` or similar.
- **Error Handling specialist gets a parsing-specific instruction**: "When code parses external output using exact string matching, check whether realistic output variations — trailing punctuation, extra whitespace, mixed casing — would cause silent misclassification."
- **Contract & Integration specialist checks for logic duplication**: "Flag new code that reimplements logic already available in the codebase."
- **Scaling for large diffs**: 500+ lines → partition files across 2 instances of each specialist.
- **Test infrastructure parity check**: When the diff includes infrastructure files (schema migrations, CI configs), checks whether test-side counterparts exist. Catches mismatches where production infrastructure was updated but test infrastructure wasn't.

### agentic-a11y (392 lines — the largest skill)

Multi-platform accessibility audit supporting 8 platform types with platform-specific API checks per specialist.

- **5 core specialists**: Screen Reader & Assistive Tech, Visual & Color, Keyboard & Motor, Cognitive & Learning, Multimedia & Temporal.
- **Conditional Platform-Specific agent**: Dispatched when framework-specific pitfalls exist (React, Vue, SwiftUI, Jetpack Compose, Flutter, etc.).
- **Platform detection phase**: Scans file indicators to identify the project's platform(s) before dispatching specialists.
- **Per-specialist, per-platform checks**: Screen Reader specialist gets different instructions for Web (ARIA, semantic HTML), iOS (accessibilityLabel, accessibilityTraits), Android (contentDescription, semantics{}), React Native (accessibilityRole), Flutter (Semantics widget), CLI (structured parseable output, --no-color flag), and Games (text alternatives for UI elements, narration mode).
- **WCAG 2.2 AA baseline**: Applied via WCAG2ICT for non-web platforms. AAA flagged as bonus.
- **Platform-specific guidelines**: Apple HIG Accessibility, Material Design Accessibility, Xbox Accessibility Guidelines.
- **Verifier skepticism specific to platform defaults**: "Standard UIKit controls, Material components, and Flutter widgets have built-in a11y — only flag if misused, overridden, or missing."

### vibe (169 lines)

Safe vibe coding for small fixes with mandatory TDD guardrails.

- **Pre-flight checks before any code**: Test infrastructure exists? Scope (4+ files = warn)? Architecture smell (simple task requires too much work = investigate deeper)? Reusable components (search before building)?
- **RED step with tri-state outcome**: Test fails as expected (proceed), test passes unexpectedly (feature may already exist — stop), test fails in unexpected way (unknown issue — stop and discuss).
- **Mandatory REFACTOR**: "This is the step AI skips unless told to, and it's where real quality comes from."
- **Contextual follow-up suggestions**: Security-sensitive change → suggest agentic-review. UI change → suggest agentic-a11y. Harder than expected → suggest agentic-architecture.

### makefile (127 lines)

Creates/updates project Makefiles. Key rule: never modifies an existing target without explicit user approval. Detects stack from project files, enforces balanced test output (concise on success, detailed on failure), and forces one-shot mode for coverage tools to avoid watch-mode hangs.

### help (334 lines, mostly display text)

Shows overview or per-skill help. All help text is inline in the SKILL.md — the skill does not read files or run commands.

## Design Philosophy

### Honest about risk

"If a specification is weak, a plan is misaligned, an architectural decision is fragile, or a code change introduces problems, the goal is to surface that early and clearly."

### Defense-in-depth, not replacement

PAAD does not replace existing AI-assisted development tools; it complements them. The intellectual lineage is from traditional software engineering's layers of protection (specs, tests, code review, CI, QA, UAT), adapted for the AI era.

### Tokens over speed

"PAAD optimizes for better decisions and fewer avoidable mistakes, not minimum token consumption." Multi-agent parallel dispatch inherently uses more tokens than a single-pass workflow.

### Human-in-the-loop

Every consequential decision requires developer approval. Fix-architecture asks one question at a time and requires explicit go-ahead. Pushback presents issues one at a time and respects "good enough." The system is designed to keep humans in control of what matters.

## Cross-Platform Strategy

The `convert_skills.py` script (`scripts/convert_skills.py`) converts PAAD's Claude Code skills to Kiro and Antigravity format:

- **Kiro**: Strips "Arguments", "Input Resolution", "Pre-flight Checks", and "Document classification" headings (Kiro handles those differently). Neutralizes `paad/` paths to `.reviews/`. Removes lines containing `/paad:` references.
- **Antigravity**: Creates wrapper files that reference the Kiro skill file, with minimal frontmatter and a single instruction: "Please refer to that file for the full criteria."

The `makefile` and `help` skills are excluded from conversion as they're Claude Code-specific.

## What Makes This Non-Obvious

1. **Digraphs as executable safety constraints** — Most skills-as-prompts just list steps. PAAD uses Graphviz digraphs to encode decision points visually, making it harder for the LLM to rationalize skipping gates. The digraph requirement in CLAUDE.md is enforced by `make check-digraphs`.

2. **The verifier as adversarial filter** — Having 5 specialists find issues and then a single skeptical verifier kill false positives is a specific, replicable pattern. The verifier prompt is deliberately adversarial: "Be skeptical — reject anything you cannot confirm by reading the code."

3. **Status tracking as multi-session protocol** — Fix-architecture's inline status annotations turn the architecture report into a state machine that survives session boundaries. This is a pragmatic solution to the fundamental problem that Claude Code sessions are stateless.

4. **Scope shape before content analysis** — Pushback checks feature cohesion (are these unrelated things bundled?) before analyzing individual requirements. This catches structural document problems that content-level analysis misses.

5. **TDD format as output, not just process** — Alignment doesn't just recommend TDD; it rewrites all tasks into a specific red/green/refactor template with expected failure modes enumerated. This makes the TDD process harder to skip because it's already been done.

6. **CLI accessibility is real** — Agentic-a11y treats CLI tools as a first-class accessibility target with specific checks: `--no-color` flag, structured parseable output, text progress indicators instead of spinners, information not conveyed by color alone.

## Related Concepts

PAAD's approach parallels several other agentic development systems:

- **Cloudflare's Security Audit skill** uses the same "parallel agents → verification" pattern but for security specifically rather than general architecture
- **SDDW (Spec-Driven Development Workflow)** covers the same spec→plan→implement pipeline but does it through modular commands rather than defense-in-depth critique skills
- **Matt Pocock's "Grill Me" alignment skill** shares the same pre-implementation critique philosophy as pushback but is a single conversational check rather than a structured 6-category analysis
- **Superpowers** (github.com/obra/superpowers) is explicitly listed as complementary — PAAD adds guardrails that Superpowers doesn't address
- **Aviator Verify** shares the intent-based verification philosophy with agentic-review but is a commercial product rather than a prompt-suite skill
