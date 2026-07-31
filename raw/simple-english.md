---
url: https://github.com/AminBlg/SimpleEnglish
title: SimpleEnglish — Write Like an Aerospace Manual
author: AminBlg (Amin Baig)
date_fetched: 2026-08-01
date_published: 2026-07
---

# SimpleEnglish — Full Analysis

## What It Is

A Claude Code skill (custom instruction set) that forces LLMs to write technical documentation in ASD-STE100 Simplified Technical English — the controlled language aerospace and defense manufacturers have used since 1983 for maintenance manuals. The key insight: STE's rules (max 20/25-word sentences, one word per meaning, simple tenses, active voice, no hedging, condition before command) happen to be a near-perfect negative of every AI writing tell. Longer sentences, synonym rotation, hedges, decorative clauses — those are exactly what LLMs default to. So applying a 40-year-old aerospace standard produces text that is both technically rigorous and free of "AI slop."

It's not a library, not a tool, not an API. It's a single 327-line SKILL.md file — pure prompt engineering — that works in Claude Code, Cursor, VS Code Copilot, OpenAI Codex, Gemini CLI, and ~25 other harnesses that speak the Agent Skills open standard. MIT licensed, zero dependencies.

## Architecture

The entire project is a prompt engineering artifact, not executable code. The architecture is:

### The Skill (SKILL.md) — 327 lines

This is the engine. It's structured as a procedure the model follows step by step:

1. **Mode selection**: Pragmatic (default — structural rules, domain words stay) or Strict (full vocabulary discipline, tells user about the official dictionary).
2. **Classification**: Before applying any rules, classify text as procedural (instructions: imperative, 20-word limit) or descriptive (explanations: simple tenses, 25-word limit). Every other rule depends on this classification.
3. **Vocabulary constraint**: Pick ONE verb for check/verify/confirm/validate BEFORE drafting. This prevents the synonym rotation LLMs default to.
4. **Rule catalog**: 53 rules in 9 sections, paraphrased from ASD-STE100 Issue 9 (2025-01-15) with software-industry examples. The rules cover words (1.1-1.14), multi-word nouns (2.1-2.2), verbs (3.1-3.7), sentences (4.1-4.5), procedural writing (5.1-5.5), descriptive writing (6.1-6.6), safety instructions (7.1-7.3), punctuation and word count (8.1-8.7), and writing practices (9.1-9.4 plus GR-1 to GR-8).
5. **Self-check**: Four mechanical checks the model runs on its own output before delivery — count words in longest sentences, search for banned patterns (contractions, "has been", "should", trailing "-ing" clauses, semicolons), verify condition placement, and catch synonym rotation.
6. **Untouchables exemption**: Code blocks, identifiers, CLI commands, file paths, quoted error messages, and product names are explicitly exempt from STE rules. Each counts as one word toward sentence limits (Rule 8.6). This is the key design decision that makes it practical for software docs.

### Supporting files

- `prompts/system-prompt.md`: Standalone version for harnesses without SKILL.md support. Including a ~60-token word-budget version for tight system prompts.
- `skills/simple-english/references/checklist.md`: Full verification pass with searchable patterns for audits and "check mode."
- `skills/simple-english/references/use-cases.md`: Long-form adaptations for error messages, runbooks, incident reports, release notes, agent instructions (prompts, AGENTS.md), translation prep, UI copy, commit messages, and support macros. Each use case names the mode and the adaptations.
- `examples/before-after.md`: Real unedited Claude outputs vs. skill-applied rewrites across README intros, troubleshooting docs, error messages, incident reports, and release notes.

### Evaluation infrastructure

- `evals/ste_lint.py` (132 lines): Deterministic regex-based STE violation counter. Catches 10 categories: sentence over limit, contractions, banned modals (should/would/may/might/could), perfect tenses (has been/have been), trailing "-ing" clauses (", making", ", allowing"), semicolons, Latin abbreviations (e.g., i.e., etc.), slop words (simply, robust, leverage, comprehensive), trailing conditions (if/when not at sentence start), and synonym rotation (check/verify/confirm rotation, config/settings rotation). Has a self-test fixture that asserts every violation category fires on a slop sample and zero violations on a clean sample.

- `evals/run_bench.py` (199 lines): Benchmark harness that runs headless `claude -p` calls across models x conditions x scenarios (96 total generations). Resumable — skips existing raw result files. Can do `--smoke` (1 model × 2 scenarios), `--report-only` (rebuild RESULTS.md from raw), or `--judge` (blind pairwise judge pass that scores both orderings to control for position bias).

- `evals/scenarios.json`: 8 standardized prompts covering README intro, getting-started docs, troubleshooting, error messages, incident reports, release notes, runbook steps, and architecture overviews. Each tagged procedural/descriptive for correct linting.

- `evals/pressure-tests.md`: TDD-style test scenarios with recorded baseline failures and pass criteria. Documents that baseline agents without the skill wrote 40-word sentences, invented rule numbers (one confidently cited "Rule 3.1: short sentences"; real Rule 3.1 is verb forms), and used passive voice. The skill was written to close each recorded failure, then re-tested until agents pass.

## Key Techniques

### Classification-first approach
The most important structural decision: the model must classify text as procedural or descriptive BEFORE applying any rules. Different sentence limits apply (20 vs 25 words), different verb forms (imperative vs simple tenses), and different structural rules (one instruction per sentence vs one topic per paragraph). Without this, a procedure that references an explanation gets the wrong limits. The skill's self-check and the linter both enforce this boundary.

### Pre-writing vocabulary constraint
Instead of catching synonym rotation after the fact, the skill makes vocabulary choice a deliberate pre-writing step: "Pick ONE verb for the check/verify/confirm/validate concept and ONE noun for config/settings." This exploits the model's ability to follow constraints it stated itself — a form of self-commitment. The self-check then scans for violations mechanically.

### Deterministic self-check as quality gate
The four mechanical checks (word count, banned-pattern grep, condition placement, synonym scan) are explicitly marked "not optional." This is prompt engineering as quality control: the model catches its own violations before the user ever sees them. It's a lightweight version of the compilation/pass pipeline that makes compilers effective — check first, deliver only if clean.

### "Untouchables" as practical boundary
STE was designed for aircraft maintenance manuals, not software docs. The "untouchables" exemption (code blocks, identifiers, CLI commands, quoted errors, product names, numbers with units) is the pragmatic bridge. Rule 8.6 — that backticked text counts as one word — prevents long identifiers from blowing the sentence budget. Without this exemption, STE would be unusable for software documentation.

### GR rules integration
The skill incorporates the General Recommendations (GR-1 through GR-8) from the official standard — obscure rules that even people familiar with STE often miss. GR-6 (no Latin abbreviations in running text), GR-8 (avoid possessive apostrophe when unsure — non-native readers find it hard), and GR-4 (prefer "this + noun" over bare "this") are exactly the kind of detail work that makes the difference between "sounds like STE" and "is STE."

### TDD-style prompt engineering
The pressure-tests.md file documents a genuinely test-driven development process: run baseline models without the skill, record what they get wrong (40-word sentences, invented rule numbers, passive voice), write rules to close each gap, re-test. The skill's self-check step was revised after recorded failures: verb choice was moved from a post-hoc check to a pre-writing step, and trailing conditions were added to the search list after the first run missed them.

### Regex linter as comparable measurement
The ste_lint.py linter explicitly acknowledges its ceiling (text undercounts because it can't detect passive voice or part-of-speech errors) but argues for comparability: it counts the same way for both conditions, so the before/after comparison is fair even where absolute numbers are low. This is a sophisticated trade-off — trading completeness for reproducibility, in the spirit of "all models are wrong; some are useful."

### Blind pairwise judging
The benchmark's `--judge` mode scores outputs in both possible orderings (baseline-first, skill-first) to control for position bias. Each pair gets two scores from independent Claude calls. This is statistical rigor applied to a domain (LLM evaluation) where subjective judgment is common but controlled experiments are rare.

### The "modal ladder"
A compact decision table for replacing banned modals: should (requirement) → must; should (recommendation) → delete or state as fact; may/might/could → can; would (hypothetical) → restructure as conditional. This is a pattern that transfers beyond STE — it's a general-purpose technique for removing hedging from any procedural text.

### Slop-to-simple substitution table
A 30+ entry table mapping AI-overused words to plain replacements. This is independent of the STE dictionary (which is copyrighted and not reproduced) — it's an empirical catalog of LLM writing tics with surgical replacements. The instruction "if the word carries no fact, delete it instead of replacing it" is the deeper insight: most filler has no replacement; it has a deletion.

## Design Decisions

### Optimized for: correctness and clarity over style
The skill explicitly refuses to handle marketing copy, blog voice, or brand writing. "STE deletes persuasion on purpose." This is a deliberate scope constraint: it does one thing (make technical text unambiguous) and refuses everything else. The limits section in SKILL.md even instructs the model to say "no" when asked to apply STE to marketing.

### Sacrificed: completeness for portability and legal safety
The official STE dictionary (~900 approved words, ~1,200 banned words with alternatives) is copyrighted by ASD. The skill paraphrases the rules and provides substitution patterns but does not reproduce dictionary content. This means it can't enforce word-level compliance in strict mode — but it can be distributed freely under MIT without licensing the standard. The two-mode design (pragmatic vs strict) is the architectural expression of this trade-off.

### Sacrificed: deep NLP for deterministic regex
The ste_lint.py linter is a regex pass, not a grammar parser. It can't detect passive voice or verify parts of speech. The authors are transparent about this ("honest caveat list" in RESULTS.md) and argue the comparison is fair because both conditions are measured the same way. This trades measurement precision for speed, determinism, and zero-dependency reproducibility.

### Optimized for: cross-harness portability over platform-specific features
By shipping as a SKILL.md file (the Agent Skills open standard) rather than a platform-specific plugin, the skill works across Claude Code, Cursor, VS Code Copilot, OpenAI Codex, Gemini CLI, and ~25 other harnesses. The standalone system-prompt version extends reach to any LLM chat interface. This is a distribution strategy that treats the prompt as the portable artifact.

### Sacrificed: per-model tuning for generality
The skill applies the same rules to all models, from Opus 4.5 to Sonnet 5. The benchmark results show this works — every model improved — but the variance in reduction (41% for Opus 4.8 vs 82% for Opus 4.6) suggests per-model adaptations could yield further gains. The authors chose generality and simplicity over maximum per-model performance.

### The benchmark design is honest about its limits
The results page leads with an "Honest number warnings" section that lists four caveats: the linter undercounts, the skill condition has higher input tokens, one generation per cell, and no tool can guarantee compliance. This is unusually candid for a benchmark page and reflects the project's overall ethic of treating claims as testable and caveats as required.

## Comparison Notes

### vs. Linters and Guardrails
[[Guardrails and Feedback Loops]] argues "linters beat prompts" — deterministic enforcement over instructions. SimpleEnglish is the hybrid: a prompt-based system whose quality gate is a deterministic self-check (and can be measured by a deterministic regex linter). It's a prompt that behaves like a linter by making the model lint itself. The ste_lint.py tool also functions as an external guardrail — measure the output, fail if violations exceed a threshold.

### vs. Slop Score
[[Slop Score]] measures AI writing tics quantitatively (60% slop words + 25% not-x-but-y patterns + 15% slop trigrams). SimpleEnglish provides the prescriptive fix — a complete rule system that eliminates those tics. The two projects are complementary: Slop Score diagnoses, SimpleEnglish treats.

### vs. Writing Style Guides for Better UIs
Ian Langworth's [[Writing Style Guides for Better UIs]] case feeds a style guide (IBM Carbon) to a coding agent for UI copy. SimpleEnglish is the same approach applied to a more rigorous standard (aerospace controlled language) for a broader domain (all technical writing, not just UI strings). The mechanics are identical: give the agent a rulebook, have it self-check, deliver clean output.

### vs. Steering Claude Code
[[Steering Claude Code]] taxonomizes seven instruction-delivery mechanisms in Claude Code, with skills as one of them. SimpleEnglish is a canonical example of the "skill" mechanism — a SKILL.md file that loads on demand, applies domain expertise, and includes its own reference materials. It demonstrates every principle in the steering taxonomy: progressive disclosure (mode selection first), self-verification (the non-optional self-check), and scoped authority (refuses marketing, stays in docs).

### vs. the Skills Ecosystem
SimpleEnglish fits into the broader ecosystem mapped by [[Agent Skills Library (dzhng)]], [[Audit Skills for AI Coding Agents (metacircu1ar)]], and [[PAAD — Defense-in-Depth for AI-Assisted Development]]. Where those skills focus on code (security reviews, architecture checks, deployment verification), SimpleEnglish focuses on text output — it's a quality guardrail for the prose agents produce, not the code they write. It's also notable as one of the most rigorously benchmarked skills in the ecosystem, with a 96-generation matrix across 6 models.

### vs. brooks-lint
[[brooks-lint]] is another pure prompt-engineering skill that embeds domain expertise (12 classic engineering books) into agent instructions. Both projects share the pattern of "encode expert judgment as searchable rules, have the agent apply them, verify mechanically." SimpleEnglish is more rigorous in its self-check mechanism and has quantitative benchmarks; brooks-lint is broader in its diagnostic scope (12 decay risks across the full SDLC).

---

*Source: https://github.com/AminBlg/SimpleEnglish*
*Last updated: 2026-08-01*
