# Inside OpenAI's In-House Data Agent

OpenAI built a bespoke internal data agent powered by GPT-5.2 that lets 3,500+ employees query 600PB across 70k datasets using natural language. Written by Bonnie Xu, Aravind Suresh, and Emma Tang (January 2026), this is the definitive engineering post on how they did it. The agent is not a product — it's an internal tool built on the same APIs (Codex, GPT-5, Evals, Embeddings) that OpenAI sells to developers. The most interesting claim: the hard problem isn't model intelligence, it's making the company's data reality legible enough for an agent to navigate.

---

## Key Quotes

> "We have a lot of tables that are fairly similar, and I spend tons of time trying to figure out how they're different and which to use. Some include logged-out users, some don't. Some have overlapping fields; it's hard to tell what is what."

The user quote that motivates the entire project. This is the data platform version of "I can't find the right file." At 70k datasets, table discovery is the bottleneck, not query writing.

> "Metadata alone isn't enough. To really tell tables apart, you need to understand how they were created and where they originate."

This is the justification for Layer #3 (Codex Enrichment) and it's the most transferable insight in the post. Schemas tell you column names. Code tells you what the data actually means. The agent crawls the codebase with Codex to understand pipeline logic — freshness guarantees, filtering assumptions, business intent — that never surfaces in SQL or metadata. This is also their Lesson #3: "Meaning Lives in Code."

> "Rather than following a fixed script, the agent evaluates its own progress. If an intermediate result looks wrong (e.g., if it has zero rows due to an incorrect join or filter), the agent investigates what went wrong, adjusts its approach, and tries again."

The self-correcting loop. This is the closed-loop reasoning that shifts iteration from the user into the agent. Combined with memory (Layer #5), it means the agent doesn't just fix mistakes — it remembers the fix for next time.

> "While many questions share a general analytical shape, the details vary enough that rigid instructions often pushed the agent down incorrect paths."

From Lesson #2: "Guide the Goal, Not the Path." Highly prescriptive prompting degraded results. Switching to higher-level guidance and trusting GPT-5's reasoning to choose execution paths made the agent more robust. This is the same insight as the [[Designing Agentic Loops]] meta-skill: choose guardrails and success criteria, don't micromanage the loop.

## The Six-Layer Context Architecture

This is the architectural heart of the post and the most directly useful part for anyone building something similar. Each layer addresses a different failure mode:

1. **Table Usage** — Schema metadata, column names/types, table lineage, historical query patterns. The basics.
2. **Human Annotations** — Curated table/column descriptions from domain experts. Captures intent, semantics, business meaning, and known caveats.
3. **Codex Enrichment** — Crawls the codebase to derive code-level definitions of tables. Understands what data actually contains, not just its schema. Distinguishes between lookalike tables (e.g., "does this include logged-out users?"). Auto-refreshed.
4. **Institutional Knowledge** — Slack, Google Docs, Notion. Captures launches, incidents, internal codenames, canonical metric definitions. Embedded with metadata and permissions.
5. **Memory** — Stores non-obvious corrections, filters, and constraints from user interactions. Scoped at global and personal levels. User-editable. Prevents the agent from repeatedly hitting the same gotchas.
6. **Runtime Context** — Live queries to the data warehouse when existing context is missing or stale. Can also talk to metadata services, Airflow, Spark.

A daily offline pipeline aggregates layers 1-3 into embeddings via the Embeddings API. At query time, RAG pulls only the most relevant context. Runtime queries fire live as needed.

The layering is smart because each layer addresses a different information gap: schemas (what), annotations (why), code (how), docs (context), memory (gotchas), runtime (validation). No single layer is sufficient. Together they approximate what an experienced data engineer carries in their head.

## Built Like a Teammate, Not a Tool

The agent is designed for conversation, not one-shot Q&A. Specific design choices:

- **Full context carryover across turns** — follow-ups, direction changes, refinements without restating
- **Interruptible mid-analysis** — users can redirect it like a human collaborator
- **Proactive clarifying questions** — asks when instructions are unclear
- **Sensible defaults** — if no date range specified, assumes last 7 or 30 days
- **Available everywhere** — Slack, web, IDE, Codex CLI via MCP, internal ChatGPT via MCP
- **Workflows** — recurring analyses packaged as reusable instruction sets (weekly reports, table validations)

The teammate framing is not just vibes. It's a specific product philosophy: the agent should be non-blocking (defaults keep it moving) but not presumptuous (asks when it matters). Compare to [[Experience Design for Agents]] — the UX matters more than model capability for adoption.

## Continuous Evaluation as Trust Infrastructure

The Evals API runs continuously during development and as production canaries. Each eval pairs a natural language question with a manually authored "golden" SQL query. The grader compares both generated SQL and resulting data, not just string matching — it accounts for syntactic variation and extra columns that don't affect the answer.

This is the production implementation of the feedback loop in [[Guardrails and Feedback Loops]]: deterministic enforcement through automated comparison, not prompt-level pleading.

## Critical Analysis

The six-layer context architecture is the most valuable part of this post, and the most honest. Each layer addresses a real failure mode they encountered. The fact that they needed all six — that schemas alone produce wrong answers, that annotations drift, that only code reveals true table semantics — is a sobering data point for anyone building an enterprise data agent. You cannot skip the platform work.

The "Less is More" lesson (Lesson #1) is underappreciated. They found that exposing the full tool set to the agent caused confusion from overlapping functionality. Consolidating tools improved reliability. This directly contradicts the "more tools = more capable" assumption behind most MCP server design. Compare to [[Elysia]]'s decision-tree approach of constraining tools per node.

"Meaning Lives in Code" (Lesson #3) is the most transferable idea. The agent crawls the codebase with Codex to understand what tables actually contain — pipeline logic, freshness guarantees, business intent. This is a concrete implementation of something the [[Agent Memory and Context]] synthesis discusses in theory: the codebase as ground truth for data semantics.

What's conspicuously absent:

- **No mention of hallucinated SQL.** For a post about data agents, the complete silence on whether the agent ever produces wrong-but-plausible-looking results is notable. The Evals section addresses quality drift but not the fundamental failure mode of confident wrong answers.
- **No cost discussion.** 600PB, 70k datasets, daily embedding pipelines, GPT-5.2 inference — the compute bill must be substantial. Not mentioned.
- **No latency numbers.** "Minutes, not days" is the only timing claim.
- **The recursive feedback loop is unexamined.** OpenAI uses Codex agents to run the data infrastructure that trains the models that power Codex agents. If the agents degrade data quality, the models degrade, the agents get worse. The Evals API is a circuit breaker but the authors don't frame it that way.

The post is a company blog, not independent analysis, and it shows in the omission pattern. But the architectural detail — six context layers, teammate design philosophy, the three lessons — is genuine engineering content, not marketing. The most useful read is as a reference architecture for anyone building an agent that needs to reason over a large, messy data estate. The message is: the platform work IS the product work, and it's a lot more work than the model.

---

*Sources: [[raw/inside-our-in-house-data-agent]]*
*Last updated: 2026-05-18*
