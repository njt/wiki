---
url: https://softwaredoug.com/blog/2026/10/08/bag-of-decisions.html
title: "Bag of Decisions"
author: Doug Turnbull
date_fetched: 2026-10-12
date_published: 2026-10-08
topics:
  - ai-research-and-models
  - databases-and-data
---

Doug Turnbull, search-relevance veteran and co-author of *AI-Powered Search*, proposes reframing reranking: instead of asking a decision model one question — "is this document relevant for this query?" — have an LLM first generate a rubric of ~10 yes/no relevance criteria specific to the query, then score the document against each criterion separately and sum the probabilities. He calls the result a "bag of decisions," inverting the classic IR notion of a query as a "bag of terms."

The mechanics use Jev (TypeSafe's System One decision model) in a multi-question flow: for `desk for kids`, the LLM produces questions about child-sized ergonomics, buying guides, age ranges, brands, pricing, and so on; each gets its own P(yes) from the decision model; the sum becomes the rerank score, usable alone or added to BM25.

On two e-commerce datasets (Wayfair WANDS, Amazon ESCI) the approach beats both BM25 and single-question Jev reranking — on WANDS 0.610 vs 0.574 vs 0.5407 mean NDCG, and on ESCI 0.360 vs 0.314 vs 0.289, with the biggest gains on the harder dataset. He frames it as *per-query feature creation*: the LLM supplies inspiration about what matters for this particular query, the decision model measures coverage. He notes the generated rubrics are somewhat repetitive and could be improved, and pitches the idea as a promising toy experiment worth deeper study, possibly with domain-specific decision models.
