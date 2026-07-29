# Writing a Good CLAUDE.md

HumanLayer's Kyle (@0xblacklight) makes the definitive case for short, hand-crafted CLAUDE.md files. The novel contribution is the instruction-budget argument: frontier LLMs reliably follow ~150-200 instructions, Claude Code's system prompt already consumes ~50, so you've got maybe 100-150 slots left. Every line you add degrades every other line — not just the new ones, all of them. The corollary: CLAUDE.md is "one of the highest leverage points of the harness." A bad line here cascades across every plan, every research task, every code generation.

---

## Key Quotes

> "LLMs are stateless functions" with frozen weights. They possess no knowledge about your codebase except what you provide in tokens.

The article's premise stated plainly. This is why CLAUDE.md matters: it's the one file injected into every session. It's the agent's permanent onboarding.

> Claude Code injects CLAUDE.md wrapped in a system reminder stating: "IMPORTANT: this context may or may not be relevant to your tasks."

Damning. The harness itself tells Claude to treat CLAUDE.md as optional. Kyle speculates Anthropic added this because users were stuffing the file with narrow "hotfixes" that degraded results — a classic case of the fix making the underlying problem worse. The practical implication: every line in your CLAUDE.md must earn its place by being *universally* applicable, or you're training the model to ignore the entire file.

> "Never send an LLM to do a linter's job."

Not just about cost. Style guidelines bloat the context window *and* degrade instruction-following across the board. Use Biome. Use a Stop hook. Don't waste instruction slots on formatting.

> "As instruction count increases, instruction-following quality decreases **uniformly**" — it's not just the new instructions that suffer, but **all** instructions.

This is the killshot finding from arXiv:2507.11538. Adding a bad instruction doesn't just waste a slot — it makes *every other instruction* less reliable. This is why auto-generating CLAUDE.md is dangerous: every mediocre line hurts the good lines. Smaller models degrade exponentially; frontier thinking models degrade linearly, but they all degrade.

> A bad line in CLAUDE.md cascades across plans, research, and code generation. A bad line of code is localized.

The strongest argument for manual crafting. `/init` is convenient but reckless. The file is too leveraged for automation.

## Key Themes

#claude-code #context-management #guardrails #best-practices #instruction-budget

## Critical Analysis

**The instruction-budget framing is the real contribution.** Most CLAUDE.md advice is vibes — "be concise," "explain your stack." The 150-200 instruction budget gives a concrete, testable constraint: count your instructions, stay under 100, and remember Claude Code's system prompt already consumed ~50. This alone justifies the post.

**The progressive disclosure advice is already dated.** The article recommends an `agent_docs/` directory of markdown files with CLAUDE.md pointing Claude at the right ones. This works, but Claude Code Skills do it better — they load on demand, can include tool instructions, and don't consume context when irrelevant. The article was written November 2025; the landscape shifted within months.

**The article dodges the CLAUDE.md vs. Skills interaction.** When both exist, what's the priority order? If a Skill says "use TDD" and CLAUDE.md says "don't write tests first," which wins? This is the practical question every team hits, and Kyle doesn't address it.

**"Linters not prompts" is correct but the article underplays the implication.** It's not just that CLI linters are cheaper than LLM token costs — it's that prompt instructions and deterministic enforcement are different *categories*. [[claude-ctrl]] states it plainly: "An instruction that lives only in model context is not a constraint." Kyle frames this as cost optimization rather than categorical difference, which undersells his own point.

**The system-reminder revelation is the article's most important finding for power users.** Knowing that the harness prefaces CLAUDE.md with "may or may not be relevant" changes how you write the file. A database schema instruction isn't just wasted bytes — it trains Claude to disregard the whole file. Universality isn't a nice-to-have; it's a survival requirement.

**Missing: what about CLAUDE.md in subdirectories?** The article only discusses root-level files, but Claude Code supports inherited CLAUDE.md files throughout the directory tree. The instruction-budget math changes if you can scope instructions to specific parts of the codebase.

The bottom line: pair this with [[CLAUDE.md (Universal)]] — that page gives you *what* to say, this gives you *how much* to say and *why less is more*. Together they're the best current guidance on CLAUDE.md authorship. But neither addresses the Skills interaction, and that's the gap that matters next. Read [[Feedback Loop is All You Need]] to understand why even a perfect CLAUDE.md is never enough.

---

*Sources: [[summary/writing-a-good-claude-md]]*
*Last updated: 2026-07-29*

*See also: [[New Rules of Context Engineering]] — Anthropic confirms the instruction-budget thesis from the inside: they removed 80% of Claude Code's system prompt with no measurable regression on Claude 5 models, and the six "then and now" reversals (especially "give Claude rules → let Claude use judgement") validate that the file's leverage demands aggressive curation.*
