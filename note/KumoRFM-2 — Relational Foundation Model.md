# KumoRFM-2: Scaling Foundation Models for Relational Learning

Kumo AI's paper on KumoRFM-2, a pre-trained foundation model that does in-context learning directly over multi-table relational databases. Its claim to fame is being the first few-shot foundation model to beat task-specific supervised models on standard relational benchmarks (41 tasks across RelBenchV1/V2, SALT, 4DBInfer), while scaling to 500B+ rows and shipping as an agent-ready platform with a query language, skills, and an MCP server.

---

## What it argues

The tabular foundation model paradigm (TabPFN-style in-context learning on one table) hits a wall on real-world data, which lives in connected tables. Naive fixes — flattening via fixed-function feature synthesis (DFS) or propositionalization — are task-agnostic and, the authors argue, fundamentally not expressive enough: their constructed counterexample (a target defined by co-occurrence of two binary columns in a child table) leaves any column-wise encoder at AUROC 0.5 while KumoRFM-2 separates it perfectly. The alternative is to model the database natively as a temporal heterogeneous graph and pre-train across four axes: rows, columns, foreign keys, and cross-sample context.

The key architectural change from v1 is early task injection plus a two-stage hierarchical attention scheme (intra-table, then cross-table/cross-sample). This is credited for both performance and the noise robustness results — task-conditioned processing filters irrelevant columns before they can pollute aggregation.

## Key quotes

> "To our knowledge, this is the first time a few-shot foundation model has been shown to surpass supervised approaches on common benchmark tasks, with performance further improving upon fine-tuning."

The paper's whole thesis in one sentence. The supervised comparison is not strawman-grade: RelGNN is a current task-specific relational GNN, and KumoRFM-2 beats it by 1.54 points AUROC on RelBenchV1 while using as little as 0.2% of the training data as context. Supervised models do keep an edge on the largest-dataset tasks (amazon, stack, hm), which the authors honestly acknowledge and attribute to context-size limits.

> "PQL turns predictive modeling into a composable primitive for agents, allowing queries to be stacked and combined with SQL operations to express complex decision-making pipelines."

The most wiki-relevant sentence. The model is the science, but the product is an agent interface: a declarative Predictive Query Language as structured intermediate representation so agents never construct model inputs directly, plus shipped skills and an MCP server. This is prediction-as-a-tool, the same move tool protocols make everywhere else.

> "While supervised relational models eliminated the need for manual feature engineering, KumoRFM-2 now eliminates the need for additional training, which can range from hours to days."

Deployment-time framing: the value proposition is seconds-to-first-prediction on unseen tasks, cold start included — the model holds strong performance with 75% of foreign keys removed.

## Key themes

#concept — Relational foundation models: ICL extended from a single table to a temporal heterogeneous graph
#tool — KumoRFM-2, its Python SDK, PQL, MCP server, and agent skills
#pattern — Task-conditioned early context injection as the robustness lever
#comparison — Few-shot ICL vs supervised training; native relational modeling vs DFS flattening

## Analysis

The honest parts of this paper are what make it credible. The authors name where supervised models still win (huge training sets, volatile regression targets where DS-engineered features occasionally beat them), and the robustness ablations (context size, hops, neighborhood breadth, noise) are more thorough than the usual benchmark table. The expressivity counterexample against DFS pipelines is a genuinely good argument, not a marketing one.

The caveat worth holding onto: benchmarks are RelBench-family relational tasks, all pre-registered with fixed splits, and the model's vendor ships the benchmarks it wins on. The 8% vs AutoGluon claim is on the SAP SALT enterprise benchmark, which is impressive but narrow. Also, "foundation model" here does a lot of work — pre-training on synthetic structural-causal-model data plus real databases, and no leakage is claimed, but the vendor-evaluates-vendor pattern always deserves a discount.

For this wiki's concerns, the interesting thing is not the AUROC table but the system design: a frozen model, stateless multi-tenant serving, context generated ad hoc by replaying database history, and prediction exposed as a composable query primitive for agents. That is a data-science workflow where the agent composes queries instead of writing pipelines — the "specify prediction tasks over a single relational database" abstraction replacing "build and maintain training pipelines." It also sharpens the question of when LLMs should touch structured prediction at all: the paper's own LLM baselines underperform badly, competitive only where memorization looks like leakage.

## Related pages

- [[Why LLMs Fail at Tabular Prediction]] — this paper supplies the quantitative version of that page's diagnosis: LLM baselines trail badly on relational prediction, and the authors locate the cause in missing inductive bias rather than prompting; KumoRFM-2 is the purpose-built alternative.
- [[QueryBrew — System-Agnostic SQL-to-SQL Query Optimization]] — a different bet on database-native intelligence: where QueryBrew optimizes the SQL itself, KumoRFM-2 layers prediction on top of the schema, both treating the database rather than the app as the substrate.
- [[Postgres Transactions Are a Distributed Systems Superpower]] — KumoRFM-2's pushdown mode executes recursive SQL along metapaths for context retrieval, a production instance of the "let the database do the work" principle.
- [[Object-Relational Impedance Mismatch]] — the paper sidesteps a modern form of this mismatch: instead of flattening relational data into rows for models, it keeps the relational structure native end to end.

---
*Sources: [[raw/2604-12596v1]], [[summary/2604-12596v1]]*
*Last updated: 2026-10-24*
