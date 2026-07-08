# The Lindy Effect

The Lindy Effect is a technology-choice heuristic: the longer something has survived, the longer it's likely to keep surviving. For software engineers, it's a check against shiny-object syndrome — invest in what's proven (algorithms, SQL, core languages) over what's trendy (this month's framework).

---

## Precis

The Lindy Effect flips biological intuition on its head. Living things lose life expectancy as they age; non-perishable things — ideas, technologies, books, traditions — gain it. Time is a brutal filter that kills what's fragile and preserves what's high-quality. Dr. Milan Milanović argues developers should apply this lens to technology choice: a tool that's been around for decades is a better bet than one that shipped last week. The examples are stark — COBOL still runs 70-80% of global financial transactions at 60+ years old, while JavaScript frameworks cycle through history every few years.

## Key Themes

**#concept — Lindy as a decision heuristic:** The Lindy Effect isn't a law of physics; it's a probabilistic bet. Something that's survived 20 years has demonstrated fitness against 20 years of changing constraints. That doesn't guarantee it'll survive 20 more, but it's a better prior than betting on something unproven.

**#pattern — Time as quality filter:** The mechanism is selection, not magic. Fragile tools die. Bad ideas get forgotten. What survives is what's good enough to keep working across shifts in hardware, platforms, and user expectations. SQL survived because it solves a genuinely hard problem (declarative data access) in a way that composes. COBOL survived because it's embedded in systems too expensive to rewrite.

**#comparison — Lindy vs. YAGNI:** These are complementary filters. Lindy says "prefer what's proven" for technology selection. [[The Cost YAGNI Was Never About|YAGNI]] says "don't build it until you need it" for feature selection. Both push back against premature complexity, but from different angles — Lindy from the outside (what tools to use), YAGNI from the inside (what features to add). Kent Beck's reframing of YAGNI as options pricing mirrors Lindy's probabilistic reasoning.

## Critical Analysis

This is a useful heuristic, not a decision rule, and the article is honest about that — it calls it a "rule of thumb," not a law. The real value is as a counterweight to the tech industry's relentless novelty bias. Every week there's a new framework, a new paradigm, a new way to do what we already know how to do. Lindy gives you permission to say "no, I'll use Postgres."

But it has limits the article doesn't explore. First, Lindy is conservative by design — it'll steer you toward the tried-and-true but also toward the stagnant. If everyone applied it strictly, we'd never have anything new. The trick isn't to never adopt new things; it's to calibrate *when* to bet on novelty (high-upside, low-switching-cost situations) vs. when to stick with the proven (core infrastructure, career skills). Second, Lindy is path-dependent: COBOL isn't around because it's good; it's around because it's expensive to replace. That's stability, not quality — and conflating the two is the heuristic's biggest blind spot.

Milanović's specific framing for developers — "invest in timeless skills" — is the strongest part and the most actionable. Core languages, algorithms, design principles, databases: these compound. Framework-specific expertise depreciates. The career advice writes itself.

## Key Quotes

> "The longer something has been in use, the more likely it is to continue being used."

The definition is deceptively simple. The interesting part isn't the claim — it's the asymmetry: this is true for technologies but false for organisms. That's the conceptual leap Taleb popularized and Milanović applies to software.

> "Things that have been around for some time stick around longer in the future."

Almost tautological, but the power is in operationalizing it: when you can't predict the future, bet on the things that have already survived a lot of pasts.

> "Truly fundamental ideas in software change slowly."

This is the core insight beneath the heuristic. The substrate changes fast (frameworks, languages, deployment targets), but the deep structure — how to model data, how to handle failure, how to manage complexity — evolves glacially. Lindy works as a heuristic precisely because software's fundamentals change slowly enough for survival time to carry signal.

## Cross-References

- [[Software Engineering Craft]] — The hub page for fundamentals that don't change. Lindy is essentially an argument for craft over novelty.
- [[The Cost YAGNI Was Never About]] — Kent Beck on why cheap code generation makes YAGNI violations *more* dangerous, not less. The complementary decision filter.
- [[The Joy and Power of Understanding]] — Deep understanding of fundamentals as both pragmatic path and intrinsic reward. Lindy applied to learning.
- [[Ponytail]] — The "lazy senior dev" plugin that forces the simplest solution first. Lindy operationalized as system prompt.
- [[The Mundanity of Excellence]] — Excellence through qualitatively different choices (like picking proven tech) rather than more hours. The mechanism behind why Lindy works.

---
*Sources: [[raw/lindy-effect]]*
*Last updated: 2026-07-08*
