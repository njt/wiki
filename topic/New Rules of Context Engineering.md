# New Rules of Context Engineering

Anthropic's Thariq Shihipar distills the lessons from removing 80% of Claude Code's system prompt — with no measurable loss on coding evaluations — into a practical guide for what context engineering looks like in the Claude 5 era. The post is part field report (here's what we deleted and why), part design philosophy (six "then and now" reversals), and part configuration manual (how to apply progressive disclosure to your own CLAUDE.md, skills, and system prompts). The throughline: newer models need fewer guardrails, better interfaces, and context that loads at the right time rather than all at once.

---

## Key Quotes

> "We were able to remove over 80% of Claude Code's system prompt for models like Claude Opus 5 and Claude Fable 5 with no measurable loss."

This is the headline number, and it's worth sitting with. The system prompt isn't some optional decoration — it's the instruction set governing every Claude Code session. Removing four-fifths of it with no regression means the old prompt wasn't just verbose; it was actively **unnecessary** for the new models. This is the strongest empirical evidence yet for the "simplify as models improve" thesis that runs through [[Tuning Claude Code Into a Better Engineering Partner]] and [[Steering Claude Code]].

> "Constraints that were once necessary to prevent worst-case outcomes can now be deleted, letting context and judgment suffice instead."

The quiet paradigm shift. Old prompt engineering was defensive — assume the model will do the worst thing unless told otherwise. The new approach is trusting the model's judgment and only intervening where it demonstrably fails. This is the same inversion [[Writing a Good CLAUDE.md]] advocates: don't use a CLAUDE.md line to do a linter's job.

> "Giving Claude examples actually constrains them to a certain exploration space."

A genuinely counterintuitive claim. For years, the first rule of tool design was "include examples." Shihipar argues that with Claude 5 models, examples narrow the solution space rather than broadening it. The new principle: design expressive interfaces (clear parameter semantics, enums that hint at state machines) and let the model reason from the interface itself. This is a design philosophy, not just a prompting trick — it's about making tools that are self-documenting.

> "Instead of a central repository for every known practice… a tree of files that loads at the right time."

Progressive disclosure as an architectural principle, not just a loading optimization. This makes CLAUDE.md and skills into composable, evaluable components rather than a monolithic context dump — exactly the argument [[Context Engineering at the Frontier (Linus Lee)]] makes for why composable retrieval beats bigger context windows.

## Key Themes

#context-engineering #claude-code #prompt-engineering #pattern #system-prompt #progressive-disclosure

## The Six Reversals

Shihipar structures the article around six "then and now" pairs, each a reversal of conventional prompting wisdom:

| Then | Now | What Changed |
|------|-----|-------------|
| Give Claude rules | Let Claude use judgement | Models handle nuance; rigid rules create conflicts |
| Give Claude examples | Design interfaces | Examples narrow exploration; expressive interfaces broaden it |
| Put it all upfront | Progressive disclosure | Load context when needed, not preemptively |
| Repeat yourself | Simple tool descriptions | Duplication was a crutch for weaker attention |
| Memory in CLAUDE.md files | Auto-memory | Models can decide what's worth remembering |
| Simple specs | Rich references | Code, HTML, and rubrics beat markdown descriptions |

The most interesting reversals aren't the ones about making things shorter (delete repetition, drop examples) — those are obvious efficiency wins. The deeper ones are structural: **progressive disclosure** changes how you architect context, and **auto-memory** changes who owns the persistence decision.

## Critical Analysis

**The 80% figure is doing a lot of work, and the article doesn't show the work.** Which 80%? What stayed? What evaluations measured "no measurable loss"? Without this, the number functions more as a vibe — "simplify aggressively" — than as a replicable finding. That's fine for a blog post, but teams should treat it as directional, not prescriptive.

**The "let Claude use judgement" advice has an unstated prerequisite: the model must be good enough.** Opus 5 and Fable 5 apparently clear the bar, but the article doesn't address what happens with weaker models, or where the threshold lies. For teams using tiered model strategies ([[Thrifty (Tiered Delegation for Claude Code)]]), the implication might be: strip guardrails for Opus, keep them for Haiku. That's useful, but unstated.

**Progressive disclosure is the most actionable concept, but it requires infrastructure.** You can't just "split your CLAUDE.md into a tree of files" — you need a mechanism that loads the right file at the right time. Claude Code Skills do this natively. Rolling your own agent harness doesn't. The article's advice is strongest for Claude Code users and thinnest for custom agent builders.

**The article is strategically silent on what they *didn't* delete.** A system prompt slashed by 80% implies the remaining 20% is extremely load-bearing. What survived? The article doesn't say, but those survivors are probably the most important instructions in Claude Code. This is the [[Load-Bearing Assumptions]] pattern applied to prompt engineering: find the things you can't delete, and those are your real requirements.

**The auto-memory section is the only part that reads as product marketing rather than engineering insight.** "Claude automatically saves memories that are relevant" — how? What's the mechanism? What's the error rate? Without this, it's a feature announcement, not a transferable lesson.

**The companion piece is conspicuously absent.** The article closes by pointing to "the companion Fable field guide" for more on prompting. That guide isn't linked, and its absence makes this article feel like part one of a series that hasn't shipped yet.

## Connections

This post sits at the intersection of several threads in the wiki:

- **[[Steering Claude Code]]** is the reference manual for the instruction-delivery mechanisms Shihipar is optimizing. Where that post maps the *what* (seven mechanisms, their loading behavior, compaction survival), this post maps the *how much* (delete 80%, use progressive disclosure, trust judgment).
- **[[Agent Memory and Context]]** provides the broader landscape this post fits into: memory taxonomies, persistence strategies, and the context-as-RAM metaphor. Shihipar's post is the "less is more" operational complement.
- **[[Writing a Good CLAUDE.md]]** makes the instruction-budget argument (150-200 instructions max) that explains *why* deleting system prompt content works. Shihipar validates it from the other direction: Anthropic did it and it worked.
- **[[Context Engineering at the Frontier (Linus Lee)]]** makes the composability argument from the search-engineering angle; Shihipar makes it from the product-engineering angle. Both converge on the same prescription: don't dump everything in context, structure it so the right things load at the right time.
- **[[Tuning Claude Code Into a Better Engineering Partner]]** is the practitioner's field manual for the same philosophy: shrink CLAUDE.md, use skills for procedures, let hooks enforce deterministically. Shihipar provides the official doctrine; jsdev.space provides the field-tested practice.
- **[[The New Software Lifecycle]]** identifies context engineering as "the underrated financial lever" in agent-assisted development. Shihipar's post is a concrete guide to pulling that lever.
- **[[Loop Engineering]]** treats context engineering as the meta-skill — designing the systems that prompt agents rather than prompting agents directly. Progressive disclosure is a loop engineering technique.
- **[[Guardrails and Feedback Loops]]** provides the framework for distinguishing what should stay as instructions from what should move to deterministic enforcement. Shihipar's "let Claude use judgement" only applies to the former.

For teams building custom agent harnesses, the system prompt section ("this is where you should spend a lot of time") pairs with [[Components of a Coding Agent]] and [[The Agentic Product Standard v2.0]]. For the skills-as-lightweight-guides philosophy, see [[PAAD — Defense-in-Depth for AI-Assisted Development]] and [[Cloudflare Security Audit Skill]] for worked examples of skills that encode "particular opinions, knowledge, or best practices" without overconstraining.

---

*Sources: [[raw/new-rules-context-engineering]]*
*Last updated: 2026-07-29*
