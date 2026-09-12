# Ramp — Lessons from Building a New AI Product

Four Ramp engineers at the Pragmatic Summit on what they learned building AI-native finance products. The through-line: coding was never the hardest part, and AI makes that truth impossible to ignore.

## Precis

Ramp has 50,000+ customers and several years of shipping AI features into production finance workflows. This talk distills what actually worked: a single agent orchestrating thousands of tools rather than a thousand micro-agents, an eval culture that started with five data points, a tool catalog as shared infrastructure, and an internal coding agent that now authors over half of merged PRs. But the sharpest material is Ian's closing contrast between "Team A" (impact-obsessed, comfortable with ambiguity) and "Team B" (bikesheds libraries, builds before understanding). His argument: AI is a 10× amplifier, not a substitute for judgment, and the teams that mistake coding speed for engineering quality are about to build bigger messes faster than ever.

## Key Quotes

> "You don't need to build a thousand agents. … Instead you want to drive your framework towards a single agent with a thousand skills."
> — Nick

This is the architecture thesis of the entire talk. Every process reduces to the same pattern: event → prompt → guardrails → context → tools. The agent framework becomes an operating system for skills, not a zoo of specialized bots. It's the same insight behind [[Fleet Supervisor (sermakarevich)]], [[Agent Orchestration]], and the [[The Agentic Product Standard v2.0]] autonomy ladder — coordination beats specialization when the coordination layer is thin enough.

> "We thought that the users would be correct. … But turns out the users are actually incorrect. They're wrong."
> — Will

The most honest thing said about building AI products. Finance approvers approve out-of-policy expenses from laziness, trust, or ignorance. Ramp's solution was cross-functional labeling sessions that produced ground truth — they defined correctness themselves rather than treating user behavior as the oracle. This is the same dynamic [[Guardrails and Feedback Loops]] describes: deterministic enforcement beats instructions. If your training signal comes from users who are systematically wrong, your eval dataset is poisoned.

> "A smaller black box becomes a bigger black box as the system becomes more complex."
> — Will

The traceability tradeoff as systems graduate from simple prompts to autonomous agents. Ramp's answer: assume only inputs and outputs are knowable, verify correctness regardless of internals. This is the [[Smart Models Dumb Pipes]] pattern applied to product engineering — audit the outcome, not the reasoning chain.

> "You could still build the wrong thing just a lot faster and you can build bigger messes."
> — Ian

The talk's thesis statement. AI amplifies velocity without improving direction. [[Software Engineering at the Tipping Point]] makes the same argument about the 10× amplifier effect. [[Product-Minded Engineers in an AI-Native World]] gets at the same thing from the other side: taste and judgment are the scarce resources now, not coding throughput.

> "Software is perpetually not finished. And so with all this extra capacity … companies are just going to chase opportunities they couldn't afford to pursue."
> — Ian

A genuinely optimistic take in a talk that's mostly hard-edged. The capacity AI creates doesn't eliminate work — it raises ambition. This is the counterargument to doomerism and it's worth sitting with: the constraint was never ideas, it was implementation cost.

## Key Themes

### #pattern Single Agent, Many Tools

Ramp tried building separate agents for every job-to-be-done. They ended up with four different approaches and a mess. The winning pattern: one agent framework that can invoke thousands of tools, triggered by events, guided by guardrails. This converges with [[Agent Orchestration]] patterns (planner/worker/judge), [[Agent-Native Architectures (Every)]]'s composability principle, and the tool-catalog approach in [[Components of a Coding Agent]]. The insight isn't new — it's the same reason monoliths beat microservices when the team is small — but it's validating to hear from a company operating at Ramp's scale.

### #tool Internal Tool Catalog

Hundreds of tools, targeting thousands. Built alongside product teams, shared across internal repos and core products. A PM vibe-coded ~20 tools themselves. Gaps become immediately visible because every team building an agent needs the same underlying capabilities. This is what [[Loop Engineering]] calls "automations + connectors" — the substrate agents compose over. The difference is Ramp treated it as shared infrastructure from day one, not something each team builds ad hoc.

### #pattern Evals from Day One

Started with five data points. Integrated into CI. Made results instantly readable. Added online evals (rate of "unsure" decisions) as a live health check. Evals are what let them swap models confidently — Opus 4.6 to GPT 5.3 with one-line config changes. This mirrors the eval pyramid in [[The Agentic Product Standard v2.0]] and the CI-integrated approach of [[OpenCodeReview]]. The detail that matters: context rot is real. Tool instructions and docstrings drift into conflict, and only evals catch it.

### #tool Ramp Inspect

Internal background coding agent: Modal sandboxes, full repo context, Datadog, read replicas, VS Code, Chrome dev tools, MCP. Runs 150,000+ tests, responds to CI failures, patches fixes before pinging the user. Authors over 50% of merged PRs. "Multiplayer first" design so non-engineers can pair with it. Usage spans engineering, product, design, risk, legal, finance, marketing, and CX. The open-sourced blueprint at builders.ramp.com makes this more than a brag — it's a reference architecture for [[Agent Coding Workflow]] at enterprise scale.

### #concept Team A vs. Team B

Ian's thought experiment is the talk's most useful artifact:

- **Team A:** Cares about impact, handles ambiguity, understands product/business/data, adopts new tools, finds creative solutions, obsesses over user experience
- **Team B:** Debates libraries, adds process under chaos, complains about headcount, bikesheds details, builds before understanding, focuses on performative code quality

AI accelerates both teams. Team A builds better products faster. Team B builds wrong products with impressive test coverage. The differentiator isn't AI skill — it's judgment, context, scar tissue from experience, and the ability to see around corners. [[Running an AI-Native Engineering Org]] and [[Product-Minded Engineers in an AI-Native World]] are the same argument from different angles.

### #pattern Autonomy Slider

Let customers dial agent autonomy from "suggestions only" to "auto-approve under $20." Trust builds incrementally. The expense policy becomes a living document — edit it like `.cursorrules` and see immediate behavior changes. Finance people loved this once they experienced the feedback loop, despite initial terror. This is the productization of [[The Agentic Product Standard v2.0]]'s autonomy ladder.

## Critical Analysis

**What's genuinely new vs. what's validating.** The single-agent-with-many-tools architecture isn't novel — [[Agent Orchestration]] has been converging on it for a year. What's valuable here is the production validation at scale: 50,000 customers, Fortune 500 design partners, real money flowing through these systems. Architecture opinions become architecture facts when they survive that.

**The unanswered questions matter.** The summary author flags them: what happened on February 6th? How are hallucinations actually prevented in financial decisions? What are the real costs? What specific failures occurred? The talk is a conference presentation — it's going to be curated. But for a talk about "lessons learned," the absence of near-miss stories is conspicuous. Every production AI team has them. Not sharing specifics makes the advice harder to operationalize.

**The Team A/Team B dichotomy is useful but incomplete.** It's a motivational framework, not an engineering one. Real teams contain both types, and real people move between them depending on context, fatigue, and organizational incentives. The sharper point — that AI amplifies existing team dynamics rather than leveling them — is true and underappreciated. But the binary framing risks becoming the thing it critiques: performative culture posturing.

**50% of merged PRs is a number to watch, not celebrate.** It tells you about adoption, not quality. [[The End of Code Review]] wrestles with what happens when that number crosses the threshold where human review becomes indefensible. Ramp Inspect patches CI failures before pinging the user — that's the right loop. But the metric that matters isn't PR authorship percentage; it's whether the codebase is getting better or worse over time.

**The tool catalog is the most underplayed idea in the talk.** "Hundreds of tools, aiming for thousands" sounds like infrastructure. It's actually product strategy. Every tool is an API that an agent can compose. The tool catalog IS the platform. Teams that treat internal tools as second-class citizens will watch their agent efforts fragment into the thousand-micro-agent mess Ramp escaped. [[Mirage (VFS)]] and [[Agent-Native Architectures (Every)]] are thinking about this from the right end — the interface layer is what makes composition possible.

## Related

- [[Agent Orchestration]] — Multi-agent coordination patterns; Ramp converged on single-agent orchestration
- [[Guardrails and Feedback Loops]] — Evals, CI integration, deterministic enforcement over instructions
- [[Agent Coding Workflow]] — Ramp Inspect as the practitioner's loop at enterprise scale
- [[Agent-Native Architectures (Every)]] — Tool catalogs as composable primitives
- [[The Agentic Product Standard v2.0]] — Autonomy ladder productized as a slider
- [[Smart Models Dumb Pipes]] — Audit outcomes, not reasoning chains
- [[Software Engineering at the Tipping Point]] — AI as 10× amplifier, Team A/B divergence
- [[Running an AI-Native Engineering Org]] — Internal AI service, dogfooding as culture
- [[Product-Minded Engineers in an AI-Native World]] — Judgment and taste as scarce resources
- [[The End of Code Review]] — What happens when agents author most PRs
- [[Fleet Supervisor (sermakarevich)]] — Production agent orchestration that parallels Ramp's approach
- [[Components of a Coding Agent]] — The harness matters more than the model
- [[The PM's Playbook for Shipping AI Features]] — Tight feedback loops, autonomy gradients
- [[Loop Engineering]] — Event-driven agent workflows as the meta-skill
- [[Uber — Agentic Engineering Shift]] — Another enterprise AI transformation, similar scale

## Source

The Pragmatic Engineer (YouTube), "Ramp: Lessons from Building a New AI Product - The Pragmatic Summit," May 2026. Summarized and transcribed by njt. [Gist](https://gist.github.com/93839a006849961f384afc60b1dcbf9f) | [Video](https://www.youtube.com/watch?v=NMs8C2_3M0w)

*Fetched 2026-07-04*
