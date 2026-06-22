# The Case Against Building Your Own Agent Platform

Pete Johnson's sharp field guide to the build-vs-buy decision for AI agent infrastructure, published on O'Reilly Radar. The core argument: agent platforms require deep investment in four components most teams underestimate (memory, governance, eval, orchestration), each of which is maturing into a standalone product category. Building made sense in 2024 when 47% of enterprise AI was in-house; by late 2025 that collapsed to 24%. The pendulum is swinging fast, and Johnson provides five diagnostic questions that function as a build-vs-buy triage tool. His most useful distinction: building *agents* on platform components is often valid; building the *platform components themselves* almost never is.

## Key Quotes

> "The cost of building this is almost always estimated before anyone has a clear picture of what 'this' actually is."

This is the essay's organizing insight. Agent platforms suffer from an especially vicious version of the planning fallacy because the category boundaries aren't stable yet — you're estimating the cost of something whose definition shifts while you build it.

> "Memory sounds like a database problem. It isn't."

Johnson's breakdown of memory into episodic, semantic, and procedural systems with distinct retention policies is the most compressed taxonomy I've seen. The competitive landscape data (21 frameworks, three hosting models, Mem0 at $24M raised) makes the case that this is already a category, not a feature.

> "RBAC was designed for humans with predictable intent. Agents don't have predictable intent."

The governance section is the strongest part of the essay. Action authorization vs. data authorization, behavioral drift detection, tiered autonomy — these aren't checkbox compliance items, they're open research problems. The Grant Thornton stat (78% of execs would fail an AI governance audit within 90 days) and the August 2026 EU AI Act deadline make this urgent, not theoretical.

> "Your eval system needs its own eval system."

The recursive trust problem: if you're building eval infrastructure, you need meta-eval to verify your evals aren't drifting, gaming, or missing failure modes. Google Vertex AI already standardized trajectory-based metrics that didn't exist 18 months ago. The ground is moving.

> "If you built your own orchestration layer in 2024, you're rewriting it in 2026."

Orchestration hasn't converged — LangGraph, CrewAI, OpenAI Agents SDK, AutoGen, Google ADK, Claude SDK, and Microsoft's Agent Framework each represent fundamentally different bets. Migration between them means rewriting most of your agent logic. This is the strongest argument against building: you're betting on an architecture when the field hasn't settled on what the right architecture is.

> "Build the things that are specific to your business. Buy the things that are specific to the technology category."

The essay's heuristic in one sentence. Agent memory, governance, eval, and orchestration are technology-category problems — they're the same for everyone, which means they benefit from specialized vendors and open-source communities. Your business logic is where differentiation lives.

## Key Themes

#build-vs-buy #agent-platform #memory #governance #eval #orchestration #platform-engineering #pattern

## Critical Analysis

This is one of the more useful articles I've read on agent infrastructure because it's specific about *what* you're building when you "build a platform." Most build-vs-buy pieces are hand-wavy; Johnson names the four components and gives each a paragraph of diagnostic criteria. The five questions at the end function as a genuine triage tool — I'd use them in a platform strategy meeting tomorrow.

**What it gets right:** The workflow-vs-agent distinction is the most important scope decision in this space, and Johnson is correct that conflating them is where cost overruns originate. The model-swap question (question 3) is also underrated — Menlo's data on the Anthropic/OpenAI spend flip shows how fast model preferences shift, and hardcoding assumptions about any one model's behavior is a rewrite waiting to happen.

**What's missing:** The essay says almost nothing about open-source platform components. The build-vs-buy framing assumes "buy" means commercial vendor, but there's a third path — assemble from open-source components (Mem0/Letta for memory, OpenRewrite for eval scaffolding, etc.) that gives you control without the full build cost. This is the path most sophisticated teams actually take.

**The tension Johnson doesn't address:** If every component is still maturing and orchestration "hasn't converged," then buying also means betting on a vendor whose architecture might be wrong. The risk of building is that you build the wrong thing; the risk of buying is that you marry the wrong thing. Johnson's "build business logic, buy category stuff" heuristic is right, but it assumes the category stuff is ready to be bought — and his own evidence suggests it isn't, quite.

**The real insight, buried in question 5:** "What happens when the platform team leaves?" This isn't really about attrition — it's about whether your platform is a *system* or a *person*. If the platform only works because a specific engineer understands its quirks, you haven't built a platform; you've built a dependency. This applies equally to building and buying: if you can't onboard a new engineer to your agent platform in a week, you don't have a platform.

## Related Pages

- [[Agent Orchestration]] — Hub page for multi-agent coordination patterns; Johnson's orchestration critique maps directly to this taxonomy
- [[Agent Memory and Context]] — Deep dive on the memory problem Johnson identifies as "not a database problem"
- [[Guardrails and Feedback Loops]] — The eval and quality dimension Johnson covers
- [[The Minimum Viable Unit of Saleable Software]] — Brandur's build-vs-buy economics with hard numbers; companion piece to Johnson's framework
- [[Elements of Agentic Systems Design]] — Ten-element taxonomy that overlaps with Johnson's four components
- [[The Agentic Product Standard v2.0]] — What a platform actually needs to be; the other side of the build decision
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE; the production reality Johnson is warning about
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management as the real challenge; a case study in building what you shouldn't
- [[Smart Models Dumb Pipes]] — The architectural philosophy behind Johnson's "buy the category stuff" advice
- [[Loop Engineering]] — Addy Osmani on the meta-skill of designing systems that prompt agents; relevant to the platform engineering dimension
- [[Maybe Coding Agents Don't Need a Bigger Memory]] — Context ≠ continuity; challenges the memory-as-database assumption from a different angle

---
*Source: [O'Reilly Radar](https://www.oreilly.com/radar/the-case-against-building-your-own-agent-platform/), June 17, 2026. Fetched 2026-06-22.*
