# RUBICON

RUBICON is a VLDB paper that takes the most damning finding in enterprise text-to-SQL — that LLMs can't reliably assemble a correct multi-source query even when handed all the ingredients — and turns it into an architecture. Instead of entrusting the whole workflow to a frontier LLM's reasoning loop, RUBICON constrains the LLM to a "baby SQL" subset (AQL), wraps every source behind an adapter that exposes it as a logical *table*, and lets a relational query optimizer do the integration. On a benchmark modeled on a Munich Department of Transportation deployment, RUBICON scores 100% end-to-end while every agentic baseline (vanilla, single-agent ReAct, multi-agent LangGraph) scores zero.

---

## Key Quotes

> "text-to-SQL fails on real enterprise data and must be dramatically subsetted to achieve reliable results."

The paper's first load-bearing claim, and it's the direct continuation of [[BEAVER]]: public benchmarks (Spider, BIRD) report 80%+; the real-world Beaver benchmark reports ~10%, rising to ~30% only when the correct tables and join terms are handed over. RUBICON's response is not better prompting — it's to shrink the surface the LLM is asked to reason over, to "baby SQL."

> "data integration across disparate corporate datasets is best performed using tables as the core abstraction rather than the text-centric pipelines of current LLMs."

This is the deeper bet, and the more original one. An insurance underwriting app must reconcile flood-zone maps, building CAD drawings, claims tables, and underwriting notes. RUBICON's argument: flattening all of that to text so an LLM can integrate it is a doomed idea; converting each source to a table and joining the tables is what databases have done for fifty years.

> "RUBICON achieves 100% end-to-end accuracy, while all agentic baselines — including single- and multi-agent ReAct systems — produce no correct answers."

The headline number. End-to-end accuracy is all-or-nothing (no partial credit for calling the right source). Every ReAct baseline scores zero on the natural-language variant. The multi-agent system, given more autonomy and a full view of the source landscape, just burns its call budget exploring — more autonomy makes things *worse*, not better.

> "The important point is not syntactic novelty; rather it is allocation of responsibility."

The design's heart. AQL's `FIND … FROM … WHERE` with a natural-language predicate looks trivial, but the point is who owns what: the human or interface specifies sources, fields, and intermediate objects; the wrapper or LLM handles only the *local* ambiguity inside the predicate. That narrows LLM use to something inspectable and replaceable.

> "Retrieval is not the step before query processing. Retrieval is one operator inside query processing."

RUBICON's answer to the RAG-era default. Enterprise copilots ingest-and-index, which improves discoverability but never integrates the underlying data models — "a text index is not a join engine." RUBICON instead follows mediator-wrapper data integration: sources stay in place, wrappers expose logical relations, and integration happens at query time, where cost-aware optimization (pushdown, join reordering, source selection) applies to tokens and tool calls, not just runtime.

> "Discarding the structure in structured data so it can be combined with text in a text-oriented system just throws away semantics."

The authors' closing jab at the hybrid agentic future. Structured data is structured for a reason; dissolving it into prose to fit a text-centric pipeline gives away the very thing that makes it reliable.

---

## Key Themes

- **#concept — Tables, not text, as the integration abstraction.** The whole paper is an argument that the relational model should be the substrate for enterprise agentic AI, and that text is a lossy intermediate representation. This is the same instinct as [[Layer-First Pattern — Keep Data Out of the LLM Context]]: don't run data through the LLM unless the LLM is making a decision about it.

- **#pattern — Mediator-wrapper over ingestion-and-indexing.** RUBICON deliberately *doesn't* crawl, extract, and index enterprise sources. It leaves them in place and wraps them, so access control is enforced source-locally and joins happen at query time. The contrast with [[Inside OpenAI's In-House Data Agent]] — six context layers to make a conversational agent navigate 70k datasets — is stark.

- **#concept — Constrained interface beats constrained reasoning.** RUBICON's bet is that reliability comes from *restricting what the model is allowed to say* (AQL's fixed structure) rather than from reasoning harder. It rhymes with [[DeepSQL]]'s correctness-over-autonomy, but reaches it from the opposite direction: DeepSQL lets the LLM draft full SQL and fences it with deterministic validation; RUBICON doesn't let the LLM draft SQL at all — it fills in predicates inside a plan the human or query processor owns.

- **#pattern — Human in the loop for plan correction.** AQL's SAVE, `?`, and inspectable intermediate results exist so a human can catch a search going "off the rails" and correct the plan. This is the plan-level analogue of [[AI-Assisted Database Work — The Machine Reads, The Human Decides]]'s machine-reads-human-decides boundary.

- **#tool — AQL (Agentic Query Language).** The concrete artifact, plus the RUBICON-Bench and the SemBench comparison against LOTUS and Palimpzest.

---

## Critical Analysis

**This is the paper that resolves the dangling pointer in [[Text-to-SQL in the Real World]].** Stonebraker's blog name-dropped Rubicon as "our own ideas on how to do better" without explaining it; this is the explanation. It lands squarely in the lineage Stonebraker's name lends weight to — mediator-wrapper integration, query processing, "don't let the LLM assemble the query." The authors are the Binnig group (TU Darmstadt, with Munich collaboration), so it's the academic database community's answer, and it reads like one: the novelty claim is explicitly *not* syntactic, and the architecture is forty-year-old database machinery re-pointed at LLMs.

**The 100% vs 0% result is dramatic but needs the same skepticism the paper itself applies to benchmarks.** RUBICON-Bench is a seven-query workload, hand-built by the system's authors, with each query requiring exactly two of five sources. That's a fair test of the *specific* skill the paper cares about — cross-source join identification — but it's a small, controlled benchmark, and 100% on it is closer to "we built the benchmark to match our system" than to "solved." The authors are candid that the Munich production data can't be shared yet, and that the real deployment is years out. The SemBench comparison is more independent, and there the wins are efficiency (62% latency, 98% token cost) as much as accuracy (14.7%).

**The most consequential claim is also the most fragile: that AQL doesn't shift the hard part, it relocates it.** RUBICON offloads structural reasoning — which source, which fields, which join — from the LLM onto the *human or interface*. In NL mode, the paper admits RUBICON "does not produce good results" when asked to compile a whole task into AQL autonomously. So the honest reading is: RUBICON doesn't solve multi-source *planning*; it requires a human (or a future system) to provide the plan, and then executes it reliably. That's a real and useful division of labor — and it's exactly why the human-in-the-loop is a core concept, not an afterthought — but it means the headline "100% vs 0%" is comparing a system *given* the plan against systems *finding* the plan. The paper is upfront about this, and the AQL-guided agentic baselines (multi-agent AQL hits 28.57% end-to-end) show the same inputs help everyone.

**The efficiency numbers are the quietest and most persuasive part.** 98.64% lower token cost and 62.64% lower latency against LOTUS and Palimpzest aren't about accuracy — they're about the paper's second thesis, that text-as-representation is *expensive*. If you're running agentic AI against a real data estate, the cost argument may matter more than the accuracy argument, and it's the part most likely to age well.

**For the wiki, RUBICON closes a loop.** [[BEAVER]] measured the failure; [[Text-to-SQL in the Real World]] publicized it; [[DeepSQL]] shipped a production answer (fenced drafting); RUBICON supplies the academic architecture (don't draft at all — plan, then execute). Together they mark the boundary of what LLMs can reliably do with enterprise data: read it and fill in local predicates, not assemble it into correct multi-source queries.

---

## Connections

- [[Text-to-SQL in the Real World]] — the article that name-dropped Rubicon; this paper is the explanation of that hint, and the two share the Beaver diagnosis.
- [[BEAVER]] — the benchmark whose ~10% / ~30% numbers RUBICON cites as the reason "baby SQL" is necessary.
- [[DeepSQL]] — the production counterpoint: DeepSQL lets the LLM draft SQL and fences it; RUBICON doesn't let the LLM draft at all. Both answer the same failure, from opposite directions.
- [[Inside OpenAI's In-House Data Agent]] — the architectural opposite: OpenAI doubled down on a conversational agent over six context layers; RUBICON argues the conversational, text-centric approach is the wrong abstraction for messy enterprise data.
- [[Layer-First Pattern — Keep Data Out of the LLM Context]] — shared principle: don't run data through the LLM unless the LLM is deciding about it.
- [[AI-Assisted Database Work — The Machine Reads, The Human Decides]] — the human-in-the-loop boundary, applied to plan correction rather than artifact reading.

---

*Sources: [[raw/2604-21413v2]], [[summary/2604-21413v2]]*
*Last updated: 2026-08-25*
