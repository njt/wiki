# Context as Infrastructure (Knowledge Compilation for Agentic Retrieval)

A RAGFlow-team arXiv paper (October 2026) arguing that agentic retrieval has saturated accuracy — a BM25+grep harness with no vector index hits 91% on FRAMES — so the marginal value of a retrieval layer has moved from *finding* to *supplying context*. Its proposal: an ingestion-time "knowledge compilation" pipeline producing typed, provenance-bound artifacts, of which claim–evidence pairs are the one mechanism with a measured effect (a 38% average token cut, bought partly with latency). Structural compilation (trees, wikis, graphs) is measured and found accuracy-neutral or worse.

---

## What it argues

- **Retrieval is a harness problem before it is an index problem.** The index determines what can be found; the harness determines whether finding becomes an answer. A better index cannot fix a loop that never discards a bad query or reads without a budget.
- **The dense-retrieval leg is not neutral.** Inside a harness, the agent performs the semantic step by reformulating queries; adding a dense leg "lowers accuracy in our runs" while costing a second index to build and refresh.
- **Compilation is a supply layer, not a retriever.** Six replaceable stages (parse, chunk, enrich, validate, assemble, register) produce typed artifacts — the chunk is "a unit of recall, not a unit of meaning," and chunking stops being the tuning problem it was in naive RAG.
- **Structure is not evidence.** Navigation trees, wikis, and graphs re-describe what the agent can already grep; they add no facts and carry extraction-error compounding.
- **The harness is discipline, not intelligence.** A five-station loop with invariants the framework — not the model — owns.

## Key quotes

> "A better index cannot fix a loop that never discards a bad query, reads without a budget, or keeps a passage because it was expensive to fetch."

The paper's thesis in one line, and a useful inversion of where RAG engineering effort has gone for years.

> "The chunk is a unit of recall, not a unit of meaning: the impossible tuning problem in which every corpus needs its own chunk size does not survive the harness."

This quietly kills an entire genre of RAG content — per-corpus chunk-size tuning — by relocating the semantic step from the index to the agent's reformulation loop.

> "The model may propose, but the source disposes."

On validation: every claim's evidence span is checked for exact presence in the source, and unmatched claims are dropped. This is why the extractor can be a small cheap model (glm-4-flash) — deterministic validation, not model size, protects quality.

> "Compilation pays only when the horizon is long enough... a pilot with [few] queries needs a much higher sufficiency rate, not 2%."

The finite-horizon boundary (Eq. 9) is the rare honest piece of economics in the retrieval literature: the asymptotic 2% break-even is routinely misread as a license to compile everything.

> "A zero-hit result is a property of the tool, not a fact about the corpus."

A small protocol with outsized value: the harness explains and retries along a fallback ladder before the model may conclude the corpus is silent — preventing the most common false negative in agentic retrieval.

> "We publish the ledger as a protocol with an empty result column rather than filling it with direction claims."

The deployment ledger for ontologies/graphs/wikis/timelines/memory has *no observations yet* — every row marked "no measured effect." Unusual candor, and a reproach to the marketing-driven artifact literature.

## Key themes

#concept — knowledge compilation, claim–evidence pairs, agent-native database
#pattern — harness invariants (evidence before thought, discard on insufficiency, citations at delivery, guaranteed close)
#tool — RAGFlow, the implementation of all of it
#comparison — structure vs. evidence vs. discipline, each judged on its own ledger

## Opinionated take

This is the most intellectually honest retrieval paper in recent memory, and its negative results are the contribution. The field has spent two years selling RAPTOR trees, PageIndex outlines, and GraphRAG community reports as accuracy wins; this team built all of them, exposed them to a competent harness, and found none of them moved accuracy — with a compounding-error explanation for why PageIndex actively rots at scale. The counterfactual question they ask — *given that the harness retrieves, what should the model read?* — is the right one, and it reframes RAGFlow's whole artifact zoo as different ledgers rather than a leaderboard.

Two caveats worth holding. First, the headline token economics (–91% hit / +1.9% miss) are single-run operating points with no confidence intervals, by the authors' own admission, and the latency tripled on a seven-question paired subset — the direction is credible, the magnitudes are soft. Second, the "agent-native database" framing is doing marketing work: the substance is data supply with provenance, and calling it a database invites the same category confusion the paper itself warns against ("it is not a relational database... and not a new retrieval paradigm").

The deepest implication is for anything that ingests documents for agents: the derived/raw channel separation rule — derived artifacts are read-only, typed, never written back into the source — is the guard against the most common LLM-corpus failure, a generated summary becoming the source for the next generation. That principle generalises far beyond retrieval.

## Related pages

- [[GraphRAG]] — the paper deliberately rejects GraphRAG-style community reports ("fragment knowledge and add a token-heavy layer of generated text"), keeping only a PageRank-over-graph recall aid; it complicates that page's framing of graphs as a RAG accuracy fix.
- [[What Is a Context Graph]] — both argue the durable artifact is provenance-bearing context compiled at write time, not search over raw text; this paper supplies the measured cost ledger that page lacks.
- [[Three Kinds of Agentic Search]] — this paper's five-station harness is a concretised, production version of the search-harness pattern that page describes, with the addition that tool choice is *not* left to the model.
- [[LLM Wiki]] — Karpathy's LLM-wiki pattern is the paper's namesake for compilation, but the measured verdict (accuracy-neutral, lossy provenance, expensive to rebuild) deflates it from a retrieval strategy to a consumption layer.

---
*Sources: [[raw/2610-41172v1]], [[summary/2610-41172v1]]*
*Last updated: 2026-11-02*
