# Ontology-Driven Observability at Netflix

Netflix's Observability Ontology: a W3C-standard knowledge graph (RDF/OWL/SHACL/SPARQL on their own RDF store, QuipuDB) that unifies MELT telemetry, incidents, deployments and incident-chat reasoning into typed triples, so that both humans and the AutoRCA agent fleet query one shared vocabulary instead of reconciling five tool-specific views of "an application."

---

At Netflix scale — 2B requests a day, 38M real-time log events per second — a home-screen regression can originate anywhere in the system. The authors open with a four-hour investigation timeline and the honest observation that most of those hours went not into computing anything hard but into *reconstructing relationships the observability systems already knew but had never written down*. Each tool is internally coherent and externally isolated: Atlas knows time series, Radar knows alerts, PagerDuty knows rotations, Spinnaker knows deployments. They do not agree on what they are talking about — an "application" in one system is a "service" in another and a "cluster" in a third. The hard part of end-to-end observability, they argue, is a **data modelling problem**, not a data-volume problem.

The ontology is the fix: triples like `api-gateway ops:ownedBy "Team Bedrock"` turn three-tool lookups into one SPARQL query. Two build decisions carry the argument. First, they refused a generic Service type — every application resolves to exactly one `ops:Application`, validated against a ~49,000-entry catalog before creation, because "unvalidated strings never become applications." Second, they model *human reasoning*, not just machine telemetry: incident Slack channels — the hypotheses tried, the mitigation that worked — are parsed into ordered triples before that knowledge evaporates when the channel goes quiet.

---

## Key quotes

> "Each of our systems is internally coherent and externally isolated. ... The difficulty is that they do not agree on what they are talking about. ... The hard part of end-to-end observability is probably better understood as a data modelling problem."

The framing that makes the whole piece worth reading. Most observability investment goes into more data and better dashboards; Netflix claims the bottleneck was the join, and the join is a schema problem.

> "We need the right needle, not any needle."

On why agents alone don't fix this: without the right context delivered to them, agents hallucinate, and on-call teams cannot afford wrong agentic analysis. Context engineering before agent orchestration.

> "At the start of this pipeline, an LLM could say anything, but it cannot emit a triple that violates the type system. We constrain the output space tightly enough and a stochastic process yields a deterministic contract."

The paper's most transferable idea — "creating determinism from non-determinism." A five-layer entropy ladder, where entropy is "the proportion of information still trapped in unstructured form," converts stochastic LLM output into typed triples. Any single step is non-deterministic; the chain in aggregate is deterministic.

> "The engine's role here is to propose with evidence, not to make conclusions. ... the graph records the confidence alongside the claim."

The correlation engine reads thirteen signal types, scores causal/correlational chains (deployment-before-log-error carries a high prior; sub-0.6 confidence is discarded), and stores evidence alongside every hypothesis. Proposals carry their provenance so humans can weigh them.

> "The ontology acts as the interface. ... A new specialist learns the same vocabulary once and can immediately read everything the other six have written."

The AutoRCA division of labour: specialist agents coordinate only through the graph. This is a coupling argument — shared typed substrate over direct agent-to-agent protocols.

---

## Themes

#concept #pattern #tool

- **Ontology as the interface.** Multi-agent coordination via a typed shared substrate rather than direct messaging.
- **Determinism from stochastic parts.** Narrow output contracts + type systems make LLM pipelines queryable.
- **Model human reasoning, not just telemetry.** The most valuable edges came from incident discussion, not metrics.
- **Knowledge flywheel.** Observe → Enrich → Infer → Adapt; encoded incident knowledge compounds into detection rules.

## Analysis

The genuinely opinionated part of this piece is its insistence that a schema, of all things, is the high-leverage investment in 2026 — right when everyone else is reaching for more agents. The argument is coherent: agents are context-starved, and the missing context here isn't retrievable from any single system because it doesn't exist in queryable form anywhere. The ontology is expensive to build (a 49,000-entry catalog, twelve namespaces, a custom graph database) and Netflix is one of few companies that could both build and *populate* it.

The determinism-from-stochasticity ladder is the idea I'd defend hardest with them — it's structurally the same move as schema-constrained structured outputs, but applied to a whole pipeline with provenance attached. Two cautions. First, the "deterministic" graph is only as good as the enrichment: a wrongly tripled fact is now confidently queryable, and the piece is thinner on detecting bad triples than on emitting good ones (the 0.6-confidence floor and human-review-of-ontology-changes are the only guards mentioned). Second, the four-hour-to-less claim is framed carefully — the graph absorbs context assembly, the engineer still fixes — but the flywheel's "patterns become wisdom" language ("api-gateway deploys cause 40% of SEV-2s") is exactly the kind of aggregate claim that goes stale without its evidence attached; they do keep evidence, which is the right call.

The honest scope-limiting — "the alert still fires at 3:30pm" — is what makes this credible rather than another self-healing-infra pitch.

## Relations

This source strengthens [[MELT]] with a production-scale argument for its central claim — the four pillars are the right raw materials but a quarter-answer each, and the missing fifth layer is the join between them. It extends [[Ontology and Information Systems (SEP)]] from philosophy into production: RDF/OWL/SHACL/SPARQL chosen deliberately over bespoke models, with the W3C portability argument made from operational scars. It nuances [[ZizkaDB — Causal Lineage for Agent Observability]]: Netflix independently arrives at "store causal chains with their evidence attached," suggesting confidence-plus-provenance is converging on as the standard shape for agent-facing causal reasoning. And it complicates [[Your Knowledge Graph Is Making Your Agent Dumber]] — which warns graphs feed agents stale or wrong structure — by showing the mitigations (catalog gating, evidence-bearing hypotheses, human approval of ontology changes) a graph builder must ship to avoid exactly that failure.

---
*Sources: [[raw/ontology-driven-observability-building-an-end-to-end-knowledge-graph-at-netflix-scale-421392cf5adf]], [[summary/ontology-driven-observability-building-an-end-to-end-knowledge-graph-at-netflix-scale-421392cf5adf]]*
*Last updated: 2026-10-20*
