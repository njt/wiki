# SimpleEnglish

A Claude Code skill that forces LLMs to write technical documentation in ASD-STE100 Simplified Technical English — the controlled language aerospace has used since 1983 so a tired mechanic cannot misread an instruction. The key insight: STE's rules (max 20/25-word sentences, one word per meaning, simple tenses, active voice, no hedging) happen to be a near-perfect negative of every AI writing tell. Applying a 40-year-old aerospace standard eliminates AI slop as a side effect. 72.9% fewer STE violations across 6 models and 96 measured generations.

---

## Architecture

The entire project is a single 327-line SKILL.md file — pure prompt engineering, zero executable code beyond the eval harness. It works in ~25 harnesses via the Agent Skills open standard.

The skill is structured as a procedure the model follows:

1. **Mode selection**: Pragmatic (structural rules, domain words stay) or Strict (full vocabulary discipline).
2. **Classification**: Procedural (instructions: imperative, 20-word limit) or descriptive (explanations: simple tenses, 25-word limit). Every other rule depends on this.
3. **Pre-writing vocabulary lock**: Pick one verb for check/verify/confirm and one noun for config/settings BEFORE drafting — prevents the synonym rotation LLMs default to.
4. **Rule catalog**: 53 rules in 9 sections, paraphrased from ASD-STE100 Issue 9 (2025) with software-industry examples.
5. **Self-check** (non-optional): Four mechanical checks — word count in longest sentences, grep for banned patterns (contractions, "has been", "should", trailing "-ing", semicolons), condition placement verification, synonym rotation scan.
6. **Untouchables exemption**: Code blocks, identifiers, CLI commands, and quoted errors are exempt. Each counts as one word toward sentence limits (Rule 8.6).

Supporting files provide a standalone system prompt (including a ~60-token version), a full verification checklist for audits, use-case adaptations (error messages, runbooks, incident reports, release notes, agent instructions, translation prep), and before/after examples.

## Eval Infrastructure

The evaluation is unusually rigorous for a prompt engineering project:

- **ste_lint.py** (132 lines): Deterministic regex linter catching 10 violation categories, with a self-test fixture asserting every violation fires on slop and zero on clean.
- **run_bench.py** (199 lines): Resumable benchmark harness running headless Claude Code across 6 models × 2 conditions × 8 scenarios = 96 generations. Includes blind pairwise judging (`--judge`) that scores both orderings to control for position bias.
- **pressure-tests.md**: TDD-style test scenarios documenting that baseline agents without the skill wrote 40-word sentences and invented rule numbers; the skill was written to close each recorded failure.

## Key Techniques

**Classification-first**: The model must classify text as procedural or descriptive before applying any rules. Different limits, verb forms, and structural rules apply to each. Without this, boundaries blur and limits misfire.

**Pre-writing vocabulary constraint**: Rather than catching synonym rotation post-hoc, the skill makes vocabulary choice a deliberate pre-drafting step — exploiting the model's ability to follow self-stated constraints.

**Deterministic self-check as quality gate**: The four mechanical checks are marked "not optional." The model catches its own violations before the user sees them — a lightweight compilation/pass pipeline, but for prose.

**"Untouchables" bridge to software**: STE was built for aircraft manuals. The exemption for code, identifiers, and commands (each counting as one word per Rule 8.6) is what makes the standard usable for software documentation.

**TDD-style prompt engineering**: Baseline failures were recorded first (40-word sentences, fabricated rule numbers, passive voice), then rules were written to close each gap, then re-tested until agents pass. The self-check was revised based on first-run failures.

**"Modal ladder"**: A compact decision table: should (requirement) → must; should (recommendation) → delete/state as fact; may/might/could → can; would (hypothetical) → restructure as conditional. A reusable pattern for removing hedging from any procedural text.

## Design Decisions

**Optimized for correctness, not style**: Explicitly refuses marketing copy and brand writing. "STE deletes persuasion on purpose." Does one thing and refuses everything else.

**Completeness sacrificed for portability**: The official STE dictionary (~900 words) is copyrighted. The skill paraphrases rules and provides substitution patterns without reproducing dictionary content — MIT-licensed and freely distributable, but can't enforce word-level compliance in strict mode.

**Regex over NLP for measurement**: ste_lint.py is a regex pass, not a grammar parser — can't detect passive voice or verify parts of speech. Transparent about this, argues comparability matters more than absolute accuracy for before/after measurement.

**Cross-harness portability over platform integration**: Ships as a SKILL.md file (Agent Skills standard) rather than a platform-specific plugin. Works in Claude Code, Cursor, Copilot, Codex, Gemini CLI, and ~25 other harnesses. Standalone system-prompt version extends reach to any LLM interface.

**Honest benchmarks**: The results page leads with four caveats (linter undercounts, skill condition has higher input tokens, one generation per cell, no tool guarantees compliance). This transparency is unusual and reflects the project's ethic of treating claims as testable.

## Comparison Notes

SimpleEnglish is the prescriptive complement to [[Slop Score]]'s diagnostic — Slop Score measures AI writing tics; SimpleEnglish provides the rule system that eliminates them.

Where [[Writing Style Guides for Better UIs]] feeds IBM Carbon to an agent for UI copy, SimpleEnglish applies a more rigorous standard (aerospace controlled language) to a broader domain (all technical writing). The mechanics are identical: give the agent a rulebook, have it self-check, deliver clean.

As a skill, SimpleEnglish exemplifies the principles in [[Steering Claude Code]]: progressive disclosure (mode selection first), self-verification (non-optional self-check), and scoped authority (refuses marketing). It's one of the most rigorously benchmarked skills in the ecosystem mapped by [[Agent Skills Library (dzhng)]].

The project is a case study in [[Guardrails and Feedback Loops]]'s core thesis — "linters beat prompts" — but with a twist: it's a prompt that makes the model lint itself, backed by a deterministic external linter for verification. The self-tightening feedback loop is explicit: baseline failures → rule changes → re-test.

Unlike most agent skills which focus on code quality (security reviews, architecture checks), SimpleEnglish targets text output — it's a quality guardrail for the prose agents produce. Its honest benchmark methodology and TDD-style prompt development make it a reference implementation for skill authors.

---
*Sources: [[raw/simple-english]]*
*Tags: #project #tool #writing #guardrails #agents*
*Last updated: 2026-08-01*
