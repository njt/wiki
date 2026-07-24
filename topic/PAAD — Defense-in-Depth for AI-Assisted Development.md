# PAAD — Defense-in-Depth for AI-Assisted Development

A Claude Code plugin suite by Curtis "Ovid" Poe that adds defense-in-depth safeguards to AI-assisted development: eight skills that critically review specs, verify plan-spec alignment, audit architecture, review code with multiple specialists, and enforce TDD guardrails on small changes. PAAD composes with existing AI coding tools — it doesn't replace them, it adds the missing layers of scrutiny that AI assistants don't reliably provide on their own.

---

## Architecture

PAAD is a Claude Code **plugin marketplace** (`plugins/paad/.claude-plugin/plugin.json`, v1.11.0) distributing eight skills as `SKILL.md` files. Each skill is a self-contained document with YAML frontmatter, phased workflow instructions, agent prompt templates, a Graphviz digraph encoding decision points, and an output template. Skills are invoked as `/paad:<name>` in Claude Code; the same `SKILL.md` files are ported for Cursor, Kiro, and Antigravity via `scripts/convert_skills.py`.

**The specialist + verifier pattern** is the architectural core of the "agentic-" skills (`agentic-architecture`, `agentic-review`, `agentic-a11y`): 5+ specialists analyze in parallel (each with a distinct lens and assigned flaw taxonomy), then a single adversarial verifier reads the actual code at each reported file:line, drops false positives, deduplicates cross-specialist findings, and assigns severity. Only verified findings reach the report.

**The defense-in-depth pipeline** chains skills across the SDLC: `pushback` (spec critique) → `alignment` (plan-spec alignment + TDD rewrite) → implementation → `agentic-review` (pre-merge bug hunting). Each gate catches problems before they become expensive.

The project auto-validates itself via `make test`, which checks version sync between `marketplace.json` and `plugin.json`, digraph presence, help/README coverage, and frontmatter consistency.

## Key Techniques

**Digraphs as executable safety gates.** Every skill except `help` includes a Graphviz `dot` block (`plugins/paad/skills/pushback/SKILL.md:12-44`) mapping decision points, stop conditions, and branching paths. The `make check-digraphs` target enforces this mechanically. The digraphs exist to prevent the LLM from skipping safety gates — they're not documentation, they're constraints.

**Adversarial verification with a skeptical prompt.** The verifier in every multi-agent skill is explicitly instructed to be adversarial: "Be skeptical — reject anything you cannot confirm by reading the code." `agentic-architecture/SKILL.md:118` adds: "file size alone doesn't make a god object, and many imports don't necessarily mean tight coupling. Check git history for context." This is a known-effective pattern for reducing LLM hallucination in analysis tasks.

**Source control reality checks before content analysis.** Both `pushback` and `alignment` scan `git log --oneline -50 --since="2 weeks ago"` before analyzing documents. If the spec assumes a table that was just renamed or an API that was just removed, that showstopper is presented *first*, before any other issue. This catches the most expensive category of spec error — assumptions invalidated by recent work — before any analysis budget is spent.

**34-flaw, 14-strength taxonomy with coverage checklist.** `agentic-architecture` provides a comprehensive catalog (`agentic-architecture/SKILL.md:199-254`) from "global mutable state" through "inconsistent error/logging conventions." Every specialist is assigned specific flaw types and strength categories. The output includes a coverage checklist table ensuring every one of 48 categories is explicitly marked Observed/Not observed/Not applicable — preventing the blind spots that single-reviewer analysis produces.

**Multi-session state via inline status annotations.** `fix-architecture` writes fix status directly into the architecture report as inline fields (`fix-architecture/SKILL.md:216-229`): `Status: Fixed`, `Status reason: ...`, `Status date: ...`, `Status commit: ...`. This turns the Markdown report into a state machine that survives Claude Code session boundaries — a pragmatic solution to the fundamental statelessness problem.

**Platform-adaptive accessibility analysis.** `agentic-a11y` detects the project platform (web, iOS, Android, React Native, Flutter, desktop, CLI, game) and adapts every specialist's instructions: the Screen Reader specialist gets ARIA/semantic HTML checks for web but `contentDescription`/`semantics{}` checks for Android and `Semantics` widget checks for Flutter. CLI tools get `--no-color` and structured-output checks. WCAG 2.2 AA is applied via WCAG2ICT for non-web platforms (`agentic-a11y/SKILL.md:12`).

**TDD format as structured output, not just advice.** `alignment` rewrites every task into a specific red/green/refactor template (`alignment/SKILL.md:232-253`) with enumerated expected failure modes: "If it passes unexpectedly: what that would mean." This makes the TDD process harder to skip because the cognitive work has already been done.

## Design Decisions

**Tokens over speed.** PAAD explicitly optimizes for catching mistakes early rather than minimizing token consumption. Multi-agent parallel dispatch inherently uses more tokens than a single-pass workflow. The trade-off is justified by the compounding cost of errors caught late: a spec flaw caught by `pushback` costs a conversation; an architectural flaw caught in production costs a rewrite.

**Sequential architecture fixes, not parallel.** `fix-architecture` works one flaw at a time, explicitly rejecting parallel execution: "merging multiple structural refactors back together is a reliable way to introduce new bugs" (`fix-architecture/SKILL.md:256`). This is counter-intuitive in an agentic system that otherwise celebrates parallelism, but it's correct: structural fixes have cascading effects that can only be discovered sequentially.

**Diagnosis-only architecture analysis.** `agentic-architecture` refuses to propose fixes — the skill spec says "Do NOT propose fixes. This is diagnosis only" (`agentic-architecture/SKILL.md:10`). Fixes are a separate skill (`fix-architecture`). This separation prevents the common LLM failure mode of jumping to solutions before understanding the problem, and allows re-running diagnosis after fixes to get a fresh baseline.

**One question at a time.** `fix-architecture`'s developer conversation phase is explicitly sequential: "One question per message. Ask, wait for the answer, then ask the next. Do not combine multiple questions into one message — it is frustrating and overwhelming" (`fix-architecture/SKILL.md:70`). This is a deliberate UX constraint on LLM behavior that prevents information-dump interactions.

**Safety-net tests before any fixes.** `fix-architecture` mandates: "ALL safety-net tests must be written and committed before ANY fixes are applied. No exceptions" (`fix-architecture/SKILL.md:129`). Even for a single fix, all tests are written first, committed together, and only then does the fix loop begin. This prevents the common failure mode where tests written alongside fixes are contaminated by fix assumptions.

**Human-in-the-loop at every consequential decision.** Every skill that makes changes requires developer approval: fix-architecture asks for explicit go-ahead before touching code; pushback presents issues one at a time and respects "good enough"; makefile refuses to modify existing targets without permission. The system is designed to keep humans in control of what matters while automating what doesn't.

## Comparison Notes

PAAD's approach contrasts with several related systems in the wiki:

- **[[Cloudflare Security Audit Skill]]** uses the same parallel-specialists + verification pattern but scoped to security vulnerabilities, while PAAD applies it to architecture, code review, and accessibility. Both share the adversarial-verifier insight.

- **[[SDDW (Spec-Driven Development Workflow)]]** covers the same spec→plan→implement pipeline but through modular commands that execute the workflow, while PAAD takes a critique-first approach: each stage is about finding problems before proceeding. SDDW builds; PAAD questions.

- **[[Matt Pocock — Grill Me, Then Go AFK]]** shares `pushback`'s pre-implementation critique philosophy as a conversational check, but PAAD's pushback is more structured: 6 analysis categories with severity ordering, source control reality check, and scope shape analysis.

- **[[Aviator Verify]]** shares `agentic-review`'s pre-merge verification intent but is a commercial product with running-code evidence, while PAAD's review is a multi-agent prompt pipeline — cheaper and more flexible but without the live-code verification Aviator provides.

- **[[Agentic Code Review]]** (Addy Osmani's field guide) describes the human-organizational challenge of AI code review (861% churn, 441% longer reviews), which `agentic-review` directly addresses by providing a structured, multi-specialist review that surfaces more than single-pass AI review.

- **[[Orchestrating AI Code Review at Scale]]** (Cloudflare's system) shares the 5+ specialist → coordinator pattern, but Cloudflare's is a production system with tiered models and circuit breakers, while PAAD encodes the same pattern as portable skill prompts.

- **[[Superpowers]]** (github.com/obra/superpowers) is explicitly listed as complementary — PAAD's README recommends using both together. Superpowers provides the workflow infrastructure (plan mode, subagent dispatch); PAAD provides the quality gates that plug into it.

- **[[The Agentic Product Standard v2.0]]** defines an 8-layer harness with Claude Code skills that operationalize it — PAAD is a concrete, field-tested implementation of several of those layers (specification quality, architectural governance, review discipline).

---

*Sources: [[raw/paad]]*
*Last updated: 2026-07-25*
