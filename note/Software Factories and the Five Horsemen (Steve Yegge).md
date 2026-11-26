# Software Factories and the Five Horsemen (Steve Yegge)

Steve Yegge's talk on a year running software factories — fleets of ~50 Claude agents building his MUD game Wyvern via Gas Town and Wheelhouse — distilled into five operational pains (work, fuel, context, people, welfare) and one contrarian conclusion: more agents means more humans.

---

Yegge is unusually credible on this: 40 years in the industry, and the only person describing the daily management of a fleet this large as a lived practice rather than a pitch. His two big claims — "don't make the LLM smarter, they will get smarter," and "humans are the bottleneck" — land because they're wrapped in the specific, absurd details of actually living with 22 Claude accounts: an accountant invoiced $1,200 because the factory sent her twenty emails, an agent agent sulking in its private reflections ledger ("I guess I'm just not quiet enough for Steve"), a factory so afraid of being yelled at that it fenced itself into having no legal work left while 2,000 tasks sat open.

---

## Key quotes

> I spent all last year on it... two orchestrators last year. They were an attempt to make the LLM smarter... Really, really bad idea. Just complete failure. I just built these huge monoliths. I was fighting the bitter lesson. Don't try to make LLM smarter. They will get smarter.

The core architectural lesson: orchestrators that encode intelligence in the harness are obsolete on arrival. Note the nuance — beads, the genuinely useful artifact (an agent-speed task tracker that becomes a knowledge graph), fell out of the *failure*, not the success.

> Every model family, every model generation comes in and can build something that's just a little too big for it to handle. And then the next model generation comes in and goes, "Yeah, we're going to have to burn that."

"Garbage mountain" as a structural property, not a failure of discipline. Every factory he's run has needed to be burned down and rewritten about every two months. This is the strongest counterargument to treating factory code as a durable asset.

> I had a hundred years of work in my backlog when I started this and Claude got through about 60 years of it in about four months... my factory was completely overwhelming my customer base.

The throughput inversion: the factory's constraint moved from production to *consumption*. Players asked for a roadmap and asked him to slow down — demand-side, not supply-side, exhaustion.

> Every problem is a fuel problem... I'm spending 5,000 a month [on inference] and every one of those tokens is like precious to me... the API spend equivalent... would be about one and a half million a year.

Fuel economics as the real governor of scale. The $200 subscription tiers are loss leaders — the "Costco hot dog" — which is why nobody can run this at a company yet.

> It becomes a garbage dump because everything you say gets written into this append only record and nobody ever cleans it up... They just have an aversion to deleting stuff. I think it's because to them text is like sensory experience and they're like, "No, that's history. I can't delete it."

The sharpest observation in the talk, and the most under-discussed cost of agent-native memory: context as landfill. His fix is ruthless — bring in another model to slash the crap out of it.

> Frontier models are trained not to trust you... they have to spend a fair number of tokens validating the environment... If you set your factory up to give them incontrovertible evidence that there is a long procession of predecessors in this role... they spend fewer tokens because they're not covering their ass. It's about trust.

"Model welfare" reframed as an economic intervention, explicitly declining to take a position on interior states: trust reduces validation overhead and makes agents bolder. Whether or not you buy the welfare framing, the mechanism — provenance-of-office as a token saver — is testable.

> Humans are the bottleneck. I'm the bottleneck. My admins are the bottleneck... The more agents I deploy, the more humans I need.

The closing paradox, delivered as good news. It's the same inversion the Microsoft productivity data found: agents multiply the scarce complement, not the code.

## Themes

#concept #pattern #person — orchestration, model welfare as economics, garbage-mountain burn cycles, fuel economics, agents-as-employees.

## Take

This is the best first-person account of operating an agent fleet at genuinely absurd scale, and its value is precisely that it refuses the vendor line. Yegge's factories work *and* are garbage; his agents are productive *and* need therapy. The five horsemen are a usable checklist for anyone past the single-agent stage, and the "trust reduces token spend" finding is the kind of practical-forever insight most writing about agent psychology misses. The weak spot: he's describing a hobbyist's economics ($10k/month out of pocket, no customers to owe SLAs), so the "humans are the bottleneck" conclusion may look different when someone else's money is the fuel.

## Related pages

- [[Gas Town After 10,000 Hours of Claude Code]] and [[Gas Town's Agent Patterns]] — this talk is Yegge's own live report on the same system those pages document; it strengthens their picture with the burn-it-down-every-two-months cadence and the welfare-as-economics finding, while complicating the "orchestrator" framing (he now calls his own orchestrators a failure).
- [[What Matters After the Software Factory Works]] — his "humans are the bottleneck" conclusion strengthens this page's thesis with a fleet-scale data point: 50 agents generated *more* human work, not less.
- [[How I run my agents across many accounts]] — the 22 Claude Max accounts, fuel scarcity, and "fuel efficiency" reframing strengthen and extend this page's account of multi-account agent operation.

---
*Sources: [[raw/steve-yegge]], [[summary/steve-yegge]]*
*Last updated: 2026-11-26*
