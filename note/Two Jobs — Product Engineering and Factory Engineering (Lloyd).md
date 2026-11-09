# Two Jobs — Product Engineering and Factory Engineering (Lloyd)

Zach Lloyd (Warp's CEO) argues that after the current era of interactive prompt-driven development comes the "software factory" — agents doing autonomous work across the whole lifecycle — and that the hard part isn't the technology but the engineer's identity. His prescription: engineers now have exactly two jobs, building the product and building the factory that builds the product, and the factory work is real engineering, not overhead.

---

## What the post says

- **The paradigm ladder.** Hand-written code → steering agents via local interactive prompting (now standard) → software factories with autonomous work across the entire lifecycle. Lloyd thinks the factory shift is *harder* for people than the last one, but inevitable for productivity and cost reasons.
- **The metaphor problem is named honestly.** "Dark factories" with no humans imply engineers are unnecessary — that breeds attrition. "Factory" itself feels like an assembly line or building your own replacement; "AI teammate" feels like managing sycophantic junior engineers. For craft-proud engineers, this reads as commoditisation of software creation.
- **Work goes public by default.** What used to be the private "inner loop" becomes visible and measured — including per-engineer token efficiency. Lloyd calls this "a feature, not a bug" while admitting it can make people feel like cogs.
- **Two jobs.** *Product engineering:* humans keep intent, taste, and context; purely automatable slices (crash fixes, papercuts, dependency upgrades) start at ~20–30% and rise. *Factory engineering:* measure the factory with DORA metrics and LLM-as-a-judge scorers, then improve skills, context, and model mix — "you don't adjust your factory on vibes."
- **The metric that carries it:** "human touches per PR," which should decline over time as the factory takes over implementation, adversarial code review, and computer-use verification videos. At Warp, factory improvement is baked into feedback and performance reviews, and senior engineers are asked to model it.

## Key quotes

> "There will be some types of product work that don't need any human input at all... This percent starts low (for Warp, it started around 20-30%), but goes up over time."

The honest number in the piece. Most vendor writing on factories avoids quantifying how much of the engineer's current work is already gone; Lloyd gives a baseline and makes driving it up an explicit goal.

> "At Warp we make clear to our engineering team that they now have two main jobs: building the product and building the factory that builds the product."

The essay's actual thesis. Rather than fighting the demoralising metaphor, he redefines the role so the factory is something engineers own rather than something done to them.

> "They could view this as a drag, but they also can view it as a game, and winning that game means shipping more for less."

Reframing measurement as motivation. It's a real management technique — and also the piece's most revealing sentence about what factories do to engineering culture.

## Analysis

This is a vendor essay doing double duty: Warp Factories is in early access with a $10k incentive for "qualified companies," and Lloyd is describing his product's category as an inevitability. Read it with that discount applied. But the discount doesn't erase the substance, because the two most interesting claims here are the ones a vendor would rather not make: that the factory metaphor is demoralising and that public, token-measured work makes engineers feel like cogs. Most factory evangelism skips straight past the human cost; Lloyd names it and then offers the two-jobs reframe as the answer.

The reframe is good but incomplete. Making factory engineering part of performance reviews is exactly the kind of move that works at a small, well-paid, mission-driven team like Warp and curdles elsewhere — measuring efficiency publicly is the mechanism Lloyd himself says makes people feel like cogs, and performance-review integration doesn't dissolve that, it formalises it. The "game" framing works only for people who are already winning. The piece is strongest as an account of what the engineer's job becomes (intent, taste, and factory tuning) and weakest on who bears the discomfort of total visibility. [[Human Judgment Doesnt Leave the Software Factory — It Relocates]] answers this from the practitioner side: judgment relocates, and a good factory is measured by how intelligently it places human attention — which is a more defensible response to the cog problem than gamified metrics.

#concept #pattern

## Related pages

- [[Build the Factory, Let It Run, Go Kayaking (Warp, Fall 2026)]] — Lloyd's later offsite post strengthens this one: the same two-jobs doctrine matures into Warp reclassifying itself as an infrastructure company, with cost-per-PR down 70% as the factory-engineering payoff.
- [[Crawl, Walk, Run to the Software Factory]] — the adoption guide Lloyd refers to ("I've written a guide for interested teams"), which gives the operational ladder behind this essay's cultural argument.
- [[Human Judgment Doesnt Leave the Software Factory — It Relocates]] — complicates Lloyd's framing by relocating judgment rather than reducing humans to touch-counts; a better answer to the demoralisation Lloyd diagnoses but doesn't fully solve.
- [[AI Software Factory (Lieben)]] — an independent build of the same pattern; comparing the two shows how much of "the factory" is cultural reorganisation versus tooling.

---
*Sources: [[raw/2104957956794057068]], [[summary/2104957956794057068]]*
*Last updated: 2026-11-09*
