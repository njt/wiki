# System One Models and Jev

TypeSafe AI's launch announcement (Diogo Almeida, ex-OpenAI) introduces "System One Models" — a model class that abandons autoregressive string generation entirely and instead emits calibrated, typed probabilistic decisions in parallel. The first model, Jev, claims LLM-level intelligence on decision tasks at ~100× the speed and ~100–400× lower cost, with schema-guaranteed outputs and per-decision confidence. This is the most aggressive productization yet of the idea that most software-facing AI work is decision-making, not prose.

---

## The Pitch

Almeida's origin story is the chat-model gap: he helped build the RLHF-era methods behind ChatGPT, concluded that chat was superhuman but automation remained absent, and spent four years on the missing piece. The answer is a stack built for decisions: new architecture, parallel sampler, and a training method called **Reinforcement Learning for Calibrated Decisions (RLCD)** — RL optimizing for epistemically honest probabilities rather than preferred prose or verifiable rewards.

> "Think of Jev as a frontier-intelligence function call: unstructured state in, typed probabilistic decisions out."

The one-line thesis, and the most useful way to remember the product. It takes the function-calling interface every agent already uses and deletes the generative model behind it.

> "While Jev gives up string generation, it's optimized for structured outputs and *can't* hallucinate."

"Giving up" strings is framed as a superpower trade: strings are general but costly; decisions are narrow but cheap, parallel, and type-safe by construction.

## Key Quotes

> "Even if prompted for a confidence estimate, models tend to be overconfident and inconsistent. If a model can do a task 95% of the time but doesn't say when it's in the 5%, it can't automate that task."

The sharpest formulation in the piece, and the correct one: automation is gated by calibration, not capability. A model that is right 95% of the time but wrong *confidently* is unusable in anything with latency guarantees or downstream dependency — you cannot branch on an answer you can't trust at the 5%.

> "Having a hallucinated tool call is inconvenient in an agent, but is an absolute deal-breaker if it's part of a system with latency guarantees or it's buried several layers deep in a dependency chain."

The claim "can't hallucinate" is carefully scoped: what's guaranteed is **schema conformance**, not correctness. Almeida admits the fine print — "Our number is not empirical. Schema matching is guaranteed, thus we can confidently add 0% into the plots." A confidently wrong typed decision is still wrong; calibration is the load-bearing claim, and it is exactly what RLCD exists to buy.

> "Every order of magnitude drop in the cost of intelligence unlocks orders of magnitude more use cases."

The Jevons-paradox naming is the business thesis made explicit: Jev (for William Stanley Jevons) bets that cheap decisions expand demand the way efficient steam engines expanded coal consumption. Efficiency is the growth engine, not the threat.

> "The most reliable real-world workflows tend to have many independent, decomposed questions, with fine-grained behavior that's dependent on probabilities instead of discrete decisions."

A workflow-design insight hiding in an eval section: reliable automation decomposes into many small calibrated judgments whose *probabilities*, not just argmax, feed the branching logic.

> "We started TypeSafe because we believe that AI needs an interface software could depend on."

The mission statement, aimed squarely at the gap between demos and production dependencies.

## What the Evidence Actually Shows

The receipt sections are unusually self-aware for a launch post, and the honest disclosures matter as much as the claims:

- **Workflow evals** are a genuinely interesting methodology: every model runs the same fixed code workflow (no harness tuning, no ground-truth labels), scored against the *average of the largest external models* (GPT-6 Astra and Fable 5.1). Jev claims the Pareto frontier for nearly two orders of magnitude, with 193.6× faster / 444.6× cheaper as the headline numbers — which the authors themselves call "on the higher end of real world gains."
- Disclosed biases: eval workflows were authored by their own capabilities team; the reference average biases toward OpenAI/Anthropic; LLM baselines run through TypeSafe's own structured-output wrapper (slower and pricier than bare generation).
- The Doom demo (10 queries/sec, ~$7/hour) and wikiracing demos showcase intelligence-per-second; the wikiracing nuance admits speedups shrink badly against non-reasoning baselines, and that Jev caps cardinality at 255 with a two-stage scoring fallback.
- Evals were run "from our laptops on the West Coast"; pricing sustainability "can't be proven" isn't subsidized.

## Key Themes

- #concept — **System 1 vs System 2 as a division of labor**: Kahneman's framework inverted; the company wants fast intuition with honest uncertainty, and argues "System 1 thinking" need not mean error-prone
- #concept — **Calibration as the automation gate**: the 95%-who-don't-know-they're-in-the-5% argument
- #pattern — **Decisions, not strings**: delete generation, keep judgment; parallel probability outputs as an API surface
- #tool — Jev and the TypeSafe API, in early access
- #person — Diogo Almeida, founder

## Opinionated Take

The architecture argument is sound and, frankly, overdue. Everything an autoregressive LLM does at inference is overhead if all you need is a decision; this is the same logic that made classifier heads and embedding models cheap, applied to the frontier. If the speed/cost numbers hold up at even half the claimed magnitude, this is a significant primitives shift — and it lands exactly where the wiki's [[Smart Models Dumb Pipes]] thesis predicted: judgment at the edges, deterministic plumbing everywhere else.

But the marketing sentence and the engineering claim are different animals. "Can't hallucinate" is definitional (no strings → no schema violations), while the thing practitioners actually fear — a plausible but wrong decision — is addressed only by the calibration claim, which is *exactly* the property LLMs are empirically worst at and the one RLCD's evidence here does not directly demonstrate. The workflow evals are honest about reference-model bias, but scoring against the average of the two largest frontier models is a circularity risk: a genuinely better-but-different distribution of judgments reads as disagreement, not superiority. And the eval-to-production gap is acknowledged only in passing ("on the higher end of real world gains").

The FAQ section in the fetched copy poses the right hostile questions ("Is Jev just a smaller LLM?", "How is it possible?") and answers none of them — collapsed accordions, or promises of future posts. That's the epistemics of a launch announcement, not a paper. Watch this one for independent replication, not for its own receipts.

## Related Pages

- [[Smart Models Dumb Pipes]] — McCormick's "smart things handle judgment, dumb things handle execution" is this product's design diagram; Jev is what you get when you take the principle literally and delete the pipe's string traffic entirely.
- [[OpenAI Structured Outputs]] — OpenAI guarantees schema conformance at the protocol layer over a generative model; TypeSafe moves the guarantee into the architecture itself. The two mark the spectrum from constrained generation to generation-free decisions.
- [[The Reasoning Trap]] — Yin et al. show better reasoning amplifies tool hallucination; Jev's answer is architectural abstention — don't generate, decide — which sidesteps rather than solves the failure mode, and only for the decision-shaped slice of it.
- [[Model Routing Is Simple Until It Isn't]] — IBM's finding that routing is systems optimization, not classification, is the deployment caution Jev's pitch skips: a calibrated decision primitive still lives inside cache economics, infrastructure state, and workload patterns that dominate real-world cost and latency.

---
*Sources: [[raw/introducing-system-one-models-and-jev]], [[summary/introducing-system-one-models-and-jev]]*
*Last updated: 2026-09-19*
