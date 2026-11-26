# Staying Under the Line — Maintaining an Agent-Written System (Dmitry)

An ai.engineer talk (machine transcript) from a practitioner maintaining a large, ~75% agent-written "pros" instruction engine in 110 countries. His thesis: agents made code cheap and entropy cheaper, rewrites don't hold, and the only sustainable posture is staying under the line of unreviewable change with weekly small sweeps and feedback loops built into the system — organised as six "stones": time, space, reality, mind, power, soul.

---

## What it argues

The talk is a maintenance memoir, not a how-to-prompt guide. The speaker tried the big gesture first — he deleted unused tests and boilerplate, rewrote the whole project, lost useful pieces — and it failed because "the agents kept writing and the mess grew back." The third option is operational: never let change volume exceed what anyone can check, and spend human judgment only where it's genuinely needed.

The sharpest insight is about what "correct" means. He borrowed mutation testing for instruction files and almost nothing turned red: the tests couldn't tell him what had to survive a rewrite. "For code, tests can tell you. For pros, only people can." Correctness in this system is a group-of-experts property, so experts are brought in from the start and review each task exactly as the agent saw it.

## Key quotes

> "Coding agents made code cheap. They made entropy even cheaper."

The talk's one-line premise, and the Thanos/snap frame that structures it. The snap (full rewrite) is a real option and he took it — the point is that it doesn't hold when agents keep producing.

> "Whatever sits in its context becomes a reason to act."

When the model could choose what to load, it loaded everything and invented a pricing check just because prices were visible. The runtime, not the model, decides what the model sees — three facts from the order, handles instead of full records. This is a stronger statement of context-as-control-surface than most context-engineering pieces: context isn't just a cost, it's an affordance for hallucinated action.

> "The one person every change waits on. Agents just get you there in weeks instead of years."

Every tech lead has been the single queue; agents compress the time-to-bottleneck. His first instrument is himself: "When I'm tired and I can't explain what changed, that's the signal. The dashboard can be green while that gauge is firing."

> "Conway said the system copies the org chart. Ours copies whoever corrects it."

The soul-stone price, admitted late: his skills and memory carry his own habits and biases. A system maintained by one corrector becomes that corrector — the honest version of the maintenance story everyone else tells as triumph.

## Themes

#pattern — walks-to-code: repeated agent paths through instructions are promoted into workflow tools and the instructions deleted, but only when the walk repeats and agrees across countries. #concept — ratchets and blast radius: a new check must be replayed over thousands of past sessions and must fail once on purpose before it's trusted. #tool — the hand-built "compiler" that lints pros files like code. #person — the burned-out architect who became the message bus between users and his agents.

## Critical take

This is one of the most honest maintenance accounts in the wiki because it refuses both happy endings. The rewrite failed; the all-human review failed (burnout); the all-agent path failed (agents "fix what is easy to see and break consistency on the way," and silently skip ~75% of a backlog when context fills). What's left is unglamorous: weekly sweeps, countable evidence, a second agent that checks "without the author's reasoning" so it can't be talked into agreeing.

The quiet weakness is that everything rests on the boundary between pros and code being drawn well — "judgment stays wherever the answer varies in a place an expert can read" is a principle that demands exactly the judgment the talk says is scarce. The boundary "moves both ways" (hundreds of lines of expressions migrated out of code because experts couldn't read them), which means the architecture is perpetually renegotiated by committee. That's honest, but it's also the part that doesn't obviously survive a different team.

## Relation to the wiki

- Strengthens [[Agentic Verification]] with the deepest data point in that conversation: ~9/10 of the time on a change goes to checking, so "verification speed is the limit" — and adds the mutation-testing-on-instructions twist showing tests are the wrong oracle for prompt-layer systems.
- Complicates [[Who Maintains the Code Agents Write (arXiv Empirical Study)]]: the arXiv study found agent code gets half the maintenance commits and humans do 83% of it; this talk is the practitioner-side account of exactly that human share, including the burnout the study can't measure.
- Extends [[The Persistent Gravity of Cross Platform]]'s verification-bottleneck logic into the instruction layer: it's not just code review that bottlenecks, it's pros review, and the fix is to compile instructions like code rather than read them like prose.
- Nuances [[How agent.ceo Fixes Its Own Bugs]]' process rules with scale: same ideas (externally checkable done, second reviewing agent, no silent failure) but at 110 countries, where the second-agent check "grew to most of my checking" and the ledger replaces the architect's memory.

---
*Sources: [[raw/online-dmitry-maintenance]], [[summary/online-dmitry-maintenance]]*
*Last updated: 2026-11-26*
