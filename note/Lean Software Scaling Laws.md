# Lean Software Scaling Laws

Gwern's proposal to use LLM perplexity as a language-design benchmark: feed codebases in different languages through frozen models, measure how predictability scales with size, and use the resulting exponents as a weak but actionable proxy for which languages will be most LLM-compatible at production scale. The paper is a research proposal, not results — but it's one of the most elegant bridges yet between the formal methods world and the LLM engineering world.

---

## Key Quotes

> "Good design is invisible — not necessarily predictable at any given point, but increasingly predictable as you see more of the system. Bad code becomes increasingly unpredictable as scale grows."

This is the article's conceptual engine. Gwern sidesteps the failed "compressibility = quality" line of research by arguing that the *derivative* of predictability matters, not the absolute level. A well-typed, modular codebase teaches you its structure as you read more of it; a dynamic, monkey-patched one just accumulates gotchas. The insight resonates with [[Constraint Decay]]'s empirical finding that structural constraints systematically degrade LLM performance — except Gwern is asking whether languages *designed* for those constraints make the degradation shallower.

> "We could use LLMs to create large Lean codebases, to train on, so we can replace all our existing code with Lean. But we can't, because there are no large Lean codebases."

The chicken-and-egg problem stated cleanly. The bootstrapping solution — buy training data by paying humans or compute — treats code as a capital investment problem rather than a technical one. This reframes formal methods advocacy: stop arguing about correctness on principle and start calculating the ROI of training-data generation. The closest existing discussion in the wiki is [[The Coming Need for Formal Specification]], which makes the case that formal methods become economically rational when review costs dominate generation costs. Gwern adds the scaling dimension: even if formal methods are rational now, they may not be *practically usable* by LLMs until the training corpus crosses a threshold.

> "Weak languages like Python may be easy at small scales, but they'll cross over with strong languages like Haskell at hundreds of thousands of lines."

The crossover prediction is the testable heart of the proposal. If true, it means the LLM era doesn't just preserve the status quo of popular languages — it actively rewards language designs that were previously niche because their benefits only manifest at scales humans rarely reach. This inverts the usual "AI will cement Python's dominance" argument. It also provides cover for the [[Guardrails and Feedback Loops]] thesis: the right constraints, enforced by the right language, become a force multiplier rather than a drag.

> "The most likely failure mode is that ecosystem maturity and corpus prior effects dominate language-level invariants."

The author's honesty here is a feature, not a bug. The null result — tooling, conventions, and documentation beat formal language properties for LLM code generation — would itself be worth knowing. It would suggest that the path to better LLM-generated code runs through better linters, type-checkers, and conventions (the [[Guardrails and Feedback Loops]] approach) rather than language-level formalism.

---

## Key Themes

#scaling-laws #formal-verification #lean #programming-languages #llm-evaluation #language-design #perplexity #research-proposal

---

## The Measurement Design

The methodology is refreshingly concrete for a Gwern piece:

1. **Corpus construction**: Concatenate source files per language into single documents, dependency-sorted
2. **Forward-pass loss measurement**: Run frozen LLMs, measure per-token perplexity, normalize to bytes-per-character
3. **Cross-checks**: Inject subtle bugs (higher surprise = better), measure correctness under context constraints, ablate type signatures, identify unpredictability hot spots, find minimal context needed for understanding
4. **Curve-fitting**: Fit power-law scaling curves per language
5. **Extrapolation**: Find crossover points, examine constants vs. exponents

The bug-detection cross-check is particularly clever — it tests whether low perplexity means "the model understands the code" or just "the model has memorized similar boilerplate." A model that's genuinely surprised by a wrong unit conversion or a plausible-but-incorrect lemma is one that has semantic grip on the codebase.

The minimal-context test connects to the SAE truesight proposal: a well-designed modular system should require reading only small, specific pieces to understand any part. Gwern bets Lean would require much less context than Python for equivalent functionality, which is testable and would have direct implications for LLM context-window economics.

---

## Critical Analysis

**The good**: This is Gwern at his best — a falsifiable hypothesis, a tractable methodology, useful in both success and failure modes. The proposal doesn't require training models from scratch, doesn't depend on any single language's fortunes, and produces actionable output (crossover point estimates) for language investment decisions. The derivative-of-predictability framing elegantly sidesteps the gzip-compresses-JavaScript problem that sank earlier compressibility-as-quality work.

**The gap**: The proposal assumes "good language design" translates to "increasingly predictable to LLMs," but there's a confound it can't escape: LLMs are trained on human-written code, and human code quality varies wildly by language community. If Lean code is more predictable simply because only experts write Lean, you're measuring programmer selection, not language design. Gwern acknowledges this ("Lean programmers are highly unusual") and proposes matching-by-topic as a partial control, but the confound runs deep. A language that enforces good practices might produce uniformly mediocre code while a permissive language produces both brilliant and terrible code — the average predictability could be misleading in either direction.

**The unstated assumption**: The whole framework assumes LLM predictability is a good proxy for what we actually want (correct, secure, maintainable software). But predictability and correctness are different things. A codebase full of verbose, repetitive boilerplate might be highly predictable while being terrible software. Conversely, a codebase that does something genuinely novel might be unpredictable while being excellent. The bug-detection cross-check partially addresses this, but the gap between "the model expected this token" and "this token is correct" remains.

**The bootstrapping economics**: The article's most provocative claim is that crossover predictions could justify *paying for training data*. If Lean's exponent is demonstrably better, organizations could calculate the ROI of commissioning Lean rewrites of their codebases — not because Lean is better for humans, but because Lean-trained LLMs will be better at maintaining the code later. This inverts the normal adoption argument: language choice becomes a bet on LLM scaling, not human productivity. The wiki has touched on adjacent ideas — [[Specifications as the Product]] treats specs as the durable asset, and [[Lean Software Production]] (different Lean!) treats the production system as the product — but Gwern is proposing something more radical: language-as-investment-vehicle for future LLM capability.

**What's missing**: The proposal doesn't engage with the possibility that LLMs might get good enough at formal verification to make the language choice moot. If an LLM can read your Python codebase and prove it correct, the scaling exponent of Python vs. Lean stops mattering. Gwern implicitly assumes LLMs will always struggle more with weaker languages, but that's an assumption that itself needs testing.

---

## Connections

- [[The Coming Need for Formal Specification]] — Congdon's argument that formal methods become economically rational when LLMs invert the code/review cost ratio. Gwern adds the scaling dimension: formal methods aren't just cheaper to review, they may be fundamentally more LLM-compatible at scale.
- [[Constraint Decay]] — Dente et al.'s empirical finding that database constraints cause 30pp drops in LLM pass rates. Gwern's framework predicts this decay should be *shallower* in languages where the constraint is structural rather than bolted-on. Testable.
- [[Guardrails and Feedback Loops]] — The linters-beat-prompts thesis. Gwern's null result (ecosystem beats language) would be evidence for the guardrails approach over the formalism approach.
- [[Specifications as the Product]] — If code is disposable and specs are durable, Gwern's proposal adds: the language you write specs *in* matters for LLM predictability, and some languages are better spec languages than others.
- [[The Reasoning Trap]] — Yin et al.'s finding that reasoning amplifies tool hallucination. Gwern's Lean bet is that strong types and proofs constrain the hallucination space enough to make reasoning safe — but "mathslop" (plausible-but-wrong proofs) is the Lean-specific failure mode that mirrors the reasoning trap's hallucination problem.

---
*Sources: [[raw/lean-scaling]]*
*Last updated: 2026-07-08*
