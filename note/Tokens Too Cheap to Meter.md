# Tokens Too Cheap to Meter

jyn compiles a quantitative case that the cost of completing a fixed AI task is falling ~2.5 orders of magnitude per year — from GPUs, model architectures (MoE, Mamba hybrids), inference engines, and specialized classifiers like Jev — and argues this turns intelligence into *infrastructure*: embedded in tools, running locally, with quality and access (not tokens) as the new bottleneck.

---

## The argument, in layers

The essay is structured as a stack of independent efficiency fronts, which is what makes it more credible than a single extrapolation:

- **Hardware**: GPU power efficiency doubles roughly every two years — a Moore's-Law-class curve.
- **Models**: per-token prices at the frontier are *not* consistently falling, but per-task cost is (small models burn more tokens re-drafting). The pareto frontier moved ~100x cheaper across 2025, then another ~100x in 2026.
- **Engines**: vLLM, NVIDIA MLPerf, and Intel all show 10-50%/year software-side gains, with serving improving faster than offline batching.
- **Architecture**: MoE makes models 7x smaller at equal benchmark quality; Mamba-Transformer hybrids (Nemotron-H-47B: 1M+ tokens in 32 GB vs ~120 GB for a dense Llama-3.1-class model) attack local inference's real constraint — RAM.
- **Specialization**: Jev, a classifier that only picks between options, costs $42/billion tokens with free output — "3 cents to read 5 books."

## Key quotes

> The cost per *token* of models is not consistently going down, at least not for the smartest ("frontier") models. But the cost per *task* is.

A sharp and often-missed distinction. Most "tokens are getting cheaper" takes conflate the two; the per-task framing is the one that matters for anyone building agents, and it's the one that survives small models burning extra tokens on retries.

> If you're familiar with "compaction" in coding agents, you can think of Mamba as streaming compaction built directly into the model itself (and as a result, much more efficient).

A genuinely useful translation between architecture research and agent practice: the memory problems practitioners hack around in harnesses are being pushed into the weights.

> The hard part of software becomes product requirements, testing, and user-interface design, not algorithms. The job market gets really weird.

The demand-side speculation is where the essay is boldest and least evidenced — but the "fourth option" framing (use it / don't / use another / *tell an LLM to build it*) is a clean way to see why codebases stop being moats and operations and security become the value drivers.

## Themes

#concept (per-task vs per-token economics, Jevons paradox, induced demand) #tool (jgrep, Jev triage) #pattern (models embedded *in* tools once tokens undercut tool-call cost) #concept (optionality as the fourth software choice)

## Opinionated take

This is the strongest single-page summary of the *cost side* of the 2026 AI picture: the author separates variables that most commentary lumps together, cites per-chart sources, and hedges honestly (the Jev pricing note admits it may be subsidized). The weak joints are the extrapolations: 2.5 orders of magnitude/year is measured over one noisy year, and "frontier-quality local inference in 3-6 years" rests on the Mamba-hybrid curve continuing. The Jevons sections are assertion, not analysis — the essay notes the paradox exists without engaging with countervailing evidence (e.g. efficiency gains that did *not* induce demand). But the load-bearing claim — that quality and access, not token supply, become the constraint — matches what practitioners already see, and the "tokens cheaper than tool calls" table is the kind of Fermi estimate that changes how you design systems: it says the model belongs *inside* the loop of the tool, not beside it.

## Related pages

This essay gives the economic substrate under [[Performance per dollar is getting faster and cheaper]], which tracks the same cost-collapse from a practitioner's measurement angle; it strengthens the case there with the architecture-level mechanisms (MoE, Mamba) behind the trend. It complicates [[Inference Cost Napkin Math]]: napkin estimates framed around per-token pricing understate how fast the per-task frontier moves, and its tool-cost table is the same Fermi method applied to hardware rather than tokens. It nuances [[The Tokens You Can't Wait For]] — that piece argues from what models can't yet do; this one argues the price of fixing exactly those gaps is collapsing. And it connects to [[You Could Have Built Jev]]: Jev appears here as the extreme end of the cost curve, with the author treating specialized classifiers as proof that architectural headroom remains rather than as a one-off trick.

---
*Sources: [[raw/tokens-too-cheap-to-meter]], [[summary/tokens-too-cheap-to-meter]]*
*Last updated: 2026-09-25*
