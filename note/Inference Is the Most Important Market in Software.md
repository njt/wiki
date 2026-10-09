# Inference Is the Most Important Market in Software

Tomasz Tunguz's market-sizing essay arguing that AI inference spending will overtake the database market — the defining software category of the mid-2010s — and that this crossover structurally mutates every software application into an inference reseller, with consequences for pricing, margins, and sales motions.

---

## The argument

The core claim is a market-size crossover: companies paid ~$25b to run AI models in 2025, ~$130b in 2026 (within reach of Gartner's $161b database forecast), and ~$350b by 2027 — nearly 2x the projected $190b database market. Whatever the precision of the numbers, the shape is plausible: inference spend is growing faster than any prior software category.

The interesting part is not the forecast but what Tunguz says it does to the application layer. Three rewirings:

- **Usage dwarfs platform fees.** The inference bill exceeds the seat license or base subscription. This changes AE compensation, demands predictable plans for customers afraid of blown budgets, and complicates forecasting.
- **Gross margins compress.** The ~72% blended SaaS gross margin decays unless the vendor builds a proprietary harness that aggressively compresses token overhead, or fierce model competition drives inference prices down faster than supply constraints push them up.
- **BYOK trades revenue for margin.** Enterprises supplying their own GPU clusters or model API credentials leave the vendor with nearly pure software margin — on a vastly smaller contract.

On the price-war escape hatch: capability-adjusted token costs fall ~47% per quarter, but frontier list prices keep resetting upward with capability leaps — the blended frontier price rose from $0.96 to $11.25 in 18 months before "GPT-6 Sol" reset the tier to $4.00. Falling cost-per-token and rising list price are both true simultaneously, at different points on the capability frontier.

> "Just like a long-haul trucker watching diesel surge beyond $8 per gallon & wondering about their economics, software companies will watch price per token & hunt for efficiencies on their rigs with smaller models, better harnesses, & new routing techniques."

The trucker analogy is the right frame: fuel as a volatile, dominant COGS input changes an industry's structure, not just its pricing pages.

## Themes

#concept #pattern

## Critical take

The thesis is strong on direction, weak on precision — Gartner forecasts and 47%-per-quarter deflation curves are exactly the numbers that don't survive contact with a budgeting cycle. But the three rewirings are the durable content, and they name tensions the rest of the wiki keeps circling: usage-based pricing vs. predictable budgets is a real product-design problem, not a finance footnote; and the margin story quietly vindicates the harness thesis. If inference is the COGS, then "proprietary harness to compress token overhead" is the moat — which is Tunguz's own nod to the argument that [[The Harness Is the Company]] makes at the strategic level and [[Managing AI Coding Costs at Scale]] makes at the operational one.

It also nuances the deflation narrative elsewhere in the wiki. [[Tokens Too Cheap to Meter]] argues per-task cost falls orders of magnitude per year; Tunguz's frontier list-price reset shows why the *average* buyer's bill may still rise — capability tier creep eats deflation. Both are true; the framing decides which one you budget for. And the BYOK dynamic gives the small-models-and-routing advice in [[Inference Cost Napkin Math]] a business justification, not just an engineering one.

The weakest link: "every application becomes an inference reseller" assumes inference stays metered and centralized. Open-weight local inference is the structural escape from that assumption — if a meaningful share of consumption moves on-prem, the market-size crossover may still happen, but the reseller economics won't.

---

*Sources: [[raw/inference-is-the-most-important-market-in-software]], [[summary/inference-is-the-most-important-market-in-software]]*
*Last updated: 2026-10-09*
