# The New Knowledge Artifacts of the AI Economy

Tim O'Reilly argues that source code is being displaced as software's primary shareable knowledge artifact, replaced by "context bundles" — prompts, specs, skills, MCP servers, agent traces, evals — and that the coordinating institutions for this new artifact class (the next GitHub) don't exist yet. He reads the present through history: binary→source→webpage→click each marked a value transition whose winners spotted the new artifact early.

---

## What the essay argues

Foo Camp anecdotes open it: people saying "I don't want someone else's code any more — I may have my agent take a look and extract just the parts I want." A repo becomes a *source of context for solving one problem*, not a packaged solution to a bundle of problems. When code is cheap, the valuable thing is what tells the model what code to generate and how.

O'Reilly's historical frame is the load-bearing part. Open source replaced value-from-scarcity with value-from-abundance (the clothesline paradox — open source *created* far more than it *captured*, which is the system working, not failing). Christensen's Law of Conservation of Attractive Profits: commoditise one layer, value migrates to the adjacent one. New artifacts show up years before their institutions — GitHub came a decade after open source went mainstream. His bet: we are at that same early moment for context, with the institutions "still unbuilt."

The candidate artifact catalogue is the essay's most useful piece:

- **Specs** — spec-driven development makes the spec the reviewed, versioned, durable thing; code is regenerated from it.
- **Agent Skills and MCP servers** — packaged instructions and context that shape model behaviour.
- **Agent traces** — "the closest thing we have to commit history for work that doesn't get formalized as code," and potentially the raw material for attribution and payment.
- **Evals** — possibly "one of the most valuable things a domain expert can create, withhold, or share" — with Nick Vincent's caveat that once test cases leak into training data, they measure memorisation, not capability.
- **Well-formed questions** — even conjectures could get priority-credit infrastructure, arXiv-style but further upstream.
- **Intent signals** — the stream of what people ask agents to do "may become to this era what the click was to the last."

The trust section, via Cronon's Chicago grain trade (grade receipts, futures, counterfeiting, *Munn v. Illinois*), lands the point that new artifacts attract fraud and need grading institutions. Spreadsheet models as the 1984 analogue: a model is only as good as its assumptions, and taking one as gospel was then what AI-usage fails are now.

## Key quotes

> "The challenge for the software development community is to identify the new knowledge artifacts of the AI economy, and to build new mechanisms and incentives for sharing them."

The whole essay compressed into one sentence — and note what it doesn't claim: not that the artifacts are identified, only that identifying them is the task.

> "A repo may now serve merely as a source of context for solving the exact problem I have, rather than the bundle of problems that the original developer(s) encoded into their library or packaged solution."

The sharpest reframing in the piece, and quietly radical for open source: the library as *corpus*, not as product.

> "The agent trace, the record of what an agent tried, which tools it called, and which context it drew on, is the closest thing we have to commit history for work that doesn't get formalized as code."

This connects provenance directly to payment: trace logs as "the raw material for attribution and payment, much as the click was the raw material for advertising."

> "Some of the people who notice the new artifacts early, while everyone else is still focused on the old ones, then invent new business models based on them."

His recurring thesis across Web 2.0 and now — the superpower is noticing the artifact before the institutions form around it.

## Themes

#concept #pattern #comparison

## Critical take

This is the best-framed version of an argument that is usually made badly ("code is dead, prompts are the product"). O'Reilly earns it three ways: he hedges ("'displaced' is too strong a word... different species of value overlap for a long time"), he explicitly keeps the old artifacts in play (SaaS for shared-metrics domains, "you pick the hat to fit the head"), and he grounds the pattern in three prior transitions rather than one cherry-picked analogy.

The weak joints are honest ones he half-acknowledges. First, the clothesline paradox cuts both ways: if context bundles create more value than they capture, the incentive to *share* them is exactly as broken as it was for open source — and that took two decades and an institutional stack to solve. He admits the core gap ("part of the problem is that we haven't yet figured out what needs to be shared!") but doesn't resolve it. Second, the eval-leakage point quietly undermines the whole catalogue: evals are valuable *because* they are withheld, which makes them private goods, not shareable artifacts — the piece's most valuable candidate artifact is the one that can't be shared. Third, the trust story (grading standards, neutral supervision of holdout sets) is stated as a need, not a mechanism; Cronon's history is evidence that this takes decades and state intervention, which is cold comfort for the builders he profiles.

Still, as a conversation-starter it does its job: it converts "what do we put in the context window" (an engineering question) into "what is the tradeable object of the new economy" (an institutional one).

## How this connects

- [[A Home for Personal Context (Duncan Davidson)]] — O'Reilly cites Davidson's work directly; this essay supplies the theoretical frame (context as the new tradeable artifact) for Davidson's practical question of where personal context should live.
- [[Spec-Driven Development]] — O'Reilly names spec-driven programming as the paradigm case of a context bundle ("the spec becomes the thing that is reviewed, versioned, and kept"), strengthening the wiki's spec-as-durable-artifact thread with a historical-economics argument rather than a workflow one.
- [[What a Useful AI Trace Should Actually Contain]] — O'Reilly's claim that the agent trace is "the closest thing we have to commit history" and the raw material for attribution gives that page's schema question an economic stake: trace design is now payment-infrastructure design.
- [[Claude Code Skills System]] — O'Reilly cites Agent Skills as an open standard and a leading example of a context bundle; this essay is the macro-theory beneath that format's micro-definition.

---
*Sources: [[raw/the-new-knowledge-artifacts-of-the]], [[summary/the-new-knowledge-artifacts-of-the]]*
*Last updated: 2026-11-11*
