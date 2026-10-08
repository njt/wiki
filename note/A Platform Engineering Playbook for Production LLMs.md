# A Platform Engineering Playbook for Production LLMs

A retail platform team's field report on running a multi-agent LLM inventory-accuracy system in production: hallucination rate fell from 15% to 1.5% not by swapping models but by building a shared LLM platform layer — gateway, coordinator, schema enforcement with classified retries, prompt registry with rollback, intent-validation routing, MCP-layer RBAC, per-team cost attribution, and golden-set evals. The companion repo is deliberately runnable end-to-end on a local Ollama model with no hidden abstractions.

---

## The number that frames everything

The article's hook is the cleanest possible statement of the platform thesis:

> By the end of our first month running an LLM-driven inventory recommendation system in production, roughly fifteen percent of agent responses were hallucinations... Six months later the rate was 1.5 percent and the lever that moved the number wasn't a better foundation model. It was the decision to stop treating the LLM stack as an application concern and start treating it as platform infrastructure.

That's a 10x improvement with the model held constant. The entire playbook is the itemisation of what the platform layer contributed.

## Key quotes

> A hallucinated response isn't a stack trace. It passes every traditional health check. The downstream service consumes it, takes action on it, and only much later does someone notice the suggested output doesn't exist.

The central diagnostic insight. LLM failures are semantically valid, which is why APM tools — "latency, error rate, and throughput" — tell you whether the model is reachable but "nothing about whether it's behaving correctly." The author calls retrofitting per-team token attribution onto a live path "weeks of engineering time," which is why the metric dimensions must exist from request ingress.

> The decision point for building this platform isn't the first LLM application. It's the second.

A concrete adoption rule, mirroring exactly how auth/logging/service-mesh centralisation decisions are traditionally framed. By the tenth application, retrofitting "is long and difficult migration work."

> LLM systems let you defer infrastructure decisions in ways traditional systems do not. They do not crash. They do not return 500 errors. They quietly produce slightly worse output. That gap between a wrong decision and its visible consequence is why platform thinking pays off up front.

The best single paragraph in the piece: the reason platform engineering matters *more* for LLMs than for ordinary services is that the failure-signal delay is longer, so undisciplined teams dig a deeper hole before noticing.

> That is exactly how an agent ends up handling a question it wasn't built for, leading to hallucinations.

On the intent gate returning "unclassified" rather than defaulting to the highest-scoring agent. A small design decision with outsized hallucination payoff — refusal as a first-class routing outcome.

> We track two key metrics at this layer: recovery rate to ensure our retry loop actually works and amplification factor to guarantee it stays affordable.

Retry loops need their own telemetry. Blind retries are "actively harmful" — a repeated prompt "yields a different hallucination," and naive retries during provider outages become traffic storms. Hence the three-way failure classification (schema / hallucination signal / infrastructure), each with its own strategy, under a hard per-request budget.

## Key themes

- #concept — Hallucination rate as a platform-controllable metric, not a model property
- #pattern — Structured outputs as instrumentation: the `grounded`/`out_of_scope` flags "cost nothing to generate but force the model to commit" in machine-readable form, making a schema violation "the highest-precision hallucination signal a platform has"
- #pattern — Prompt-as-executable-text: registry, versioning, runtime resolution, rollback without redeploy, eval-gated promotion
- #tool — ADK (orchestration), LiteLLM (model seam), Ollama, FastMCP (tool servers), Pydantic (schemas), OpenTelemetry (metrics)
- #pattern — Security enforced at the resource server: "relying solely on API gateway checks means any single bug in your orchestrator can instantly expose every connected tool"

## Opinionated take

This is one of the most actionable production write-ups in the wiki because every claim is backed by runnable code rather than a diagram-and-vibes architecture. Three things stand out.

First, the honest limitations are unusually credible: the prompt registry is in-memory (acknowledged as demo-only, production needs a database and per-environment version binding), and the repo's deterministic intent classifier is a stand-in for a trained one. The author says where the scaffold ends.

Second, the "unclassified" routing gate deserves more attention than it gets in most multi-agent writing. Most topology literature obsesses over which agent handles a request; this piece argues the more important question is when *nobody* should, and that refusal is a hallucination-prevention mechanism, not a UX failure. That inverts a common default: routers tuned to always answer are hallucination generators by construction.

Third, a quiet tension: the piece advocates centralising everything in a platform, but its own architecture shows the platform is mostly *discipline* (schemas, retries, registries, tags) rather than shared machinery. A sceptical reader could ask how much of the 15%→1.5% came from the platform per se versus from the grounding pipeline — tools as the only permitted evidence source, restricted prompts, and declaration flags. Probably most of it. That doesn't undermine the thesis so much as sharpen it: the platform's real product is enforced discipline, which is exactly why it can't live in N copies of application code.

The grounding pipeline itself — "Tools fetch the evidence, the prompt restricts the model to it, the schema forces a declaration and validation turns a broken declaration into a counted, retryable failure" — is a four-stage pattern worth stealing wholesale.

---

## Related pages

This source is the production-grade, enterprise-scale companion to [[Platform Engineering as the AI Control Plane]] — that piece argued platform teams are absorbing AI toolchain ownership (model approval, cost governance, agent auth); this one shows what those responsibilities concretely look like when a platform actually ships them, down to the JWT role claim that carries the team dimension into every metric.

It strengthens [[Managing AI Coding Costs at Scale]] on the attribution problem: Databricks' playbook treats per-team token cost as the prerequisite for any optimisation, and this article supplies the mechanism — tags set at request ingress via authenticated role, recorded at every model call site, because the dimension "cannot be rebuilt later from logs."

It nuances [[No Escape, No Leak — Egor Kraev on Structured Objects]]: Kraev argues every structured-output approach leaks and the validator must be the only exit; this article's schema-with-retry loop is exactly the "retries leak" case, but shows the leak repurposed as a feature — validation failure as the highest-precision hallucination signal, counted and retried rather than escaped.

It complicates [[Multi-Agent Systems Have a Distributed Systems Problem]]: Meiklejohn argues multi-agent systems inherit distributed-systems failure modes with no coordinator discipline; this article's coordinator pattern is the practitioner's answer — "without a central owner to manage circuit breakers, enforce timeout budgets, and validate data between transitions... agents blindly retry the components above them, turning a single isolated failure into a system-wide crash."

---
*Sources: [[raw/platform-engineering-playbook-production-llms]], [[summary/platform-engineering-playbook-production-llms]]*
*Last updated: 2026-10-08*
