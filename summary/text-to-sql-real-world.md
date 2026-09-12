---
url: https://cacm.acm.org/blogcacm/if-you-think-you-can-do-real-world-text-to-sql/
title: "If You Think You Can Do Real-World Text-to-SQL"
author: Michael Stonebraker, Peter Baile Chen
date_fetched: 2026-07-25
date_published: 2026-07-20
topics:
  - databases-and-data
---

Stonebraker and Chen argue that existing text-to-SQL benchmarks (Spider,
Bird-SQL, Spider 2.0) paint an overly rosy picture because they miss four
real-world data warehouse challenges.

First, **training data leakage**: real warehouse data sits behind enterprise
access controls, so LLMs haven't seen it. Public benchmark data is in the pile.
Second, **schema rot**: schemas mutate over years of business changes, leaving
non-intuitive table and column names — six different "salary" columns with
overlapping but undocumented semantics. Third, **bespoke data**: institutional
jargon (MIT's "J-term," building numbers instead of names) that an LLM can't
infer. Fourth, **query complexity**: real ad-hoc queries typically have 2–3
joins, not the single-join student-generated queries in Spider/Bird.

They created the **Beaver benchmark** from four real data warehouses (starting
with MIT's 1,400+ table Oracle warehouse), using real query logs and
user-validated natural-language equivalents. On Beaver, a pure LLM scored **0%**
accuracy. Adding RAG, prompt engineering, and agentic AI raised it to the low
teens. Giving the LLM the correct tables and join clauses pushed it to the 30s.
That's 50+ points below public benchmark results — the gap between "promising"
and "doesn't work."

The authors point to their Rubicon system as one direction forward and invite
others to use Beaver for research on text-to-SQL that actually handles
real-world conditions.

---
*Source: [[raw/text-to-sql-real-world]]*
*Last updated: 2026-08-01*
