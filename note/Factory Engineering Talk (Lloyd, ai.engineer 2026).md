# Factory Engineering (Zach Lloyd, ai.engineer 2026)

Zach Lloyd's ai.engineer talk defines the "software factory" as a code-defined, closed-loop system for continuously measuring and improving cloud coding agents — with declarative factory definitions, full agent-run recording, online evals scored by other agents, benchmarks on your own data, and self-improvement that takes the form of agents PR-ing against the factory definition itself.

---

Lloyd opens with the transitional-state diagnosis: most teams still run interactive local agents, one or a few per developer, and that caps everything. Interactive prompting limits you to human throughput; per-developer choices of models, skills, and setups defeat cost and productivity optimisation; and it creates a security and governance mess. His audience show-of-hands already shifts — more people are building internal factories than working purely interactively.

The definition is precise and worth quoting: a software factory is "a code-defined closed-loop system for continuously measuring and improving cloud coding agents." And he is equally clear on what it is *not* — not loose automations, not AI teammates, not a shared skills/MCP bundle, not personal agent swarms. Those are primitives, not the end state.

---

## Key quotes

> "A code-defined closed-loop system for continuously measuring and improving cloud coding agents."

Declarative encoding is the load-bearing part: branching and rolling back factory configurations, and — crucially — opening the way for self-improvement, where coding agents suggest changes to the factory itself as PRs against its definition.

> "The way that I would think about inference and models in a factory is they are a degree of freedom to test."

Model and harness choice become benchmark variables, not commitments. Your platform should support multiple models and multiple harnesses ("Cloud Code might be better for some things. Codex, Pi, Warp…"), including your own organisational harness. This commoditises the model choice in a way few teams have operationalised.

> "If you're just going on vibes in terms of how you deploy coding agents, you can't be confident that you're actually getting the best, highest quality outputs at the best cost."

The measurement mindset is the talk's real thesis: record every conversation, artifact, screenshot and video; run online evals scored by a second agent (his example: catching redundant tests); replay benchmarks on your own real data; A/B test factory configurations.

> "Engineer your agents. It's a new engineering primitive."

The reframe that closes the talk: there is a new fundamental engineering problem — building the system around the primitive — and a new toolset to master (evals, benchmarks, self-improvement loops, A/B testing the factory). He names this "factory engineering" and insists it coexists with, not replaces, product engineering.

---

## Key themes

#concept factory engineering as a distinct discipline — improving the system that builds the product, not just the product
#pattern factories-as-code: Terraform-style declarative definitions enabling branch/rollback/agent-proposed factory changes
#tool online evals scored by a reviewing agent, plus self-serve benchmark replay on real organisational data
#person Zach Lloyd, Warp founder, arguing his own product category into existence

---

## Opinionated take

The strongest move here is the strict negative definition. Most "factory" content in the wild describes loose automations or agent swarms; Lloyd fences those off as primitives and insists on measurability and declarativeness as the dividing line. That's a real maturity bar, not marketing — though it is unavoidably marketing, since every layer he describes is a feature Warp sells. The talk is the roadmap for Warp's product; the honest reading is "this stack is right *and* conveniently ours."

The genuinely novel contribution is self-improvement as PRs-against-the-factory-definition. Because the factory is code, the same agents that build software can diagnose their own harness and submit fixes — a recursion loop that most teams haven't even imagined, let alone gated. The unaddressed risk: who reviews the factory-defining PRs, and what happens when an agent optimises the factory for the evals rather than for shipped software? His own "if you're going on vibes, you're flying blind" cuts both ways — the eval scores are themselves vibes until validated against real outcomes.

There's also an unstated tension between "measurement mindset" and the human cost. He acknowledges "angst around what it means to be an engineer" but waves it off in two sentences. Compared to sources that take the comprehension-debt and judgment-relocation problems seriously, this is the talk's thinnest moment.

---

## Related pages

This talk strengthens [[Two Jobs — Product Engineering and Factory Engineering (Lloyd)]], giving the talk-stage articulation of the factory-engineering job that essay names. It nuances [[Cloud Software Factories (Zach Lloyd)]], the earlier blueprint, by adding the eval/benchmark/self-improvement tooling layer. It complicates [[Human Judgment Doesnt Leave the Software Factory — It Relocates]]: Lloyd's self-improvement loop relocates judgment into agent-scored evals, raising exactly the question of where human review of factory-defining PRs sits. And it pairs naturally with [[Five Mistakes People Make with Agentic]], whose "factory mode" end state this talk operationalises.

---
*Sources: [[raw/zach-lloyd]], [[summary/zach-lloyd]]*
*Last updated: 2026-11-26*
