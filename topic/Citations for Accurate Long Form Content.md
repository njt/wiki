# Citations for Accurate Long Form Content

Ian (statico) solved Claude Opus's persistent hallucination problem on long-form technical blog posts with a single, surgical prompt addition: require the model to emit a citation callout after every paragraph, listing the exact files, line numbers, commit hashes, and Discord URLs backing its claims. The citations aren't for human review — they're breadcrumbs that let subagents verify claims one by one, turning an impossible fact-checking problem into a pile of small, local verification jobs that agents handle well.

---

## Key Quotes

> "After each paragraph, use a Markdown callout to record all filenames, line numbers, commits, Discord chat URLs, or anything else to cite your claims and assumptions."

The prompt addition is one sentence. Not a framework, not a multi-agent pipeline, not a new model. Just a formatting constraint that structures the output into verifiable units.

> "The citations aren't for me to check. They're breadcrumbs for the next subagent to fact-check against."

This is the key inversion. Citations as a protocol between agents, not as a concession to human skepticism. The first agent generates claims with attached evidence; subsequent agents verify. Each agent does what it's good at.

> "The model was now bounded by the quality of its sources, not by its own confabulation. That's the line I wanted to get to."

The destination: error comes from missing or stale data, not from the model inventing facts. This is the same line between "the model doesn't know" and "the model is lying" — the former is fixable with better sources, the latter requires a different approach entirely.

> "An essay interleaved with footnotes the model wrote to itself."

A vivid description of the output format, but also a concise summary of what compound engineering looks like in practice: the system talks to itself across passes, not just to the user.

## Key Themes

#prompt-engineering #fact-checking #verification #subagents #compound-engineering #pattern

## Critical Analysis

**This is Compound Engineering in miniature.** Ian's move is a textbook example of [[Compound Engineering]]: when you can't trust the output, don't switch to manual review — add a system. The citation callout is a system, not a suggestion. It constrains the output format so tightly that verification becomes automatable. This is the difference between "be more careful" (a prompt) and "produce verifiable claims" (a harness).

**The breadcrumb metaphor is more precise than it sounds.** Breadcrumbs aren't the meal — they're a path back through territory the first agent traversed. Without them, a fact-checker has to re-traverse the entire codebase, re-read every Discord thread, re-derive every conclusion. That's exactly what Opus is bad at (long-horizon reasoning over messy sources). With breadcrumbs, each claim is a local lookup: "does this commit actually change this behavior?" That's what subagents are good at. The insight isn't just "add citations" — it's "decompose verification into tasks that match agent capabilities."

**This solves the verification bottleneck without solving verification.** The dirty secret is that the subagents doing fact-checking might also hallucinate. Ian's result — "the only remaining inaccuracies were things not recorded in git or Discord" — suggests this works in practice, but there's no guarantee. A subagent reading a commit might misunderstand the diff just as easily as the drafting agent did. The pattern works because it makes verification *cheap enough to afford errors in the verifiers* — you can run three subagents per claim and vote, or run one and accept the residual error rate. Compare [[Agentic Manual Testing]] where Willison makes the same case for `python -c` and `curl` as verification tools.

**The boundary matters.** Ian notes that Bug Bot is barred from touching the gameserver directly — "the boundary is drawn at the API." This isn't incidental. The citation pattern works because the sources (git, Discord, file system) are read-only and well-structured. If the sources were unreliable or agent-writable, the entire verification chain collapses. This is the same insight as [[Guardrails and Feedback Loops]]: the guardrail has to be at a different level than the thing being guarded.

**What's missing:** Ian doesn't address what happens when sources contradict each other, or when the "ground truth" in git/Discord is itself ambiguous. He also doesn't discuss how to handle claims that *can't* be cited — the drafting agent's interpretation of developer intent, or synthesis across multiple sources. Those are the hard cases, and they're where confabulation creeps back in. The citation pattern pares the problem down but doesn't eliminate it.

**A one-sentence prompt fix that works is more valuable than a 10-agent architecture that doesn't.** The sophistication-to-impact ratio here is off the charts. One sentence. Massive accuracy gain. This is the kind of thing that makes prompt engineering feel less like engineering and more like lock-picking — small, precise moves that exploit specific model behaviors.

---

*Sources: [[summary/citations-for-accurate-long-form-content]]*
*Last updated: 2026-05-22*
