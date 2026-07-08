# The Lindy Effect in Software

Clément Sauvage applies the Lindy effect — the longer something has survived, the longer it can be expected to survive — to technology choice in software engineering. Old technologies aren't just not-dead-yet; their longevity is a positive signal of robustness, ecosystem maturity, and reduced risk that newer alternatives structurally cannot offer.

---

## Key Quotes

> "The longer something has survived, the longer it can be expected to continue surviving."

The core Lindy formulation, imported from its original domains (fashion, books) into software. Sauvage is careful to say this isn't a deterministic law — software is young, and real evolution (Apple Silicon, Rust's memory model) is welcome — but it's a heuristic that inverts the default startup instinct of "newer is better."

> "Groundbreaking benchmark results shouldn't be the sole trigger for a full migration."

The pointed critique of Hacker News engineering culture: treating benchmark graphs as migration warrants. Sauvage doesn't say benchmarks are useless; he says they're a single data point that tells you nothing about the decade of operational knowledge embedded in the boring alternative.

> "Large companies play the safest bet — and that bet is Java."

Sauvage's Go-vs-Java contrast is neatly self-aware: he personally prefers Go but understands why enterprises default to Java. The Lindy effect isn't about personal taste — it's about institutional risk management. Java's 30 years of production history is a moat that no benchmark can cross in a weekend.

## Key Themes

- **#concept Lindy effect in software** — longevity as a positive signal for technology robustness, not just inertia
- **#pattern Choose boring technology** — the operational philosophy Sauvage invokes: established tech for core infrastructure, novelty for the edges
- **#concept Prudent adoption** — evaluate before committing; the mirror image of "move fast and break things"
- **#concept Evolution over revolution** — incremental improvement beats constant rewrites, and the Lindy effect provides the theoretical justification for *why* — old systems have absorbed lessons that rewrites discard
- **#person Clément Sauvage** — software engineer and blogger at clemsau.com

## Critical Analysis

**The Lindy effect is real but misapplied at your peril.** The heuristic works for technologies whose failure modes are well-understood, whose maintainers are known, and whose ecosystem is broad enough that you're not the only one betting on it. It does *not* work for technologies that are old but brittle — COBOL on a mainframe with one retiree who knows the source code is not Lindy; it's a hostage situation. Longevity alone is insufficient; you also need the attributes Sauvage lists (stability, ecosystem, industry acceptance). These five benefits are the filter that separates Lindy technologies from merely old ones.

**The "choose boring tech" framing is a useful cultural counterweight.** Software engineering has a well-documented novelty bias — new frameworks, new languages, new paradigms are intrinsically interesting in a way that "Postgres again" is not. Sauvage's contribution is giving that counterweight a name (Lindy) and a mechanism (survival implies fitness). It's the same move Beck made with YAGNI: take an intuition programmers already half-believe and give it enough theoretical scaffolding to win arguments in design reviews.

**But the essay dodges the hardest question: when *should* you bet on something new?** Sauvage acknowledges that evolution is welcome — Apple Silicon is his example — but doesn't give criteria for when the Lindy calculus flips. In practice, the most consequential technology choices are precisely the ones where Lindy offers no guidance: a technology is either old enough to be obviously safe (SQL, C, Unix) or new enough to have no track record at all. The interesting cases — five-year-old technologies with growing but uncertain adoption — fall through the cracks.

**The agent era adds a new dimension Sauvage couldn't have anticipated in 2023.** [[Constraint Decay]] shows that agents perform dramatically worse when structural constraints (databases, architectures, ORMs) are layered on. This means the Lindy effect cuts both ways for AI-assisted development: established technologies have battle-tested patterns that agents can draw on from training data, but they also impose the structural complexity that causes agents to fail. A technology that's Lindy for humans may not be Lindy for agents — and vice versa.

**The deepest connection is to [[The Cost YAGNI Was Never About]].** Beck argues YAGNI is options pricing: building early destroys the time value of your options. Sauvage's Lindy effect is the same logic applied to technology choice: choosing new tech early exercises an option (the right to adopt once the technology has proven itself) before expiry. The Lindy effect is YAGNI for your architecture.

**The essay is thin on negative examples.** Sauvage names C and SQL as Lindy successes but doesn't name any technologies that *seemed* Lindy and turned out to be dead ends (SOAP? CORBA? XML schemas?). A fair treatment would acknowledge that survival bias is real — we remember the old technologies that survived and forget the old technologies that died. The Lindy effect is a probabilistic argument, not a guarantee, and the essay would be stronger for acknowledging that.

---

## Connections

- [[Software Engineering Craft]] — hub for fundamentals that don't change; the Lindy effect is the theoretical spine for "boring technology" as an engineering discipline
- [[The Cost YAGNI Was Never About]] — Beck's options-pricing reframe of YAGNI: choosing new tech early is exercising an option before expiry; Lindy is YAGNI for architecture
- [[Claude Is Not Your Architect]] — Holland's warning against deferring architectural judgment to AI; choosing Lindy technologies requires contextual judgment that AI structurally cannot provide
- [[Constraint Decay]] — the empirical finding that structural constraints degrade agent performance; Lindy technologies impose structure that helps humans but hurts agents
- [[Engineering for Bounded Cognition]] — working within cognitive limits; established tech reduces the cognitive surface area you need to hold in your head
- [[Specifications as the Product]] — code is disposable, specs are the durable artifact; the Lindy effect suggests the durable artifact is actually the *choice* of technology, not just the spec

---
*Sources: [[raw/lindy-effect-in-software]]*
*Last updated: 2026-07-08*
