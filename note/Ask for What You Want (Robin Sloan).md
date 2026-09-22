# Ask for What You Want (Robin Sloan)

Robin Sloan — novelist, programmer, and author of the 2020 "apps as home-cooked meals" essay — returns with the agent-era sequel: building a personal app with an AI agent still counts as home cooking, and the skill the whole practice exercises is *asking for what you want*. The piece is equal parts workflow manual (his actual `VISION.md`, reproduced in full), cultural argument against ambient AI, and honest confession ("And, no: I don't look at the code").

---

## Key Quotes

> "My alternative: get your wish, then stuff the genie back into the lamp."

Against the consensus vision of AI as "a constant ambient presence, sparkles in everything," Sloan proposes episodic use: invoke the genie for a wish, then put it away. What he builds is meant to outlast the session — "steady, sensible tools that will continue to work, no matter what."

> "You should imagine the agent 'reading everything at once' — all the words understood in parallel... Rather than 'a clear, sensible explanation', you are aiming for 'useful guidance on the page'. More is usually better."

His theory of the vision document, and a quiet inversion of the terse-prompt school: for a greenfield personal project, exhaustiveness beats tidiness — inconsistencies included, since the agent holds them all simultaneously. Compare [[How I Prompt (Thorsten Ball)]], where the regime is the opposite: pointers and constraints for an agent inside a large existing codebase. Different settings, different prose economics.

> "This process wasn't particularly fast, maybe not even particularly efficient... you must become unembarrassed about nitpicking. You must become Steve Jobs, pointing out every misalignment, every detail insufficiently considered. You can be nicer than Steve Jobs, though."

Unusually honest cost accounting for this genre: screenshots pasted into chat with "this looks weird," features tried once and ripped out ("no, I was wrong, rip that out"). The human's job in the loop is taste, applied relentlessly and without shame.

> "And, no: I don't look at the code."

The essay's most consequential sentence, dropped almost casually. His defense is structural: when designer and user are the same person, "anything/everything becomes sensible, because the underlying logic is always totally clear to you! ... No mysteries." He skips code-understanding while possessing complete *behavioral* understanding — the exact inversion of [[Understand to Participate]]'s premise (see Analysis).

> "Making these apps, I felt my programming muscles wither in realtime, the timelapse of the rotting apple... That feeling honestly frightens me, and I intend to keep it firewalled into this domain."

The cost named plainly and ring-fenced by intent rather than denied. This is comprehension debt ([[Agents and Acquiring Debt]]) as lived experience: he knows the interest is accruing and takes the loan anyway, deliberately scoped to hobby software.

> "Call this Battlestar Galactica engineering: you must at least IMAGINE the Cylon uprising, and ensure that your spaceship will still function if or when that day arrives."

His durability standard for personal software — and the rebuttal to the obvious objection that not reading code means not caring. He repeatedly asks the agent to "Review this code and make sure it's simple, sturdy, and maintainable" and to hunt down whatever is slowest. The review he won't do himself, he institutionalizes as a standing request — the agent will "happily do this again and again, as many times as you want."

> "Discussing these AI systems, everybody wants to talk about 'intelligence', but I don't think that's the profoundest thing about them. Rather, I believe it's the other -ences: patience, diligence."

His candidate for the models' real superpower: the agent that "really, REALLY wants to record the configurations" and spins up a little admin page to track its own trials, in two minutes. The bookkeeping no human hobbyist ever does, done unprompted.

> "What's important is not that it's a notes app, or a sleek jukebox, or whatever... What's important is that it's yours. It's weird and specific, and it won't change unless you want it to change."

The payoff. Backed by rules that define personal software against the attention economy: use anything you build for two months before posting about it; ideally never post; pick the name yourself ("Don't let the AI agent name the app"); never distribute, never publish the code — "This is not software for glory; it is software for you."

> "The moment of asking for an app and getting it is dangerous, because you feel like you accomplished something. You didn't. ... You do, at some point, have to use your tools. You do have to actually make something."

The warning that keeps the essay from being boosterism: generation is not achievement. Compare the "evaluative anesthesia" diagnosis in [[Vibe Coding and the Maker Movement]] — Sloan's two-month-use rule is a behavioral countermeasure to exactly that failure.

## Key Themes

#person #concept #pattern

### The vision document is the human's real work

Sloan's workflow — write `VISION.md`, request `PLAN.md`, approve, co-build — is spec-driven development in miniature, but with the spec written for an audience of one and deliberately loose rather than ceremonial. The writing itself has value even when the agent could have extracted everything by asking: "If you're going to ask for what you want ... you need to know what you want." The document is how the human finds out. Compare the plan-as-artifact discipline in [[The Plan Is the Program]] and [[SDD Case Study — 13 Apps in 70 Days]] — same shape, one order of magnitude less ceremony.

### Episodic, not ambient

The essay's cultural stance: not sparkles in everything, but concentrated wishing followed by retreat. The habit he proposes generalizes beyond apps: "Is the computer working the way I think it ought to work? No? Okay, I'll ask the AI agent to help me change it" — and it applies to the agent itself, "which is, of course, just software."

### Diligence over intelligence

The deepest reframe in the piece. What transforms his creative work isn't model brilliance but the agent's tireless organization — recording configurations, building tracking pages, holding the whole spec in parallel. A claim about where the value actually sits in agent-assisted work, and it sits *below* intelligence.

## Analysis

This is the best first-person specimen yet of the mode [[The Cathedral, the Bazaar, and the Winchester Mystery House]] names from the outside: implementation nearly free, feedback loop of exactly one, idiosyncratic personal tools. But Sloan is not a naive Winchester builder. His houses are small and *finished* ("It was, and is, basically finished"), he caps his own scope (~20,000 notes, load them all into memory), and he runs standing simplicity reviews — anti-sprawl discipline that Breunig's sprawl framing leaves out. The essay strengthens the Winchester thesis while correcting its implied license.

The sharpest tension is with [[Understand to Participate]]. Litt says understanding is the price of staying a collaborator; Sloan says "I don't look at the code" and steers fine. The reconciliation matters more than either pole: Sloan's understanding lives entirely in the *behavior* of the tool, which he designed and uses daily, and his software is disposable by construction (never distributed, no users to break). Litt's rule is written for codebases with stakes and successors; Sloan's exception is written for software that will never outlive its author's interest. The unnerving part is that Sloan concedes the cost himself — the withering-muscles passage is Litt's cognitive-debt warning arriving in the first person — and his response is containment ("firewalled into this domain") rather than avoidance. Whether the firewall holds for anyone whose job *is* software is the question the essay wisely doesn't answer.

It also pairs directly with [[Building When It Feels Like There's Nothing Left to Build]]. Huyen asks why build at all when anyone can rebuild anything instantly; Sloan's answer is the strongest one available, because it opts out of the replication economy entirely: what he builds is "weird and specific," used by one person, never posted, never distributed. There is nothing to replicate and no audience to serve — the value is the fit, and the fit is uncopyable. That is Huyen's "local human preference" argument made concrete by a single practitioner.

Finally, note what the essay is *not*: it is not a claim that agents make software development efficient. Sloan says the opposite — the process "wasn't particularly fast, maybe not even particularly efficient" — and claims something else instead: the "oof" of modification became "a light and inviting 'what if?'". The product here is not throughput; it is the removal of dread from the act of changing your own tools. That framing survives contact with his sources, which is more than most agent-utopia essays can say.

## Related Pages

- [[AGI Is Here (Robin Sloan)]] — the same author, six years earlier, declared AGI arrived in 2020 and left the PC revolution's dangling question: "what now?" This essay is his own 2026 answer — the question dissolves into practice, and the home-cooked-meals metaphor is where the "and...?" finally lands.
- [[The Cathedral, the Bazaar, and the Winchester Mystery House]] — Breunig names the mode (cheap code, feedback-of-one, idiosyncratic tools) that Sloan here exemplifies from the inside; Sloan adds the finished-ness and simplicity-review discipline that keeps his Mystery House from growing 160 rooms.
- [[Understand to Participate]] — Litt's "understand to remain a collaborator" meets its cleanest counterexample in Sloan's "I don't look at the code"; the resolution — behavioral understanding substituting for code understanding, at deliberately zero stakes — sharpens both positions.
- [[Building When It Feels Like There's Nothing Left to Build]] — Huyen's "why build when anyone can rebuild anything" gets its practical answer here: software that is never posted and never distributed sits outside the replication economy, where "it's yours" is the whole moat.

---
*Sources: [[raw/ask-for-what-you-want]], [[summary/ask-for-what-you-want]]*
*Last updated: 2026-09-22*
