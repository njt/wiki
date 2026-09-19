# LLM Judge Decision Tree

Doug Turnbull measures whether a local LLM on a laptop can act as a search-relevance judge, comparing its pairwise product preferences against human labels on the WANDS e-commerce dataset. His answer is architectural rather than modelological: run many deliberately dumb single-attribute judgments, let the judge abstain ("Neither") and double-check itself by swapping LHS/RHS, then stop treating the LLM as the decision-maker at all — feed its judgments into a plain scikit-learn decision tree as *features*, with the human label as the target. The LLM becomes a cheap, cacheable feature generator; "fast boring, old ML" finishes the job.

---

## Key Quotes

> "My goal, not so much to replace other labels but to at least be a reliable to flag what looks amiss / promising much faster without needing to always recruit humans."

The right-sized ambition, and the most quotable sentence in the piece. An LLM judge pitched as a *replacement* for human raters fails the moment its precision is examined; pitched as a triage filter that tells you where to spend scarce human attention, 90% precision on 12% of pairs is genuinely useful. The typo-ridden sentence is also pure lab-notebook Turnbull.

> "When the agent chickens-out (or isn't consistent in double checking) we improve precision, but lose recall."

The whole measurement story in one line. Abstention ("Neither") and swap-consistency checks are both forms of the model saying "I don't know" — and the tables show exactly what that reluctance buys: product-name judgments go from 75.08% precision at 100% coverage to 90.76% at 11.9%. Calibration as a dial, not a property.

> "The really astute reader will notice something. It's a Machine Learning problem!"

The pivot, delivered with relish. Once you have a table of per-attribute LLM judgments and human labels, you don't need a cleverer prompt or a weighted vote — you have features and a label. The ensemble isn't designed; it's *trained*.

> "Local LLMs could become ML feature generators, maybe heavily cachable ones at that! We keep their decisions dumb, simple, and interpretable combining them at the end with fast boring, old ML that can finish the job."

The thesis, and the reason this post outlives its dataset. Two claims worth savoring: small local judgments are cacheable (same query + same attribute = same answer, no reason to re-run), and dumbness is a *feature* — an interpretable LHS/RHS/Neither per attribute is something a tree can consume and a human can audit, unlike a 2,000-token judge rationale.

> "(all assuming human evaluators are any good at this!)"

A parenthesis doing the work of a chapter. The entire edifice is calibrated against human pairwise preference labels, which are themselves noisy. Turnbull waves at the problem and moves on; everyone building on human-labeled evals should wave more nervously.

## Key Themes

- **#concept — LLM as feature generator.** The inversion that makes the piece memorable: the LLM's job is not to decide but to produce labeled, composable evidence for a downstream classifier. This runs directly against the era's dominant "uber prompt" instinct — and tellingly, his own uber prompt (91.72% precision at 65.2% coverage) is competitive, so the tree's win is at the precision extremes, not everywhere.
- **#pattern — Abstention and self-consistency as calibration.** "Neither" as a first-class output plus the LHS/RHS swap re-ask is a cheap, pre-logprob-era ensemble of one model with itself. Reliability comes from the judgment's *willingness to not answer*, not from its accuracy when forced.
- **#pattern — Old ML as the combinator.** A decision tree with a probability threshold (0.9) is the precision/recall dial on top of the whole stack. The tree doubles as an exploratory instrument: its splits ("category before name") read as hypotheses about which signals a search solution should invest in.
- **#person — Doug Turnbull.** The search-relevance veteran's instinct on display: measure against human labels, distrust your own ensemble designs, and treat results as "an interesting data point in a scientific lab notebook, not a guarantee."

## Critical Analysis

The keeper is the inversion. Most LLM-as-judge work in 2025 raced toward bigger prompts, richer rubrics, and stronger judge models. Turnbull goes the other way: decompose into judgments so small they're boring, then reassemble with machinery designed for exactly this shape of problem. It's the same instinct behind classic learning-to-rank — and it means the judge stack inherits ML's honest toolkit: precision/recall curves, thresholds, train/test splits, cross-validation. The piece is also unusually honest about its own epistemics, flagging that the numbers come from a single 1,000-pair test split without cross-validation.

It deserves skepticism in three places. First, the decision tree doesn't clearly dominate its own strawman: the four-field uber prompt with double-checking hits 91.72% precision at 65.2% coverage, while the tree's best operating points are either extreme (100% precision at 1.3% coverage — nearly useless in practice) or 96.5% at 40.5%. The tree's real advantages are the tunable threshold and the interpretable splits, not raw supremacy, and the post slightly oversells the framing over the numbers. Second, the tree-as-explanation move ("focus on category first") is suggestive but shaky — the exported thresholds are artifacts of how LHS/RHS/Neither were encoded (−1/0/+1), not discovered laws of relevance. Third, the human-label foundation is acknowledged only parenthetically: pairwise relevance judgments are noisy, and a judge calibrated to noisy labels inherits their biases with extra steps.

Minor errata mark the lab-notebook character: the uber-prompt example mislabels an RHS line as LHS, "caveants" appears in place of caveats, and the bash collection script is charmingly literal about stressing "my laptops heat disapation capabilities". None of it undermines the measurement; all of it signals this is a working scientist's notebook, not a polished framework announcement.

Dated January 2025, this post now reads as an early, practitioner-grade articulation of ideas the wiki has seen mature elsewhere: judge calibration against human agreement, abstention as a precision lever, and verification as a system distinct from generation.

---

## Connections

- [[Eval-Driven Development (Airbnb)]] — Same discipline, different mechanism: Airbnb calibrates LLM judges against human labels with Cohen's kappa and rubric iteration, while Turnbull calibrates by measuring precision/coverage across judgment variants. His abstention-and-double-check tables are a pairwise-preference flavour of Airbnb's "a judge that hasn't been calibrated is worse than no judge at all."
- [[The Lifecycle of LLM-as-a-Judge]] — Netflix industrializes what Turnbull prototypes: human labels as ground truth, agreement measurement, and drift monitoring. His "(all assuming human evaluators are any good at this!)" aside is the raw nerve Netflix's ±2σ-of-human-raters band is built to manage.
- [[AI Team Mistakes]] — Same author, same doctrine, seven months later: this post is the lab-bench execution of the "spend ~50% of investment on understanding the problem" thesis, showing what evals-first looks like when the eval target is the judge itself.
- [[Fine-Tuning a Local LLM to Categorize Questions]] — Complementary answers to the same question — how to make a small local model trustworthy enough to pipe downstream. Helgevold fine-tunes a 600M model into a single reliable classifier with opaque output codes; Turnbull keeps the model off-the-shelf and dumb, and pushes reliability into a decision tree above it. Both constrain the output space (LHS/RHS/Neither ≈ two-letter codes) so failures stay machine-consumable.

---
*Sources: [[raw/llm-judge-decision-tree]], [[summary/llm-judge-decision-tree]]*
*Last updated: 2026-09-19*
