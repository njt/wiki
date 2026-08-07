# Text-to-SQL in the Real World

Michael Stonebraker and Peter Baile Chen argue that the public text-to-SQL benchmarks everyone is celebrating (80–90%+ accuracy on Spider and Bird-SQL) are "of academic interest only" — they don't capture any of the four structural challenges of real enterprise data warehouses: training data contamination, schema rot, idiosyncratic institutional knowledge, and query complexity. Their Beaver benchmark, built from real MIT and enterprise data warehouses, humbles the best LLMs to 10% accuracy. The gap between 90% and 10% is the difference between "the technology has promise" and "the technology does not work."

---

## Key Quotes

> "The data is in 'the pile,' so LLMs can train on it."

The quiet scandal of text-to-SQL benchmarks. Spider and Bird-SQL use public databases whose schemas and content are in the training data. The models aren't demonstrating reasoning about an unfamiliar schema — they're regurgitating. As the authors note, "almost all data warehouses we know about are behind serious enterprise access controls and are not publicly available."

> "Real data warehouses exhibit 'schema rot.'"

The most vivid concept in the piece. Schemas accumulate changes over years as companies merge, regulations shift, and business logic evolves. Nobody rebuilds the schema cleanly each time. The result: "non-intuitive table and column names," six different columns all labeled "salary" with undocumented, overlapping semantics, and materialized views that create "multiple ways to solve a query" — each a headache for an LLM.

> "A pure LLM generated an accuracy score of zero. Adding RAG, prompt engineering, and agentic AI raised accuracy to the 10+% range."

Not 10% with a naive prompt. 10% *with everything thrown at it*. Even giving the LLM the correct tables and join clauses only pushed it to 30%. This is the most damning data point — they handed the model the right ingredients and it still mostly failed to assemble them.

> "This is 50+% lower in accuracy than the public benchmarks and is the difference between 'the technology has promise' and 'the technology does not work.'"

Stonebraker's closing frame. This isn't "impressive progress with room to improve." It's a category error: the benchmarks measure the wrong thing.

---

## Key Themes

#benchmark #database #text-to-sql #enterprise #LLMs #person

---

## The Four Problems Public Benchmarks Miss

### 1. Training Data Contamination

Public databases are literally in the training corpus. The LLM has seen the schema, the data, and likely the queries before. Enterprise warehouses are behind access controls — the LLM walks in cold. This alone invalidates Spider/Bird as predictors of real-world performance.

### 2. Schema Rot

Stonebraker's term for the organic degradation of database schemas across years of business change: mergers, acquisitions, regulatory shifts, ad-hoc patches. The schema that started clean accumulates warts. Column names become cryptic, semantics drift, the same concept appears six times across tables with undocumented differences. The LLM can't look up "what does this column *really* mean" — that knowledge lives only in the heads of the DBAs who've been there ten years.

### 3. Idiosyncratic Data

Every organization has its internal vocabulary. MIT's data warehouse knows about "J-term" (January's one-month term) and building *numbers* (the Stata Center is "Building 32," never "Stata"). These are obvious to the humans who work there and invisible to an LLM. RAG can help with the dictionary entries, but only if someone writes them down first — which, per schema rot, they haven't.

### 4. Query Complexity

Real enterprise queries have 2–3 joins minimum, plus CTEs, window functions, and nested subqueries. Spider 2.0 addressed this, but not the first three problems. And the combination — complex SQL *plus* domain knowledge *plus* rotten schema — is where enterprise queries actually live. On Beaver, the "complex + domain knowledge" category averages 5.4% accuracy.

---

## The Beaver Benchmark

Built over two years from real query logs at MIT (1,400+ tables, Oracle) and three other enterprise warehouses. The methodology: extract real SQL from logs, generate natural-language equivalents with the help of actual users, anonymize and extend. The result: 9,128 question-SQL pairs across 812 tables in 19 domains.

Key results:
- Pure LLM: **0%**
- + RAG + prompt engineering + agentic AI (GPT-5.2): **~10%**
- + oracle hints (correct tables + join clauses): **~30%**

A leaderboard is maintained at beaverbench.github.io.

---

## Critical Analysis

**This article is the public-facing companion to the Beaver paper, and it's more effective than the paper at making the case.** The paper is a rigorous benchmark; this blog post is a polemic. Stonebraker's authority — 50 years in databases, Ingres, Postgres, Turing Award — gives the "academic interest only" dismissal real weight. He's not some VC-backed AI CEO trying to sell a product; he's the guy who literally designed the database systems that underpin the industry. When he says text-to-SQL doesn't work in the real world, it lands differently.

**The "schema rot" concept deserves to be more widely adopted.** It names something every data engineer knows but nobody has articulated this crisply. It's the database analogue of technical debt, but worse because it's invisible to anyone outside the organization. An LLM can read a codebase and see the accumulated cruft; it cannot see that `SALARY_6` means "net after taxes and commissions for part-time employees hired after 2019."

**The 0% → 10% → 30% progression is a better research roadmap than any leaderboard.** The jump from 0% (raw LLM) to 10% (RAG + agents) to 30% (oracle hints) isolates where the difficulty lives. The fact that oracle hints only get you to 30% says the bottleneck isn't information retrieval — it's compositional reasoning. The LLM can't reliably assemble the pieces even when you hand it all the pieces. That's a much deeper problem than "better RAG."

**The Rubicon pointer is tantalizing but thin.** The article name-drops Rubicon (arxiv.org/abs/2604.21413) as "our own ideas on how to do better" but doesn't explain the approach. Given Stonebraker's lineage — he built Postgres, Vertica, VoltDB, and C-Store — Rubicon is likely a systems-level answer, not a prompt-engineering answer. If LLMs can't reassemble a query given the ingredients, maybe the answer is to *not let the LLM assemble the query* — instead, use the LLM to map natural language to a structured intermediate representation, then compile deterministically to SQL. That's the [[Malloy]]/[[Malloyyo]] architecture applied to the text-to-SQL problem.

**This pairs destructively with [[Constraint Decay]].** That paper found databases are the primary failure driver for LLM coding agents — a 19.3 percentage point accuracy drop when PostgreSQL enters the picture. Stonebraker's article explains *why*: real databases are far harder than the clean schemas in training data, and every additional constraint (rotten schema, domain vocabulary, complex analytical structure) compounds the difficulty. The two pieces together make a compelling case that enterprise data is the hardest unsolved problem in applied LLMs.

**The missing piece: a public instance.** As with the Beaver paper, I want a real (anonymized) database I can connect to and try querying. Reading about `FCLT_ROOMS.FCLT_ROOM_KEY` is one thing; trying to join it correctly is another. A public playground would make the benchmark's case more visceral than any paper can.

**A complementary finding from [[AI-Assisted Database Work — The Machine Reads, The Human Decides]]:** Pinal Dave's nine-engagement field report draws a boundary that Stonebraker's piece implies but doesn't state: AI fails at *generating* queries for real schemas but succeeds at *reading and classifying* existing database artifacts at volume. The machine doesn't need to understand the schema to extract business rules from 400K lines of PL/SQL, reverse-engineer an undocumented vendor database, or trace column lineage through dynamic SQL. Reading is a different capability from generating, and the two pieces together bound the useful surface area of AI for database work.

**For the wiki's synthesis: this is a data point in the emerging pattern that LLM benchmarks are systematically misleading.** [[BEAVER]] vs. BIRD is the text-to-SQL version of the same story told by [[FrontierCode]] (13.4% Diamond on real-world PR mergeability) and [[Five Studies That Are Changing How I Think About AI in Software Engineering]] (AI compresses upstream coding, everything downstream breaks). The benchmarks that matter are the ones that measure what happens in production, not in the lab. Beaver is the gold standard for what that looks like in the database domain.

---

*Sources: [[raw/text-to-sql-real-world]], beaverbench.github.io*
*Last updated: 2026-07-25*
