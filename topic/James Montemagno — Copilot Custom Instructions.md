# James Montemagno — Copilot Custom Instructions

James Montemagno makes the case that Copilot's custom instructions feature — a `.github/copilot-instructions.md` file that travels with every chat request — is the single highest-leverage thing a developer can do to improve AI code generation. The concept is identical to CLAUDE.md: define your coding standards, project conventions, and architectural rules in a file, and the AI respects them. VS Code Insiders can now auto-generate these by scanning your workspace. The talk is a practical walkthrough, not a theoretical argument — Montemagno treats instructions as obvious infrastructure and focuses on showing how to use them.

---

## Key Quotes

> "It is the very first thing that you should do."

The talk's thesis compressed into a single sentence. Before you write a line of code with Copilot, define the rules. This mirrors the CLAUDE.md philosophy in [[Claude Code Mastery]] — "CLAUDE.md is compounding infrastructure." The difference is that Copilot's version is per-project by convention (`.github/` directory), not per-user.

> Custom instructions are "the same exact rules that you would tell another team member."

This is the right framing. Instructions aren't mysterious AI incantations — they're onboarding docs. If you wouldn't explain it to a new hire, don't put it in the instructions file. Compare with [[Writing a Good CLAUDE.md]], which makes the same argument but with an instruction-budget constraint: every line degrades every other line. Montemagno doesn't address this limit, which is the talk's biggest gap.

> Instructions are sent "with every chat request so it knows how to respond back."

The mechanism matters. Copilot doesn't re-scan your codebase on every request — it relies on the instructions file as a cached summary of how you work. This is the same pattern as context windows in [[Agent Memory and Context]]: you trade token budget for consistency. The question the talk doesn't answer is what happens when the instructions grow stale and the codebase has moved on.

---

## Key Themes

#copilot #prompt-engineering #context-management #tool #best-practices #ai-coding

The talk covers: custom instructions markdown files, VS Code Insiders' one-click auto-generate feature, the `awesome-copilot` GitHub repo for pre-made instructions, agent mode's use of instruction references, and scoped `.instruction` files for token-efficient language-specific rules.

---

## Critical Analysis

**This is CLAUDE.md for Copilot, and it inherits all the same problems.** Montemagno presents instructions as a pure win — add rules, get better output. But [[Writing a Good CLAUDE.md]] documents a finding from arXiv:2507.11538 that instruction-following quality decreases *uniformly* as instruction count increases. Adding a bad instruction doesn't just waste a slot — it makes every other instruction less reliable. The auto-generation feature is the Copilot equivalent of `/init`, and carries the same risk: auto-generated mediocrity that degrades the whole file.

**The talk dodges the maintenance question entirely.** Instructions are "living documents" that can be "refreshed as the project evolves." Who refreshes them? When? What's the trigger? CLAUDE.md benefits from being user-owned and session-persistent — every mistake becomes a rule ([[Claude Code Mastery]]). A project-level `.github/copilot-instructions.md` has no equivalent feedback loop unless someone manually updates it after every bad Copilot suggestion.

**The auto-generation feature is both the most exciting and most dangerous part.** Scanning a workspace to infer conventions is genuinely clever — it catches things humans forget to document. But it also risks codifying bad patterns. A codebase with inconsistent naming becomes a machine that *enforces* inconsistency. The talk mentions this as an unanswered question; it should be the central question.

**"Linters not prompts" applies here, but Copilot instructions are prompts by definition.** [[Guardrails and Feedback Loops]] argues that deterministic enforcement (linters, type checkers, formatters) beats contextual instructions every time. A copilot-instructions.md that says "use camelCase" is strictly worse than a linter that rejects snake_case at commit time. The instruction file can *complement* deterministic tooling but shouldn't *replace* it. Montemagno doesn't make this distinction.

**The scoped `.instruction` files are the smartest idea in the talk.** Token efficiency matters. Sending your entire project's conventions for a two-line Python change when the Python rules are in a separate `.instruction` file is wasteful. This is the one design choice that shows awareness of the instruction-budget problem, even if Montemagno doesn't name it.

**The team dynamics question is real and underexplored.** If `.github/copilot-instructions.md` is version-controlled, who owns it? What happens when two team members disagree on a convention? Does the file become a battleground for style wars that were previously settled by "just format on save"? The talk treats instructions as individual productivity tools, but in a team repo they're shared infrastructure — and shared infrastructure needs governance.

---

## Cross-Links

- [[Writing a Good CLAUDE.md]] — The same concept for Claude Code, with the instruction-budget critique this talk lacks
- [[Claude Code Mastery]] — CLAUDE.md as compounding infrastructure; the direct parallel
- [[Refactor Legacy Code with Copilot]] — Existing Copilot page in the wiki; shallower but same tool
- [[Guardrails and Feedback Loops]] — Why deterministic enforcement beats prompt-level instructions
- [[Agent Coding Workflow]] — Where custom instructions fit in the daily AI development loop
- [[Engineering for Bounded Cognition]] — The cognitive limit that makes instruction budgets matter
- [[Loop Engineering]] — Prompt engineering as a meta-skill; instructions as one loop component
- [[Matt Pocock — Grill Me, Then Go AFK]] — Similar format: YouTube talk on AI coding workflow optimization
- [[Agent Memory and Context]] — Context windows and the token budget that instructions consume

---
*Sources: [[summary/ytx-james-montemagno-copilot-custom-instructions]]*
*Source URL: https://gist.github.com/485e7e4c84f54b2ff34f060f19593ae2*
*Source video: https://www.youtube.com/watch?v=ZohAaUQBDbs*
*Last updated: 2026-07-04*
