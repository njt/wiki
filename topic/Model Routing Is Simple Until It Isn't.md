# Model Routing Is Simple Until It Isn't

IBM Research's field report on why model routing — the seemingly straightforward idea of sending easy queries to cheap models and hard ones to expensive ones — becomes a systems optimization problem the moment you try it in production. Three dimensions make it hard: actual cost depends on caching infrastructure, not pricing sheets; task difficulty is invisible at routing time; and latency is dominated by serving conditions, not model speed. The team's solution treats routing as multi-objective optimization (cost, quality, latency) rather than classification, achieving 21% cost reduction and 9% latency reduction at a 4% accuracy drop on AppWorld.

---

## Key Quotes

> "Actual cost depends on the interaction between the model, the workload, and the serving infrastructure."

The central empirical finding, earned the hard way. GPT-4.1 has lower per-token pricing than Claude Sonnet 4.6, and Sonnet takes ~3× more reasoning steps — on pricing alone, GPT-4.1 should win. It doesn't. Sonnet ended up half the cost ($79 vs $155 on 417 tasks) because agent workloads reuse context blocks across steps, and Sonnet's cache-read pricing is dramatically lower. A router that only reads model pricing cards is optimizing against the wrong numbers. This is the article's most important data point, and it's the one that should make every agent builder reconsider their model selection heuristics.

> "Routers aren't solving one problem."

The article's diagnostic thesis, delivered after walking through the three dimensions. A production router juggles cost, latency, quality, specialization, reliability, compliance, data residency, privacy, and approved-model-list constraints simultaneously. Each of those dimensions can veto a routing decision. Framing routing as "which model is best for this task?" — the classification framing — assumes a single objective function. Production systems have at least five, and they conflict.

> "A router that ignores the serving system is optimizing against the wrong reality."

The latency corollary to the cost finding. Model speed is one variable; cache warmth, endpoint load, and hardware allocation often dominate. A theoretically faster model on a cold endpoint produces a worse user experience than a slower model on a warm one. The article's insistence that infrastructure state IS a routing input — not an implementation detail to be abstracted away — is its sharpest architectural claim.

> "Routing isn't really about choosing models. It's about optimizing systems."

The closing thesis. Models are one variable among many. Caching behavior, infrastructure state, compliance constraints, and workload patterns all matter. This reframes routing from an ML problem (classification) to a systems engineering problem (constrained multi-objective optimization), and it's a reframe that makes the article's findings generalizable beyond any specific model or benchmark.

## Key Themes

#concept **Routing as systems optimization, not classification.** The article's central reframe. Classification asks "which model is best for this task?" — a question that breaks down when cost, latency, and quality trade against each other and infrastructure state keeps changing. Optimization asks "what's the best operating point for the entire system given current conditions?" — a fundamentally different question that admits multiple valid answers depending on business priorities.

#concept **Cache economics as the hidden cost lever.** The GPT-4.1 vs Sonnet comparison is the killer exhibit. Token pricing is the number everyone quotes; cache-read pricing is the number that actually determines cost in agent workloads. This is a structural insight: agent trajectories are cache-friendly workloads (repeated system prompts, tool definitions, conversation prefixes), so the model with better cache economics wins regardless of base pricing. It's the same dynamic that makes [[KV Cache Locality]] a hardware multiplier rather than a tuning knob — caching turns cost optimization from a pricing-sheet exercise into a systems-design exercise.

#pattern **Difficulty is invisible at routing time.** A request that looks simple may require retrieval, compliance checks, tool use, and multiple refinement rounds. A request that looks complex may be handled well by a small specialized model. The article identifies this as the fundamental limit of difficulty-based routing: you can't classify what you can't see. This is a stronger version of the argument in [[SLM Routing for Knowledge Workers]] — Singh's nano-router works because knowledge-worker tasks have a detectable difficulty signal; IBM's finding is that agent tasks often don't, and pretending they do leads to expensive misroutes.

#pattern **Multi-objective routing with enterprise constraints.** The article's most practical contribution is naming the full set of constraints a production router faces: cost, latency, quality, specialization, reliability, compliance, data residency, privacy, approved model lists. Each can veto a routing decision. This is the list that separates research routers from production routers, and it's conspicuously absent from most routing papers.

#tool **Lightweight optimization at routing time.** The IBM team's router runs in ~6 ms and 2 kB per task. This is the engineering detail that makes the approach viable: you can't spend more on the routing decision than you save by making it. The optimization isn't doing anything fancy — it's just framing the problem correctly and solving it with the right objective function.

## Critical Analysis

This is one of the most practically useful agent infrastructure posts of 2026, and it earns that status by being honest about what breaks. The article doesn't propose a new algorithm — it diagnoses why the obvious algorithm (difficulty-based classification) fails in production, then shows that the right framing (multi-objective optimization) solves the problem with lightweight tooling. The honesty is the contribution.

**The GPT-4.1 vs Sonnet cost comparison is the article's strongest empirical claim and also its most fragile.** It's a single benchmark (AppWorld), a single agent harness (CodeAct), a single point in time. Cache pricing changes. Model pricing changes. The specific numbers will rot. But the structural insight — that agent workloads are cache-friendly and cache economics dominate per-token pricing — won't. This is the same structural dynamic that [[Inference Cost Napkin Math]] identifies at the hardware level (memory bandwidth, not compute, is the bottleneck) and that [[KV Cache Locality]] quantifies at the infrastructure level (22% more throughput from prefix-aware routing). The article is one data point in a converging story: caching is the dominant cost variable in LLM systems, and anyone optimizing for anything else is optimizing for the wrong thing.

**The article leaves the hardest problem unsolved.** It tells you that difficulty is invisible at routing time, then solves the problem with multi-objective optimization that still needs *some* estimate of task difficulty to work. The optimization approach is better than classification, but it still requires a quality signal — and the article doesn't explain how to get one for arbitrary tasks. This isn't a criticism so much as an honest boundary: the team is saying "here's what we solved and here's what we didn't," and the unsolved part is genuinely hard.

**The comparison to [[Devin Fusion]] is instructive.** Both articles argue that multi-model architectures are the end of single-model agent design. But they arrive from opposite directions. Cognition's approach is architectural — co-resident models with cache-aligned switching — while IBM's is mathematical — multi-objective optimization over a cost-accuracy-latency frontier. Devin Fusion's key insight (switch models at compaction boundaries to make the switch free) is an engineering detail; IBM's key insight (the objective function matters more than the classification algorithm) is a mathematical one. They're complementary: Fusion tells you *when* to switch, IBM tells you *what to optimize for* when you do.

**The biggest omission is the failure mode.** The article shows you the cost-accuracy frontier but doesn't tell you what happens at the bad operating points. When the optimizer makes a wrong call — routes a hard task to a cheap model that fails, or routes an easy task to an expensive model that wastes money — what's the blast radius? How do you detect and recover? The article's router is lightweight, which is good, but lightweight routers are also brittle routers. The omission of failure analysis is conspicuous given how meticulously the rest of the article catalogs what breaks.

**The enterprise constraint list is important and underdeveloped.** Compliance, data residency, privacy, and approved model lists appear once and are never mentioned again. But these are the constraints that kill routing projects in practice — you can have a perfect cost-accuracy frontier and still get blocked because your cheapest model isn't on the approved list. The article names the problem but doesn't explore it, which suggests it's a future-work item rather than an oversight.

**What's genuinely new here:** The reframe from classification to optimization is the article's lasting contribution. Most routing work — including [[SLM Routing for Knowledge Workers]], the [[The Advisor Strategy]], [[Thrifty (Tiered Delegation for Claude Code)]], and [[Devin Fusion]] — operates within a classification or escalation paradigm: decide which model handles a task, then commit. IBM's argument is that you shouldn't commit to a model. You should commit to an *operating point* on a cost-accuracy-latency frontier, and let the optimizer pick the model that achieves it under current conditions. This is a more general framework that subsumes the others — Advisor, Thrifty, and Fusion are all specific operating-point choices within a broader optimization space.

---

*Sources: [[raw/model-routing-is-simple-until-it-isnt]]*
*Last updated: 2026-07-18*
