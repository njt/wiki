# BEAVER: An Enterprise Benchmark for Text-to-SQL

The first text-to-SQL benchmark sourced from real private enterprise data warehouses -- and it humiliates every current approach. 9,128 queries across 812 tables in 19 domains. GPT-5.2 with the best agentic framework gets 10.8%. With all oracle hints: 30.1%. The gap between public-benchmark performance (82% on BIRD) and enterprise reality is a chasm, and BEAVER is the first honest measurement of it.

---

## Key Quotes

> "SOTA agentic frameworks using the advanced model GPT-5.2 achieve only 10.8% accuracy. Even with all subtask annotations as oracle hints, accuracy increases to only 30.1%."

The lede is the finding. This isn't a marginal improvement over existing benchmarks -- it's a category error. Public benchmarks have been measuring the wrong thing.

> "Existing all-or-nothing evaluation metrics based on accuracy make error diagnosis difficult."

The paper's real contribution is decomposing the monolithic "did it work?" into five subtasks that each have their own metric. This is [[Demystifying Evals for AI Agents]] applied to a specific domain: stop asking "is the SQL correct?" and start asking *which part* of the SQL generation pipeline broke.

> "Even with all oracle hints, execution accuracy maxes at 30.1% and subtask metrics fall short of perfection."

The most damning sentence in the paper. They gave the models the answers -- told them exactly which tables, which join keys, which column mappings, which domain facts, and how to decompose the query -- and the best system still got it wrong 70% of the time. The bottleneck is not retrieval. It's reasoning about analytical structure.

> "Domain-specific complex queries: average execution accuracy is only 5.4%."

The combined category (complex SQL + domain knowledge) is where real enterprise queries live, and it's effectively unsolved. 5.4% isn't "room for improvement" -- it's proof that current approaches don't work.

---

## Key Themes

#benchmark #database #text-to-sql #evaluation #enterprise #LLMs

BEAVER sits at the intersection of several threads in this wiki. It's a [[Database and Data|database]] benchmark built with the rigor advocated in [[Demystifying Evals for AI Agents]]. It's the anti-[[Benchmark Exploitation]]: a benchmark constructed to be genuinely hard, with subtask annotations that make gaming it impossible (you can't fake the join key F1 score).

The Structural Template Recomposition pipeline is its own contribution: extracting atomic templates from real queries, de-duplicating, then composing them iteratively to create queries deeper than any individual seed. This is a [[Compound Engineering]] approach to dataset construction -- mechanical verification at each step rather than trusting an LLM to generate the whole thing.

---

## Architecture of the Benchmark

The dataset has three source databases (DW, NW, SP) with radically different schemas:
- **DW** (education/facilities): 97 tables, Oracle SQL -- the kind of schema where "rooms" means `FCLT_ROOMS.FCLT_ROOM_KEY`
- **NW** (compute infrastructure): 366 tables, MySQL
- **SP** (housing management): 349 tables, MySQL

The five-subtask framework is the real innovation. Previous benchmarks treat text-to-SQL as a black box. BEAVER gives you per-subtask F1 scores. You can see *exactly* where your system fails. The error taxonomy (Section 6) shows that even with oracle hints, 46.8% of errors are analytical/structural -- grouping, window functions, decomposition. These are reasoning errors, not retrieval errors.

The 594 extracted templates are reused across query categories, enabling controlled experiments: does your system fail because of domain knowledge, query complexity, or the combination?

---

## Critical Analysis

**This is what a benchmark should look like.** It's expensive to build (6 grad students + 6 DBAs for months), it's grounded in real data, it isolates variables, and it produces results that are genuinely informative rather than just leaderboard fodder. The subtask annotation framework should be standard practice for any domain-specific benchmark.

**The 10.8% number is more useful than the 82% on BIRD.** A benchmark where models score 82% tells you nothing about what still breaks. A benchmark where the best system scores 10.8% tells you the problem is unsolved and gives you a measurement stick for progress. The gap between BIRD and BEAVER (82% vs 10.8%) is the clearest illustration I've seen of why public-database benchmarks are measuring the wrong thing.

**The oracle-hint result (30.1%) is the paper's most important finding** and I wish they'd emphasized it more. Giving a model the correct table set, join keys, column mappings, domain facts, and decomposition strategy -- and still getting it wrong 70% of the time -- means the hard part of text-to-SQL is not information retrieval. It's compositional reasoning: assembling the pieces correctly. That's a much harder problem and one that scale alone may not solve.

**The Structural Template Recomposition pipeline is clever but GPT-5.2-dependent.** The paper uses GPT-5.2 for initial query generation and GPT-5-mini for decomposition scoring. If those models become unavailable or change behavior, the dataset's characteristics could shift. The expert verification at each step mitigates this, but the pipeline isn't fully model-agnostic.

**Stonebraker's involvement is telling.** Mike Stonebraker has spent 50 years building database systems (Ingres, Postgres, Vertica, C-Store, VoltDB). His name on this paper signals that the database research community sees enterprise text-to-SQL as a hard systems problem, not a solved ML problem. The architecture that follows from this diagnosis is now published: [[RUBICON]] (VLDB), by the Binnig group and sharing an author with BEAVER (Fabian Wenz), takes BEAVER's ~10% / ~30% numbers as the reason full SQL must be subsetted to "baby SQL" and proposes a table-centric, query-processor-driven system that scores 100% on its own RUBICON-Bench while agentic baselines score zero.

**The move from peterbaile.github.io to beaverbench.github.io between May 13 (paper v3) and May 20 (repo archive) suggests the project is in active institutionalization.** The dataset is on Hugging Face, the code is on a dedicated org GitHub, and there's a leaderboard and submission process. This is becoming the standard benchmark for the subfield.

The one thing I'd want that's missing: a public instance of one of the databases with real (anonymized) data loaded, so you can actually run queries and get the experience of the schema complexity. The paper describes it vividly, but there's no substitute for trying to write `FCLT_ROOMS JOIN CATALOG_SUBJECT_OFFERED ON FCLT_ROOM_KEY = MEET_PLACE` yourself.

---

*Sources: [[summary/beaver]], arXiv:2409.02038v3*
*Last updated: 2026-05-22*
