# Bag of Decisions (Doug Turnbull)

Doug Turnbull's playful reranking experiment: replace the single relevance verdict "is this document relevant for this query?" with an LLM-generated rubric of ~10 yes/no relevance questions scored individually by the Jev decision model, then sum the probabilities as the rerank score. On two e-commerce benchmarks it beats both BM25 and ordinary Jev reranking, turning query understanding into per-query feature creation.

---

## The idea

Classical IR treats a query as a bag of terms. Turnbull inverts it into a **bag of decisions**: decompose the *intent* of the query into concrete, verifiable criteria. For `desk for kids`, an LLM generates questions like "does it mention child-sized or age-appropriate desk dimensions or ergonomics?", "does it include buying guides, reviews, or comparisons?", "does it provide pricing or availability?". Each criterion is scored by Jev's P(yes); the sum is the relevance score, which can be added to BM25 or used alone.

This is an elegant reframing of what a reranker does. A single "is this relevant?" question forces the decision model to compress an entire notion of relevance into one probability. The rubric version outsources the *decomposition* of relevance to the LLM and keeps only the *measurement* with the decision model — a clean division of labor between a generative and a discriminative component.

## Results

| Dataset | Strategy | Mean NDCG |
|---|---|---|
| WANDS | bm25 | 0.5407 |
| WANDS | jev_reranker | 0.574 |
| WANDS | bag_of_decisions | 0.610 |
| ESCI | bm25 | 0.289 |
| ESCI | jev_reranker | 0.314 |
| ESCI | bag_of_decisions | 0.360 |

The ESCI jump is the interesting one: the gap between single-question reranking and the rubric widens on the harder dataset, suggesting decomposition pays off most where "relevance" is genuinely ambiguous.

## Key quotes

> "We're essentially doing per-query feature creation. Use an LLM as inspiration of what's important to consider for *this query*. Then use Jev to measure whether we're covering that criteria."

The honest one-line summary of the technique. It also quietly concedes the division of labor: the LLM doesn't judge relevance, it only *names dimensions* of relevance — the cheap, fuzzy part — while the calibrated classifier does the scoring.

> "(IMO some are repetitive)"

Turnbull flags his own rubrics' redundancy, which matters more than he lets on: if 10 questions effectively measure 4 dimensions, the sum is a weighted vote with arbitrary weights, and "summing probabilities" is a rough proxy for a real scoring function. A smarter rubric generator (diverse, non-overlapping criteria) is where most of the remaining headroom probably lives.

## Key themes

#concept #tool #pattern

- **Rubrics as query representation** — relevance becomes a checklist instead of a vibe.
- **Generative-decompose, discriminative-score** — LLM proposes criteria, decision model measures them.
- **Ensembling via decomposition** — averaging over many narrow questions instead of trusting one broad judgment.

## Critical take

This is a toy experiment, and Turnbull says so — no cost/latency accounting, no significance testing, two datasets, a self-admittedly repetitive rubric generator. Ten Jev calls per document per query is not free. But the direction is sound and connects to a broader pattern in this wiki: narrow, cheap decision models compose better than one big judgment when the problem can be factored. It also gives a concrete answer to [[Jev Can't Be Calibrated]]'s complaint — if you only ever *sum* P(yes) values for ranking, calibration barely matters, only ordering does, which is exactly the "treat outputs as ranking scores" prescription Molas arrived at from the theory side.

The risk is rubric collapse: the LLM's criteria are only as diverse as its imagination for the query, and repetitive questions systematically double-count whatever the LLM happens to think of first. Rubric diversity, not question count, is the knob that matters.

## Related pages

- Strengthens [[Jev Can't Be Calibrated]]'s practical advice: this pipeline only consumes summed probabilities for ordering, sidestepping the calibration problem entirely.
- Extends [[Hobson — Brooker's Home-Built Jev-Class Classifier]]'s picture of the Jev-class model ecosystem with a novel composition pattern — many O(1)-cost decision calls per document, which Hobson's cached-torso architecture makes cheap.
- Nuances [[Scaling RAG — Chunking, Reranking, and Cost Optimization]]: rerank scoring doesn't have to be a single model call; the scoring function itself can be generated per query.
- Complements [[What Is RLCD — The Secret Behind Jev]] by showing another consumer of decision models beyond direct classification — rubric scoring over generated criteria.

---
*Sources: [[raw/bag-of-decisions-html]], [[summary/bag-of-decisions-html]]*
*Last updated: 2026-10-12*
