# Unit Economics of AI Software

The zero-marginal-cost superpower that defined SaaS for fifteen years is eroding. Every LLM call costs real money, turning software from a fixed-cost business into one with genuine per-user variable costs. The result: margins compress (from 75-85% to ~52% in AI products), pricing models shift from flat-rate to usage-based, and the "grow now, margins later" playbook stops working.

---

## Key Quotes

> "The moment your product makes an LLM call every time a user does something, that superpower starts to erode."

The thesis in one sentence. It's not that AI makes software expensive — it's that AI makes software *normal*. Every other industry has always had to manage per-unit costs. Software's exceptionalism was a historical anomaly, and AI is closing that anomaly.

> "For the first time, margin and quality are in direct conflict on a fundamental per-unit basis. It is a hardware-style decision in a software world."

The cleanest articulation of the new tradeoff. Traditional SaaS could improve quality without meaningfully affecting marginal cost. AI flips this: a better user experience literally costs more per interaction. Component selection — a discipline software founders have never needed — is now a core skill. #concept

> "Flat-rate subscriptions socialise that difference across your user base, which works until your heaviest users are expensive enough to blow up your margins. Usage-based pricing is the rational response: align what you charge to what it actually costs you to serve."

This explains the visible shift toward usage-based pricing across AI-native companies. It's not a product strategy choice — it's survival arithmetic. The pricing model changed because the cost model changed first. #pattern

> "ICONIQ puts average gross margins on AI products at around 52% in 2026, improving year on year as companies get smarter about cost management, but still well below the 75-85% that SaaS investors internalised as a baseline."

The number that makes the argument concrete. At 52% margins, the LTV/CAC math that justified aggressive SaaS growth spending no longer works. The venture playbook was calibrated on 75%+ margins. At 52%, every assumption in the LTV model needs recalibrating.

> "Jevons paradox tends to kick in: historically, when a resource gets cheaper, we find more uses for it rather than consuming less. Cheaper inference does not get banked as margin improvement; it gets consumed by deeper integration, more calls per interaction, more autonomous agents running in the background."

The optimist's counterargument — "inference costs are falling!" — gets the correct rebuttal. Cheaper ≠ less total spend. The history of coal, bandwidth, and compute all point the same direction: efficiency increases total consumption. #concept

> "The zero marginal cost era produced extraordinary companies. It also produced a lot of businesses that mistook a favourable cost structure for a genuine competitive advantage."

The sharpest observation in the piece. Many SaaS companies weren't actually good at building valuable software — they were good at operating in an industry where the cost structure did the heavy lifting. AI exposes the difference. #concept

## Key Themes

### The End of Software Exceptionalism

Software was unique among industries because its marginal cost of production approached zero. This wasn't clever business design — it was a property of the medium. AI makes software resemble every other industry: you have input costs, you have to manage them, and your margins depend on how well you do. The hardware-style "component selection" analogy is apt: which model you serve on becomes a make-or-buy decision for every feature.

### The Margin-Quality Frontier

Before AI, you could improve product quality without meaningfully increasing per-user costs. Now the two are in direct tension. Use GPT-5.5 and your product is better but your margins are worse. Use a cheaper model and you protect economics but risk competitive displacement. This is a genuine tradeoff frontier — you can't have both, and where you sit on it becomes your strategy.

### Usage-Based Pricing as Survival Arithmetic

The shift toward usage-based pricing isn't a product philosophy — it's math. When a power user can cost 20× more to serve than a casual user, flat-rate subscriptions become an adverse selection problem: you attract the expensive users and lose money on them. Usage-based pricing aligns what you charge with what it costs. The companies adopting it aren't innovating on business models — they're responding to a cost structure that flat-rate pricing was never designed for.

### Jevons and the Cost-Reduction Trap

Falling inference costs are real, but they don't solve the margin problem — they change its shape. Cheaper inference means more inference: deeper integration, more calls per interaction, autonomous background agents. The cost per call drops but the calls per user multiply. This isn't a prediction; it's a pattern that holds across coal, bandwidth, compute, and now inference. The companies banking on cost reductions to rescue their margins are making the same bet that proved wrong in every prior domain.

### The Honest Founder's Advantage

The piece ends on a surprisingly optimistic note: the founders who take these tradeoffs seriously might build *better* companies than the SaaS generation. Not because the economics are better — they're worse — but because the discipline forces honesty about where value is actually being created. The SaaS era's favourable cost structure let mediocre products survive on good unit economics. AI forces the product to earn its place.

## Critical Analysis

**What the piece gets right:** The core economic argument is airtight. Software's zero-marginal-cost property was always the engine under SaaS's growth-at-all-costs playbook. Break that property, and the entire venture math recalibrates. The ICONIQ 52% figure anchors what could otherwise be an abstract argument, and the tradeoff between model quality and margin is genuinely novel — it's a tension that didn't exist in any prior era of software.

The Jevons paradox rebuttal to the "costs are falling" argument is the most important analytical move in the piece. Without it, the entire thesis could be dismissed as a temporary problem that Moore's Law will solve. With it, the thesis becomes structural: efficiency gains increase consumption, they don't reduce it.

**What's undersold:** The piece focuses on the application layer (startups building on foundation models) and doesn't grapple with what happens if model companies vertically integrate. If Anthropic or OpenAI decides to build the application themselves, the inference-cost problem gets swallowed by the model company's own margins — and now the startup isn't just paying per-token costs, it's competing with its own supplier. This is the same structural trap [[AI Will Not Make You Rich]] identifies: success invites extraction.

**The pricing model insight deserves more development.** The shift to usage-based pricing is correct and well-argued, but the piece doesn't explore the second-order effects: usage-based pricing makes revenue less predictable, which makes venture funding harder to model, which changes who gets funded and on what terms. The SaaS predictability premium — the reason SaaS companies traded at higher multiples — was always downstream of subscription revenue's smoothness. Usage-based pricing trades margin alignment for revenue volatility.

**Compared to [[AI Killing B2B SaaS]]:** This piece is the cost-side companion. [[AI Killing B2B SaaS]] argues the threat is customers building their own replacements. This piece argues the threat is the cost structure itself — even if nobody rebuilds your product, your margins might not support the business you thought you were building. Both are right; they're different failure modes.

**Compared to [[The Minimum Viable Unit of Saleable Software]]:** This complicates Brandur's framework. Brandur assumes LLMs make building cheaper but not free, and human oversight remains the expensive input. But if inference costs become the dominant expense, then the build/buy calculus Brandur works through is the wrong question — the real question is whether the *operating* economics work at all, regardless of how cheap the initial build was.

**Compared to [[The Road Runner Economy]]:** Raford argues software becomes trivially replicable. This piece adds a crucial complication: replicable doesn't mean free to *run*. You can one-shot a competitor's product, but if running it requires inference calls that eat your margins, one-shotting it might still be a bad business. The Road Runner thesis needs to account for the operating economics this piece describes.

**The unanswered question:** What's the new LTV/CAC math? The piece says the old playbook stops working but doesn't sketch what replaces it. At 52% margins, with usage-based pricing, with variable cost-to-serve — what does "good unit economics" actually look like? The ICONIQ data suggests companies are getting better at cost management (41% → 52% over two years), but even at 52%, the venture math is fundamentally different. Someone needs to do that math.

---

## Related

- [[AI Killing B2B SaaS]] — The demand-side companion: customers rebuilding your product with LLMs
- [[The Minimum Viable Unit of Saleable Software]] — Brandur's buy-vs-build economics, complicated by the inference-cost structure this piece describes
- [[The Road Runner Economy]] — Software becomes replicable; this piece adds that replicable ≠ free to operate
- [[AI Will Not Make You Rich]] — Neumann's containerization thesis: value flows to customers, application-layer startups are structurally trapped
- [[Inference Cost Napkin Math]] — The technical companion: KV-cache hit rate IS your margin, duty cycle as the 5× multiplier
- [[AI Pricing]] — Model pricing data across providers; the raw inputs to the margin-quality tradeoff
- [[AI Value Chain]] — Where durable value sits across the AI stack; the three questions that determine who captures margin
- [[The AI Productivity Paradox]] — Output vs. outcomes; this piece adds: even *good* outcomes may not produce good margins

---

*Sources: [[raw/something-is-changing-in-the-unit-economics-of-software]], [[summary/something-is-changing-in-the-unit-economics-of-software]]*
*Last updated: 2026-08-06*
