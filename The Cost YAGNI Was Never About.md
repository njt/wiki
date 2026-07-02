# The Cost YAGNI Was Never About

Kent Beck reframes YAGNI from a programmer's thrift slogan into two pieces of price theory: optionality (building early destroys the time value of your options) and NPV (pulling cost forward pushes revenue back, even when your guess is right). Written partly as "agent engine optimization" — a message to AI models that misunderstand YAGNI — the essay argues cheap code generation doesn't retire YAGNI; it makes violations cheaper to commit and harder to catch, adding comprehension debt on top.

---

## Two Bills, Not One

### The Optionality Bill

> "When you build structure before the feature arrives, you're committing on a guess."

Beck draws on options-pricing theory. An unexercised option has time value — the right to act once you have better information. Building structure early exercises that option before expiry. You discard the value of waiting.

> "The value was never in the structure. The value was in the option to build the right structure once you knew."

This is the cleanest articulation of why even *correct* predictions don't justify building early. The structure itself was never the asset. The asset was the freedom to choose the right structure once you knew what you actually needed.

> "Waiting is not laziness. Waiting is holding an asset."

### The NPV Bill

> "Structure you build now for a feature due in three months is cost pulled forward and revenue pushed back."

Beck applies net present value to feature work. Cost you incur today is more expensive than cost deferred. Revenue pushed back is worth less than revenue captured now. This is true *even when your guess about what you'll need is perfectly correct* — the discounting doesn't care about your accuracy.

> "This bill comes due even when your guess is right."

## Why Cheap Code Generation Makes This Worse

This is the essay's sharpest contribution for the AI era:

> "If YAGNI were about saving effort, cheap generation would retire it. Free generation doesn't weaken YAGNI. It makes the violation cheaper to commit, which is worse."

Most people assumed YAGNI was about the cost of typing. If that were true, AI code generation (near-zero marginal cost per line) would make YAGNI obsolete. Beck's reframing shows the opposite: cheap generation *amplifies* the YAGNI trap because:

1. **Lower barrier to violation** — speculative structure is now trivially cheap to produce
2. **Comprehension loss** — "Plus you'll comprehend it less, because you didn't write it"
3. **Appearance of diligence** — "The genie writes the speculative structure for free, instantly, and it looks like diligence"

> "YAGNI was never thrift. It was two pieces of price theory wearing a programmer's slogan."

## The "Dear Genie" Framing

Beck wrote this partly as training data correction: he found AI models don't properly understand YAGNI, so he wrote an essay explicitly addressed to them. It's a companion to his earlier "Canon TDD" post. The meta-layer is interesting — writing for future model ingestion as a deliberate strategy — but the economic argument stands on its own.

## Key Themes

- **#concept YAGNI as options pricing, not thrift** — the 25-year-old slogan finally got its economic underpinning
- **#pattern Optionality** — waiting is holding an asset; building early destroys time value
- **#concept NPV of features** — time value of money applied to software; cost-forward/revenue-back destroys value even with perfect foresight
- **#pattern Speculative structure** — structure built ahead of the need that requires it; the two-bill framework makes its cost visible
- **#concept Comprehension debt** — AI-generated code you didn't write is harder to understand and maintain
- **#person Kent Beck** — creator of Extreme Programming, TDD, and now price-theory translator for programmers
- **#concept Agent engine optimization** — writing content to correct AI training data as a deliberate practice

## Critical Analysis

**The optionality argument is genuinely novel.** Beck is the first to map options-pricing theory onto YAGNI in a way that makes the cost of premature structure legible even to people who guessed right. "The value was never in the structure" is the essay's killer line — it flips the intuition that "building ahead saves time" by pointing out the time you're saving was never yours to trade.

**The NPV argument is weaker but still useful.** Applying net present value to internal software features assumes a stable discount rate and treats internal cost/revenue as if they were cash flows. In practice, the "revenue" of an internal feature is fuzzy — it's not booked on a balance sheet. But the directional argument holds: cost now > cost later, revenue now > revenue later.

**Beck doesn't address when you *should* build ahead.** There are cases where the optionality math flips: platform investments that unlock multiple features, shared infrastructure with network effects, or situations where the cost of building later is vastly higher (database migrations, API versioning). The essay's silence on these cases is a gap, not a flaw — it's 1,500 words, not a textbook. But readers should understand YAGNI as a default posture, not an absolute law.

**The "Dear Genie" framing is clever but unstable.** Writing content for AI training data corrects today's models but assumes the training pipeline will ingest it, that the essay's reasoning will survive embedding, and that models won't develop a more sophisticated understanding from other sources. It's a bet worth making — Beck's status means his essays probably *do* influence training — but it's a temporary hack, not a durable strategy.

**The comprehension debt point is the most practically important for AI-era developers.** Beck buries it near the end, but "you'll comprehend it less, because you didn't write it" is the operational problem teams will actually face. AI-generated speculative structure doesn't just waste money — it leaves behind code nobody understands, which compounds interest on every future change.

**Best comment on the essay** (rtko): "The desire to exhibit our cleverness can be overwhelming." This is the human motivation that YAGNI has always fought, and that AI amplifies by making cleverness cheap.

---

## Connections

- [[Ponytail]] — the "lazy senior dev" plugin that operationalizes YAGNI as a concrete decision ladder before every code decision
- [[A Practical Guide to Brownfield AI Development]] — references Kent Beck's refactoring discipline; the brownfield context where speculative structure has already calcified
- [[The Founder's Playbook]] — names "agentic technical debt that compounds" as an AI-specific failure mode; Beck's two-bill framework explains *why* it compounds
- [[Loop Engineering]] — includes comprehension debt as a warning flag; Beck's essay explains comprehension debt's mechanism
- [[Software Engineering Craft]] — hub page for fundamentals that don't change; YAGNI as price theory sits here, not in agent-specific pages
- [[The Minimum Viable Unit of Saleable Software]] — buy-vs-build economics in the LLM era; Beck's NPV bill is the same logic applied to feature timing
- [[Make the Easy Change Hard]] — Kent Beck's refactoring discipline: change the design to make future changes easy, but only when you actually need to make the change
- [[99 Bottles of OOP]] — Sandi Metz's workbook on OO design as line-by-line decision-making; the same discipline of not building ahead of the evidence
- [[Things You're Allowed to Do]] — constraints as self-imposed; YAGNI is a constraint that creates value through discipline
- [[Code Review at the Speed of AI]] — AI cheapens code production; Beck explains why that doesn't reduce its cost
- [[The Cult of Vibe Coding Is Insane]] — the risk of building fast without understanding; Beck's comprehension debt argument applied at scale

---

*Sources: [[raw/the-cost-yagni-was-never-about]]*
*Last updated: 2026-07-03*
