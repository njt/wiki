# Inside a Software Factory

Paul Iusztin (Decoding AI, republished on O'Reilly Radar) defines the **software factory** — the entire SDLC run with autonomy — as eight stages in three buckets (what to build, building and checking, self-improving) wrapped around an orthogonal context layer, then answers the two questions that matter: where the human still belongs (brainstorming, planning, and the final merge), and when a factory becomes overbuild (his own Squid v1 taught him the hard way). It is equal parts blueprint, field report, and a shot at the "____ engineering" label arms race.

---

## Key quotes

> "The right frame is the software factory… defined by Tereza Tížková as 'the whole loop, the whole lifecycle of developing software with autonomy.'"

The definitional move. Iusztin's argument is that the factory — not the loop, not the graph — is the unit worth theorizing, because it is the only frame that spans from intake to production monitoring.

> "We're overexplaining intuitive things we started doing years ago. While you're defining what counts as a loop, you're not thinking about the processes that actually deliver software."

The essay's hook and its sharpest line. Prompt/context/harness/loop/graph engineering are treated as a marketing arms race — the terms aren't wrong (he concedes Boris Cherny's "My job is to write loops"), but the taxonomy has outrun the practice.

> "Addy Osmani frames the stack as loop, harness, factory: 'The loop is the atom'; a factory is 'an org chart made of loops.'"

The composition hierarchy the essay adopts wholesale: loops nest into harnesses, harnesses into factories. Osmani supplies the vocabulary; Iusztin supplies the SDLC instantiation.

> "As agents tend to have a positive bias towards their own work… the model that wrote the code is 'way too nice grading its own homework.'"

The reason the implement stage pairs a software-engineer agent with a *separate* QA agent. Same adversarial-review instinct as [[Poor Man's Loop Engineering]], embedded in the factory's stage structure.

> "Total cost is tokens × price, not model tier. So more failures equals more reasoning, more tokens, and more cost."

The crispest cost argument in the cluster: spend the strong model (Fable) on brainstorming and planning because everything downstream compounds off plan quality. A weak plan makes Sonnet-on-high-reasoning out-cost Opus on the same task. Planning is the financial lever, not implementation.

> "When the loop keeps failing, the root cause is almost always missing plumbing, not the agents."

Debugging discipline for factories: if the QA agent can't drive the app, the fix is "one command that starts the whole stack reproducibly," not a better prompt.

> "The bottleneck is me, and that's by design… I don't understand who the people shipping 100 features in parallel are."

Solo-scale honesty. Features build on each other, the parallelizable subset is limited, and he confines parallelism to local agents in worktrees — never 24/7 remote fleets. This is the essay's most valuable admission: even the factory's evangelist runs a factory sized to one person.

> "You cross the buy line the moment engineers you don't personally supervise run agents."

The build-vs-buy heuristic, expanded into a three-zone map: the smallest builds, the middle buys, the largest builds again. Buy when observability, tracing, and pay-per-token billing become someone's full-time job (Factory.ai, Warp's Oz); build back when platform constraints cost more than the team it would take to replace the platform.

## Key themes

#concept — the software factory as eight-stage SDLC pipeline: triage → brainstorm → plan → implement → review → review-CI → release → monitor, with monitor feeding back into triage
#concept — the context layer as orthogonal infrastructure; LLM wikis ([[LLM Wiki]], Factory's AutoWiki, LangChain's OpenWiki) as agent-queryable knowledge bases
#pattern — human placement: indispensable at brainstorming and planning, agents own the middle, human returns for the final merge; "planning is, and always will be, human-driven"
#pattern — granular commands plus an optional end-to-end chain: the ability to halt, redirect, and step in as the design constraint that makes autonomy safe
#person — Paul Iusztin (Decoding AI), Addy Osmani, Zach Lloyd, Tereza Tížková, Boris Cherny
#tool — Squid (his factory), Claude Code, Codex, OpenCode, Pi, Factory.ai (Droid), Warp's Oz, BMad method
#comparison — build vs. buy as three zones, not a binary

## Opinionated analysis

**This is a consolidation piece, and that is its value.** Nothing here is discovered; the Lloyd blueprint and the Osmani stack predate it. What Iusztin adds is the tempering that vendor write-ups omit: Squid v1's failure is the honest data point. The first version chased full autonomy — "a big monolith that took me too far out of the loop" — and died the first time reality went off-script. His two-mode fix (granular commands for steering, one end-to-end command for when you trust the run) is the same containment instinct as [[Harness Engineering is not Enough]], arrived at from the builder's side rather than the critic's.

**The label-sneering is half right.** The essay mocks loop/graph engineering as overtheorized marketing while its own eight-stage model is literally a graph of loops. The difference — and it's real — is that his graph is grounded in the SDLC rather than in dataflow aesthetics: the loops exist because intake, planning, review, and monitoring are jobs that need doing. The critique lands on overexplaining, not on the concepts. [[Agentic Engineering at Kenn]] makes the same move from the other direction ("loops are bullshit" — except human-operator loops).

**The load-bearing assumption deserves suspicion.** "If you spend enough time creating a strong plan, the PR that reaches you is usually ready to ship as-is" is the hinge the whole human-placement argument swings on — it is why he can claim brainstorming and planning are the *only* permanent human stages. Horthy's counter-thesis is that no amount of upfront planning compensates for a training objective that selects against maintainability; Iusztin sidesteps it (strong plan + human merge gate) rather than refuting it. His own closing honesty — factories are "far from being fully 'autonomous,'" and anyone claiming otherwise "either hasn't tested the idea enough or is trying to sell it" — buys him more credit than the confidence in ship-as-is PRs.

**Where it's weakest:** the evidence base is one solo practitioner. "I don't understand who the people shipping 100 features in parallel are" is a confession, not an argument, and the build-vs-buy thresholds are intuition dressed as map. But as a *solo-scale* counterweight to the fleet-size visions of [[Cloud Software Factories]], that's precisely the point — this is what the factory looks like when the org chart it automates has one human on it.

## Relate

- [[Cloud Software Factories]] — strengthens it: Zach Lloyd's control-room blueprint gets its missing counterpart here — Iusztin cites Lloyd twice and supplies the tempering (don't overbuild, the buy line, the solo ceiling) that a CEO's pitch never volunteers.
- [[Loop Engineering]] — nuances it: the essay accepts Osmani's loop mechanics ("the loop is the atom") but argues the label game is overtheorized; the design leverage sits in composing loops across the SDLC, not in defining the loop.
- [[Own the Outer Loop]] — strengthens it from the same publisher: Osmani's outer-loop accountability becomes named checkpoints (brainstorm, plan, merge) here, and Iusztin's "the bottleneck is me, and that's by design" is the outer loop admitting it stays outer.
- [[Harness Engineering is not Enough]] — complicates it: Horthy blames lights-off factory failures on the training objective; Squid v1's failure was plumbing and monolithic autonomy, and Iusztin's fix (human-driven planning, granular control) sidesteps rather than tests Horthy's claim.

Also adjacent: [[Matt Pocock — Grill Me, Then Go AFK]] (the agent-grills-you planning stage and the plan-human/implement-agent split are the same pipeline), [[How to Build an AI Software Factory]] (Firecrawl's five-stage control plane is the systems-engineering cousin of this essay's SDLC sketch), and [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] (the level taxonomy Iusztin implicitly climbs from Level 2 to the edge of Level 4 and stops).

---
*Sources: [[raw/inside-a-software-factory]], [[summary/inside-a-software-factory]]*
*Last updated: 2026-09-13*
