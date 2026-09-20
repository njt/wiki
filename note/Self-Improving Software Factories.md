# Self-Improving Software Factories

Warp's case that agentic engineering only becomes tractable when the whole SDLC runs as a cloud-hosted, code-defined factory that measures its own agents and edits its own configuration.

---

## The argument in one paragraph

Warp claims that the hand-waving around "which agents and models are best" dissolves only when an organization runs a closed-loop software factory: agents execute the SDLC in the cloud, every run is traced and stored, scorers grade those runs on a schedule, self-improvement agents read the grades and propose diffs to the version-controlled factory definition, and humans merge those diffs as PRs. The falsifiable core is that measurable, compounding improvement in agent output requires this specific shape — code-defined, cloud-centralized, API-first, with machine-generated configuration changes — and that anything local-first "is not really a factory." If teams achieve equivalent measured improvement with ad-hoc local setups, or if self-improvement agents' diffs turn out to be noise that humans constantly reject, the thesis fails.

---

## Key quotes

> The solution is to set up a closed-loop system in the cloud where all of your agents are tracked and measured *against your own data and workflows*, so you can adjust your setup based on actual data and not vibes.

The framing move of the whole piece: the problem isn't picking the right agent, it's building the apparatus that makes picking measurable. "Against your own data" is the load-bearing phrase — generic benchmarks are explicitly insufficient.

> If someone describes a factory product that is local-first, it's not really a factory.

Definition-by-marketing, and the most aggressive line in the piece. It conveniently excludes every competitor and every individual developer's setup, and it does real argumentative work: cloud-centralization is what makes the data collection possible.

> Factory metrics are related to DORA metrics but more granular. DORA measures externally visible productivity around how fast you deploy, and how high quality the deployments are. Factory metrics measure inner loop metrics around the velocity and cost of building the features that get deployed.

A genuinely useful distinction. DORA tells you what shipped; factory metrics tell you what the machine that ships it costs and how much of it runs without humans. The catch is that two of the five proposed metrics ("savings over human work," "acceleration of shipped product") are admitted estimates or "hard to measure."

> Crucially, having your factory defined as code makes it easy for agents to suggest diffs. This is the basis of self-improvement: agents that observe how your factory is working and suggest changes to the underlying models, skills, etc.

The insight that ties the piece together: infrastructure-as-code isn't just an ops nicety, it's the interface that makes the system legible to its own agents. A factory that isn't code is a factory that can't be edited by the thing running it.

> Self-improvement loops are effective but they don't provide true A/B testing of different configurations. They are more like "what would an intelligent person change by looking at past results to improve the system."

The most honest sentence in the article. It concedes that the headline self-improvement loop is an intelligent heuristic, not an experiment, and motivates the benchmark system as the actual causal instrument.

---

## Critical analysis

The non-obvious contribution is the taxonomy of improvement mechanisms. Most factory writing stops at "agents do the SDLC"; this piece separates three distinct loops that are usually blurred: scorers grading runs (evaluation), self-improvement agents proposing diffs (learning from history), and benchmarks running reference tasks across configurations (experimentation). The explicit admission that the second is not a substitute for the third is more rigorous than most vendor writing manages, and the benchmark matrix — pick 5–10 representative tasks from your own past runs, vary the model routing, grade with the same scorers — is a concrete, stealable procedure.

The weaknesses are the ones you'd expect from a product document wearing an engineering costume. There is not a single number in the piece: no cost of running scorers ("these scorers cost money to run, so you should be thoughtful" is the entire treatment), no example diff a self-improvement agent ever produced, no evidence the loop improves anything. "Savings over human work" is an estimate of a counterfactual — what a human would have cost on a similar issue — which is exactly the kind of metric that flatters whatever built it. And the self-improvement loop has a Goodhart problem the piece never names: scorers are LLM-as-a-judge prompts, the self-improvement agents optimize against those scores, and the factory definition drifts toward whatever the judge rewards. The human merge gate on factory-definition PRs is the only backstop, and the article treats it as a feature rather than as the point where the "self-improving" claim quietly becomes "human-improving with machine-suggested diffs."

What's left out: failure modes of the loop itself. What happens when a self-improvement agent's diff degrades throughput and the scorers only notice two weeks later? Who audits the scorers for drift? What is the rollback story when a merged factory PR is bad — version control gives you the mechanism, but nothing about detecting that you need it. The related Warp posts listed at the bottom ("The missing feedback loop for software factories") suggest the company knows this; this piece doesn't go there.

---

## Related

- [[Cloud Software Factories]] — Strengthens: Zach Lloyd's blueprint defines the factory pipeline (triage → spec → implement → review → verify → ship) but treats measurement as infrastructure plumbing; this piece supplies the missing layer — scorers, self-improvement agents, benchmarks — and pushes the same "must be cloud" thesis harder.
- [[How to Build an AI Software Factory]] — Complicates: Firecrawl's five-stage control plane runs from intake to merge gate with no self-improvement stage at all; Warp names and operationalizes exactly the loop that synthesis leaves implicit, while sharing its character as a vendor document whose every stage resolves to its own product.
- [[Inside a Software Factory]] — Nuances: Iusztin's third bucket is "self-improving" and he warns about when a factory becomes overbuild; Warp takes that bucket seriously enough to build primitives for it but drops the caution entirely — its closing argument is that waiting to build this is how you fall behind.

---
*Sources: [[raw/agent-self-improving-software-factories]], [[summary/agent-self-improving-software-factories]]*
