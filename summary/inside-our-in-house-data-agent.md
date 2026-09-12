---
url: https://openai.com/index/inside-our-in-house-data-agent/
title: "Inside OpenAI's in-house data agent"
author: Bonnie Xu, Aravind Suresh, and Emma Tang
date_fetched: 2026-05-18
date_published: 2026-01-29
topics:
  - agent-coding-workflow
---

Data powers how systems learn, products evolve, and how companies make choices. But getting answers quickly, correctly, and with the right context is often harder than it should be. To make this easier as OpenAI scales, we built our own bespoke in-house AI data agent that explores and reasons over our own platform.

Our agent is a custom internal-only tool (not an external offering), built specifically around OpenAI's data, permissions, and workflows. We're showing how we built and use it to help surface examples of the real, impactful ways AI can support day-to-day work across our teams. The OpenAI tools we used to build and run it (Codex, our GPT-5 flagship model, the Evals API, and the Embeddings API) are the same tools we make available to developers everywhere.

Our data agent lets employees go from question to insight in minutes, not days. This lowers the bar to pulling data and nuanced analysis across all functions, not just by our data team. Today, teams across Engineering, Data Science, Go-To-Market, Finance, and Research at OpenAI lean on the agent to answer high-impact data questions. For example, it can help answer how to evaluate launches and understand business health, all through the intuitive format of natural language. The agent combines Codex-powered table-level knowledge with product and organizational context. Its continuously learning memory system means it also improves with every turn.

## Why we needed a custom tool

OpenAI's data platform serves more than 3.5k internal users working across Engineering, Product, and Research, spanning over 600 petabytes of data across 70k datasets. At that size, simply finding the right table can be one of the most time-consuming parts of doing analysis.

As one internal user put it:

> "We have a lot of tables that are fairly similar, and I spend tons of time trying to figure out how they're different and which to use. Some include logged-out users, some don't. Some have overlapping fields; it's hard to tell what is what."

Even with the correct tables selected, producing correct results can be challenging. Analysts must reason about table data and table relationships to ensure transformations and filters are applied correctly. Common failure modes — many-to-many joins, filter pushdown errors, and unhandled nulls — can silently invalidate results. At OpenAI's scale, analysts should not have to sink time into debugging SQL semantics or query performance: their focus should be on defining metrics, validating assumptions, and making data-driven decisions.

## How it works

The agent is powered by GPT-5.2 and is designed to reason over OpenAI's data platform. It's available wherever employees already work: as a Slack agent, through a web interface, inside IDEs, in the Codex CLI via MCP, and directly in OpenAI's internal ChatGPT app through a MCP connector.

Users can ask complex, open-ended questions which would typically require multiple rounds of manual exploration. Example prompt (using a test data set): "For NYC taxi trips, which pickup-to-dropoff ZIP pairs are the most unreliable, with the largest gap between typical and worst-case travel times, and when does that variability occur?"

The agent handles the analysis end-to-end, from understanding the question to exploring the data, running queries, and synthesizing findings.

One of the agent's superpowers is how it reasons through problems. Rather than following a fixed script, the agent evaluates its own progress. If an intermediate result looks wrong (e.g., if it has zero rows due to an incorrect join or filter), the agent investigates what went wrong, adjusts its approach, and tries again. Throughout this process, it retains full context, and carries learnings forward between steps. This closed-loop, self-learning process shifts iteration from the user into the agent itself, enabling faster results and consistently higher-quality analyses than manual workflows.

The agent covers the full analytics workflow: discovering data, running SQL, and publishing notebooks and reports. It understands internal company knowledge, can web search for external information, and improves over time through learned usage and memory.

## Context is everything

High-quality answers depend on rich, accurate context. Without context, even strong models can produce wrong results, such as vastly misestimating user counts or misinterpreting internal terminology.

To avoid these failure modes, the agent is built around multiple layers of context that ground it in OpenAI's data and institutional knowledge.

**Layer #1: Table Usage** — Metadata grounding (schema, column names, data types, table lineage) plus query inference from historical queries to understand which tables are typically joined together.

**Layer #2: Human Annotations** — Curated descriptions of tables and columns provided by domain experts, capturing intent, semantics, business meaning, and known caveats not easily inferred from schemas or past queries. "Metadata alone isn't enough. To really tell tables apart, you need to understand how they were created and where they originate."

**Layer #3: Codex Enrichment** — By deriving a code-level definition of a table, the agent builds a deeper understanding of what the data actually contains. Nuances on what is stored in the table and how it is derived from an analytics event provide extra information. For example, it can give context on the uniqueness of values, how often the table data is updated, the scope of the data (e.g., if the table excludes certain fields, it has this level of granularity), etc. This provides enhanced usage context by showing how the table is used beyond SQL in Spark, Python, and other data systems. The agent can distinguish between tables that look similar but differ in critical ways — for example, whether a table only includes first-party ChatGPT traffic. This context is refreshed automatically, so it stays up to date without manual maintenance.

**Layer #4: Institutional Knowledge** — The agent can access Slack, Google Docs, and Notion, capturing critical company context such as launches, reliability incidents, internal codenames and tools, and the canonical definitions and computation logic for key metrics. These documents are ingested, embedded, and stored with metadata and permissions. A retrieval service handles access control and caching at runtime.

**Layer #5: Memory** — When the agent is given corrections or discovers nuances about certain data questions, it saves these learnings for next time, allowing it to constantly improve. Future answers begin from a more accurate baseline rather than repeatedly encountering the same issues. The goal of memory is to retain and reuse non-obvious corrections, filters, and constraints that are critical for data correctness but difficult to infer from the other layers alone. Example: the agent didn't know how to filter for a particular analytics experiment (it relied on matching against a specific string defined in an experiment gate). Memory was crucially important to ensure it was able to filter correctly, instead of fuzzily trying to string match. Memories are scoped at global and personal levels, and users can manually create and edit them.

**Layer #6: Runtime Context** — When no prior context exists for a table or when existing information is stale, the agent can issue live queries to the data warehouse to inspect and query the table directly. It can also talk to other Data Platform systems (metadata service, Airflow, Spark) as needed.

A daily offline pipeline aggregates table usage, human annotations, and Codex-derived enrichment into a single, normalized representation. This enriched context is converted into embeddings using the OpenAI embeddings API and stored for retrieval. At query time, the agent pulls only the most relevant embedded context via RAG instead of scanning raw metadata or logs.

## Built to think and work like a teammate

One-shot answers work when the problem is clear, but most questions aren't. The agent is built to behave like a teammate you can reason with — conversational, always-on, handling both quick answers and iterative exploration.

It carries over complete context across turns, so users can ask follow-up questions, adjust their intent, or change direction without restating everything. If the agent starts heading down the wrong path, users can interrupt mid-analysis and redirect it.

When instructions are unclear or incomplete, the agent proactively asks clarifying questions. If no response is provided, it applies sensible defaults to make progress. For example, if a user asks about business growth with no date range specified, it may assume the last seven or 30 days.

After rollout, the team observed that users frequently ran the same analyses for routine repetitive work. To expedite this, the agent's workflows package recurring analyses into reusable instruction sets. Examples include workflows for weekly business reports and table validations. By encoding context and best practices once, workflows streamline repeat analyses and ensure consistent results across users.

## Moving fast without breaking trust

Building an always-on, evolving agent means quality can drift just as easily as it can improve. Without a tight feedback loop, regressions are inevitable and invisible. The team leverages OpenAI's Evals API to measure and protect response quality.

Evals are built on curated sets of question-answer pairs. Each question targets an important metric or analytical pattern, paired with a manually authored "golden" SQL query that produces the expected result. For each eval, they send the natural language question to the query-generation endpoint, execute the generated SQL, and compare the output against the expected result.

Evaluation doesn't rely on naive string matching. Generated SQL can differ syntactically while still being correct, and result sets may include extra columns that don't materially affect the answer. To account for this, they compare both the SQL and the resulting data, and feed these signals into OpenAI's Evals grader. The grader produces a final score along with an explanation, capturing both correctness and acceptable variation.

These evals run continuously during development to identify regressions and act as canaries in production.

## Agent security

The agent plugs directly into OpenAI's existing security and access-control model. It operates purely as an interface layer, inheriting and enforcing the same permissions and guardrails that govern OpenAI's data. All access is strictly pass-through — users can only query tables they already have permission to access. When access is missing, it flags this or falls back to alternative datasets the user is authorized to use.

The agent is built for transparency. It exposes its reasoning process by summarizing assumptions and execution steps alongside each answer. When queries are executed, it links directly to the underlying results, allowing users to inspect raw data and verify every step of the analysis.

## Lessons learned

**Lesson #1: Less is More** — Early on, they exposed the full tool set to the agent and quickly ran into problems with overlapping functionality. While redundancy can be helpful for specific custom cases and is more obvious to a human when manually invoking, it's confusing to agents. To reduce ambiguity and improve reliability, they restricted and consolidated certain tool calls.

**Lesson #2: Guide the Goal, Not the Path** — Highly prescriptive prompting degraded results. While many questions share a general analytical shape, the details vary enough that rigid instructions often pushed the agent down incorrect paths. By shifting to higher-level guidance and relying on GPT-5's reasoning to choose the appropriate execution path, the agent became more robust and produced better results.

**Lesson #3: Meaning Lives in Code** — Schemas and query history describe a table's shape and usage, but its true meaning lives in the code that produces it. Pipeline logic captures assumptions, freshness guarantees, and business intent that never surface in SQL or metadata. By crawling the codebase with Codex, the agent understands how datasets are actually constructed and is able to better reason about what each table actually contains. It can answer "what's in here" and "when can I use it" far more accurately than from warehouse signals alone.

## Same vision, new tools

The team is constantly working to improve the agent by increasing its ability to handle ambiguous questions, improving reliability and accuracy with stronger validations, and integrating it more deeply into workflows. The goal: the agent should blend naturally into how people already work, instead of functioning like a separate tool.

While the tooling will keep benefiting from underlying improvements in agent reasoning, validation, and self-correction, the team's mission remains the same: seamlessly deliver fast, trustworthy data analysis across OpenAI's data ecosystem.
