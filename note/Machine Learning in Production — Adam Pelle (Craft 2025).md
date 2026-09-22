# Machine Learning in Production — Adam Pelle (Craft 2025)

Adam Pelle, staff engineer at insurtech Marshmallow (Budapest), on the operational side of machine learning: 14 SageMaker online-inference endpoints hosting ~25 models across claims, fraud, and pricing (busiest ~30 req/s at P99). The central claim: once models consume the data pipeline, the flow becomes a cycle — "operational data equals to analytical data" — so best-effort emission from services is dead, data quality becomes the binding constraint, and ML endpoints must be run exactly like microservices plus ML-specific verification: feature stores, drift monitoring, contract testing, schema verification, and backtesting.

---

## What it argues

- **Real-time killed the offline loop.** Insurance use cases — risk scoring, fraud detection, dynamic pricing — need answers at request time. The old loop (services → Snowflake → data scientists → insights → product engineers) was "offline, reactive and it's very slow."
- **The data flow became a cycle.** Services emit analytical data → pipeline (Kafka) → raw warehouse → transformations → training → deployed models → live inference consumed by those same services. Once the cycle closes, there is no analytical tier you can treat casually: a missing field now degrades a live prediction instead of a dashboard.
- **Data quality is the binding constraint.** Model success is "explicitly bound to the data"; small inconsistencies compound downstream — garbage in, garbage out.
- **An organizational tension needs governance.** Services optimize with a need-to-know approach to data; models want to be fed everything. The middle ground "requires clear governance."
- **ML endpoints are microservices.** SageMaker HTTP endpoints called like any other service: latency SLAs, high availability, the four golden signals, Python project templates on a shared Docker base image, CI/CD, canary releases via SageMaker production variants.
- **Plus ML-specific machinery.** A Tecton feature store (versioning with visible history, cross-team consistency, automated backfill); a feature taxonomy (on-demand computed from request inputs, aggregational per-customer lookups, periodically updated batch features); drift monitoring — data drift (input distributions shift, e.g. pandemic-era vehicle mileage) vs concept drift (the input→output relationship breaks, e.g. repair-shop price inflation) — via PSI and Kolmogorov–Smirnov, remediated by proactive scheduled retraining rather than reactive.
- **The verification trio.** Bidirectional Pact contract testing (WireMock captures consumer pacts on the Java side; the Python provider publishes Pydantic-generated OpenAPI to a Pact broker that validates pacts against contracts); Schemathesis/Dredd schema verification, generating random data from the contract to catch providers that "do not verify its own schema"; and backtesting — replay historical production inputs through the PR branch's Docker image, re-scored with the *current* production model state, to confirm expected outcomes like "20% lower prices" before release.
- **An honest origin story.** Shipped first, governed later: models went to production "without any sort of considerations just to really get to the market very fast," the failures arrived, and only then was an MLOps team and roadmap assembled.

## Key quotes

> "We all know that sentence. Let me get back to you next week. Change to let me get back to you in 150 milliseconds."

The entire economic argument for online inference in one line. Every architectural decision in the talk — feature stores, drift monitoring, contract tests — is downstream of this latency inversion, and it's the rare talk that states its constraint up front instead of burying it.

> "Operational data equals to analytical data... If it's sent, it's sent. If not, then that's life. It's no longer true because it has consequences."

The reframe that does the real work. A decade of data engineering treated the warehouse as a reporting sink with best-effort emission; the moment a model consumes the pipeline, emission becomes load-bearing. Stating it as an equality rather than a warning is what makes the argument stick.

> "Data quality is like a marshmallow in a campfire. We ignore it and suddenly everything is on fire."

His signature joke, and an insurance company earning its own metaphor. It's also structurally accurate: quality failures in the input path are invisible until the model's output is consumed at scale — at which point everything is on fire.

> "Schema verification catches more bugs than we do next to a campfire" (the transcript garbles the punchline)

The gist's digest flags the garble, but the claim underneath survives: schema mismatches are the biggest root cause of endpoint issues. The decimals-as-strings saga — custom fields for min/max bounds, "very cumbersome" support built across the stack — is the war story, and "definitely worth it" is the verdict of someone who paid.

> "First we started to add these models into production without any sort of considerations just to really get to the market very fast. And then it introduced problems, failures." (Q&A)

The most valuable minute of the talk. Asked for the biggest challenge, he doesn't name a technology — he names timing, and admits the MLOps discipline was assembled *after* the failures. Ship-fast-then-govern is the actual sequence at most companies; it is almost never said out loud on stage.

> "In one year time we probably have more models than actual Marshmallows in the office snack bar."

Growth framing: 14 endpoints and ~25 models today, a goal of 100+ models across every major business domain. The joke is load-bearing — that scale is exactly why the template-propagation problem matters.

## Key themes

- #concept **Operational data equals analytical data** — the cycle that retires "it's just analytics." The single most transferable idea in the talk.
- #pattern **Models as microservices, plus an ML-specific verification layer** — the same production standards (SLAs, golden signals, CI/CD, canary), then contract testing, schema verification, and backtesting stacked on top.
- #pattern **Bidirectional contract testing** — consumer pacts generated from captured network calls (WireMock), provider OpenAPI generated from Pydantic, a Pact broker doing the validation. "Prevents integration failures before deployments."
- #tool **The Marshmallow ML stack** — SageMaker, Tecton feature store, Pact broker, Schemathesis/Dredd, PSI and Kolmogorov–Smirnov for drift.
- #concept **Drift with delayed ground truth** — data drift vs concept drift, monitored by distribution comparison and remediated by scheduled retraining. What the talk never says: in insurance, true labels arrive months late, so distribution drift is a proxy, not accuracy.
- #person **Adam Pelle** — staff engineer at Marshmallow, joined the Budapest office as its second engineer; drives ML integration and data architecture.

## Critical analysis

The central equality — operational data equals analytical data — is the right reframe, but the talk asserts the conclusion and skips the mechanism. Best-effort emission is declared dead; what replaced it is never described. No schemas on Kafka topics, no data-quality contracts at the streaming layer, no ownership rules. The entire "how" of the data-quality half is missing while the verification half gets real detail — an asymmetry that tells you where a staff engineer's tools live and where the org chart lives.

The cycle he proudly draws cuts deeper than he confronts. A cycle means models are trained on data shaped by their own prior decisions: a pricing model changes who buys, which changes the claims data the next model trains on. Adverse selection and self-referential training data are the elephant in his own diagram, and fraud detection adds adversarial drift on top — fraudsters actively adapt to the model. None of it is mentioned.

The verification stack has a hole exactly where it matters most. Golden signals catch latency and errors; a canary model that responds perfectly and predicts garbage passes every check described. There is no shadow deployment, no champion/challenger, no prediction-quality evaluation during rollout, and no rollback story. Backtesting is the closest thing, and it is pre-release only. Regulation is invoked three times — compliance, legal risk — yet explainability, audit trails, and how they'd defend a pricing decision to a regulator never appear, which for insurance is table stakes.

Still, the honest parts carry it. The Q&A confession is the actual sequence at most companies, said out loud. And the one agent-related detail is a quiet gem: when template changes are non-trivial — "you need to mock things differently at each and every model" — they started experimenting with AI agents to iterate over every model and add the tests. Agents as maintenance crew for test propagation: an experiment with no results reported, but the right shape of problem for them. Also conspicuous: a 2025 ML-in-production talk with zero mention of LLMs.

## Related pages

- [[Essentials from a Real-World Microservices Journey — Sander Hoogendoorn (Craft 2025)]] — the sibling Craft 2025 talk: Hoogendoorn preaches build-quality-in automation and small releases for services, and Pelle's "ML endpoints are microservices" is that discipline extended to models — arrived at, tellingly, by retrofit after shipping fast broke things, the wrong-reasons adoption Hoogendoorn warns about.
- [[In-House LLM Serving at Netflix]] — strengthens it from the LLM side: Netflix's finding that deployment strategies "break when I/O schemas change" is exactly the failure class Pelle attacks with contract and schema verification; together the two bracket the same production-serving discipline at classic-ML and LLM scales.
- [[Science and Statistics (Box)]] — deepens it conceptually: Box's "all models are wrong" and the motivated theory↔practice loop is the theory behind drift monitoring and proactive retraining; Pelle's talk is that loop implemented as infrastructure — minus the confrontation with the loop's own feedback term, models shaping their own training data.
- [[A New Era for Software Testing]] — parallels: antirez hands QA checklists to agents and Pelle experiments with agents propagating template tests across every model; and Pelle's backtesting-by-replay is the classic-ML sibling of antirez's regression checks — verification as the response to cheap generation.

---
*Sources: [[raw/machine-learning-in-production-adam-pelle-marshmallow-craft-2025]], [[summary/machine-learning-in-production-adam-pelle-marshmallow-craft-2025]]*
*Last updated: 2026-09-13*
