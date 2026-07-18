# Writing Style Guides for Better UIs

Ian Langworth's crisp two-minute case for feeding a writing style guide to a coding agent and letting it enforce the rules across every user-facing string in a single pass. The thesis: UI copy was "committee-shaped work" — slow, collaborative, and easy to defer — and agents compress it from days to seconds.

---

> "An LLM does a competent version of it in seconds."

This is the core economic argument. For UI copy, *competent-in-seconds beats perfect-in-a-week* because the alternative isn't perfection — it's inconsistency and delay. The bar was never that high to begin with; it was that the process couldn't clear even a low bar efficiently.

> The IBM Carbon Design System's content guidelines emphasize "everyday language, short words, and a tone that adapts to the moment (economical for errors, friendlier for onboarding)."

Langworth's go-to resource. Carbon's content guidelines are available as raw MDX on GitHub, which means they can be fed directly to an agent as context. This is the practical detail that makes the whole workflow work: a published, machine-readable style guide that's actually good.

---

## Key Themes

**#pattern — Style guide as agent skill.** Feed the guide once, have the agent extract and apply the rules, save as a reusable project skill. The extract-once-apply-everywhere pattern is the same one behind [[Claude Code Mastery]] and [[Steering Claude Code]].

**#tool — IBM Carbon Design System.** Not just a component library — its content guidelines are the specific resource Langworth recommends. Available as raw MDX, making them agent-digestible without scraping or conversion.

**#concept — Competent-in-seconds beats perfect-in-a-week.** The economic reframe that makes agentic copy editing viable. Not "AI writes better copy than humans" but "the process was so slow that even competent output is a dramatic improvement."

**#tool — The Elements of Style as Claude Code plugin.** Strunk's 1918 text, packaged for agent consumption. Langworth mentions it but hasn't tried it. The idea of canonical style guides as installable agent skills is under-explored territory. Closest kin: [[Elements of Code]], which borrows the Strunk & White framing for software.

---

## Critical Analysis

**The argument is right but incomplete.** Langworth correctly identifies that UI copy is a coordination problem, not a craft problem — it's slow because it requires consensus across designers, developers, and PMs, not because the writing itself is hard. An agent bypassing the committee is genuinely faster. But he doesn't address the obvious failure mode: what happens when the agent applies rules mechanically and produces copy that's *consistent* but *wrong* — tone-deaf for the context, technically inaccurate, or culturally inappropriate?

**The Carbon pick is interesting and underexplained.** Carbon's content guidelines are good, but they're designed for enterprise SaaS, not consumer apps, developer tools, or games. Langworth doesn't discuss what makes a style guide *agent-compatible* vs. just well-written. The implicit property is that Carbon's guidelines are *machine-readable* (raw MDX), *comprehensive* (covers tone for multiple contexts), and *actionable* (gives rules, not philosophy). Most style guides fail on at least one of these.

**The skill-extraction workflow is the real insight, and it's barely a sentence.** Langworth mentions extracting rules into a reusable skill as a "better approach" and moves on. But this is the durable engineering move: the agent reads the guide once, produces a compressed rule set, and that rule set becomes project infrastructure. This is exactly the [[Compound Engineering]] pattern — encode taste once, apply forever. The article would be stronger if it developed this thread.

**The Elements of Style mention is a tease.** Strunk & White as a Claude Code plugin is a fascinating idea — but Langworth hasn't tried it, and neither has anyone he cites. The gap between "classic writing advice" and "actionable rules for UI strings" is large, and an agent might not bridge it well without careful prompting. [[Slop Score]] and [[Why Does AI Write Like That]] document the specific ways AI prose goes wrong; a style guide that doesn't address those failure modes directly is only half a solution.

**Missing: verification.** Langworth describes the *generation* step but not the *check* step. How do you know the agent actually applied the style guide correctly across 200 strings? The answer is probably "read them" — but at scale, that's the original bottleneck returning in a different form. A linter for style guide compliance would close the loop. [[Guardrails and Feedback Loops]] has the pattern: deterministic enforcement beats prompt pleading.

---

*Sources: [[raw/writing-style-guides-for-better-uis]]*
*Last updated: 2026-07-18*
