---
url: https://www.oreilly.com/radar/the-case-against-building-your-own-agent-platform/
date_fetched: 2026-07-05
backfilled: true
---

You know the meeting. The board wants an AI agent strategy by end of quarter. Someone on the leadership team has read a McKinsey report. You’ve been voluntold to build the platform. The slide deck says “AI-native.” The acceptance criteria are vague. Somebody mentions LangGraph, and somebody else says, “We’ll just wrap it ourselves.”

You ask what “done” looks like. Nobody in the room can answer.

The cost of building this is almost always estimated before anyone has a clear picture of what “this” actually is. And that’s the problem I want to work through here, because the scope of the work being casually assigned to internal platform teams right now is genuinely larger than the people assigning it understand.

**Build versus buy, flipped in a year**

This particular pendulum has swung before. App servers in the late 1990s. Content management systems in the 2000s. Container orchestration in the 2010s. The pattern rhymes every time: When a category is new, the components look deceptively simple. Early adopters build their own. The market catches up. Within 18 months, building becomes the expensive path. Within 36 months, the teams that built internally are rewriting on top of the category winner that emerged while they weren’t looking.

What’s different about the current moment is the speed. Menlo Ventures’ *2025 State of Generative AI in the Enterprise* report shows the build-versus-buy split inverted in a single year. In 2024, 47% of enterprise AI solutions were built internally. By late 2025, that number had collapsed to 24%. The market made the decision in 12 months, which is unusual.

I’ve lived through enough of these transitions to recognize the shape. What I want to do in this piece is explain why I think the scope of “agent platform” is systematically underestimated right now, and what platform engineers should be asking before they commit to building one.

**Most “agent platforms” aren’t**

A lot of the projects labeled “agent platform” right now are actually workflow systems with an LLM in the loop. That’s a meaningful distinction. As Anthropic pointed out in its “Building Effective Agents” guidance, workflows are systems where LLMs and tools are **orchestrated** through **predefined code paths**. Agents are systems where LLMs **dynamically** direct their **own processes and tool usage**.

Most of what enterprises are shipping today sits on the workflow side. That’s fine. Workflows have bounded requirements, tractable testing, and predictable failure modes. If your team is building a workflow system, you might reasonably build it yourselves.

The trap is that teams start building for workflows, then get asked to support agents, and discover the jump isn’t incremental. Agents need memory that survives across sessions. They need evaluation that handles nondeterminism. They need governance that tracks actions, not just outputs. They need orchestration that recovers from failure modes a workflow engine never sees.

Here’s the thesis I want to put on the table: The decision to build an agent platform almost always underestimates the long tail. Memory, governance, eval, and orchestration aren’t features you add to a workflow engine. They’re separate product bets, each with its own maturity curve, its own vendor landscape, and its own team of specialists who’ve been working on it full-time for 18 months while you’ve been doing something else.

Let me walk through them.

### Memory

The assumption inside most build proposals is that memory is a database problem. You’ll pick a vector store, shove conversation history into it, and retrieve relevant chunks when the agent needs context. Done.

Production memory is three separate systems: episodic, semantic, and procedural, each with different retention and retrieval policies. It’s temporal reasoning that tracks when facts were valid, not just what they were. It’s deduplication, multitenant isolation, and explicit source-of-truth governance.

The signal that this is a separate product category, not a feature: Mem0 raised $24 million across seed and Series A. Letta (formerly MemGPT) raised $10M from Felicis. Zep exists as an independent company with a temporal knowledge graph engine. Mem0’s *State of AI Agent Memory 2026* report maps 21 frameworks across three hosting models with measurable benchmark gaps between them. On LongMemEval, Zep scores 15 points higher than Mem0 on temporal queries, which tells you these aren’t interchangeable tools that happen to serve the same market.

This is the component that platform teams underestimate hardest. Memory sounds like a database problem. It isn’t.

### Governance

The assumption is that governance is RBAC plus audit logging. Your agents are services. Services get role-based access controls. You log the tool calls. Compliance is happy.

Agent governance is something different. It spans action authorization, not just data authorization. It requires decision-chain auditability, where you can reconstruct why the agent did what it did, not just what it did. It needs behavioral drift detection, tiered autonomy, and compliance mapped to agent actions rather than data accesses.

Grant Thornton’s *2026 AI Impact Survey* of 950 business executives found that 78% lack strong confidence they could pass an independent AI governance audit within 90 days. Meanwhile, enterprises are moving to increase agent autonomy faster than their governance frameworks can keep up. Traditional AI governance wasn’t designed for action-level authorization, which is where most agent-specific risk accumulates.

And there’s a hard deadline attached to this. The EU AI Act becomes fully enforceable for high-risk systems in August 2026. Credit scoring, hiring decisions, healthcare support, and critical infrastructure all fall in scope. If your internal platform doesn’t handle conformity assessments, human oversight mechanisms, complete audit trails, and ongoing monitoring, that’s not a v2 feature. That’s a legal exposure.

OWASP now documents “excessive agency” as a top vulnerability class for LLM applications. Cornell researchers have demonstrated indirect prompt injection attacks that manipulate agents through content they ingest. These are agent-specific attack surfaces, and traditional security tooling doesn’t see them.

RBAC was designed for humans with predictable intent. Agents don’t have predictable intent.

### Eval

The assumption is that evaluation means writing test cases and measuring accuracy. You built software before. You know how to test things.

Agent evaluation is qualitatively different from traditional software testing or even LLM evaluation, McKinsey’s QuantumBlack team noted: For LLMs, you evaluate the response to a prompt. For a single agent, you evaluate the full trajectory, including tool calls, state transitions, and intermediate decisions. For multi-agent systems, you evaluate system dynamics, including coordination patterns and collective invariants.

This matters because agent behavior is nondeterministic by design. The same input produces different valid execution paths. “Did the agent succeed?” is no longer a yes-or-no question, because the agent might reach the right answer through a trajectory you didn’t anticipate, or reach the wrong answer through a trajectory that looks reasonable until the last step.

The tooling ecosystem reflects this. Google Vertex AI has standardized `trajectory_exact_match`, `trajectory_precision`, and `trajectory_recall` as production metrics. These didn’t exist 18 months ago. LangSmith, Braintrust, Arize, Galileo, Maxim, and others are building full evaluation platforms around trajectory-based analysis, LLM-as-judge scoring with statistical validation, and regression testing against production failures.

Here’s the signal that the category is real: LangChain’s *2026 State of AI Agents* report found that 57% of organizations now have agents in production, and 32% cite quality as the top deployment barrier. Gartner projects that 60% of software engineering teams will adopt AI evaluation and observability platforms by 2028, up from 18% in 2025. When a category jumps from 18% to 60% adoption in three years, that’s not a “we can build this in a sprint” situation.

You can’t tell whether your evaluation is working without another evaluation. Judge drift, calibration against human experts, internal consistency across independent runs. . .your eval system needs its own eval system, which is exactly the kind of recursion that eats platform teams alive.

### Orchestration

The orchestration layer hasn’t converged. LangGraph uses directed graphs with conditional edges. CrewAI uses role-based crews. OpenAI’s Agents SDK uses explicit handoffs. AutoGen uses conversational GroupChat. Google ADK uses hierarchical agent trees. Claude’s Agent SDK uses tool-use chains with subagents. Microsoft’s Agent Framework is its own thing. Each represents a different bet on state management, communication pattern, and coordination model. None of them are interchangeable. Migration between them isn’t a config change—it’s rewriting most of your agent logic.

Underneath them, the protocol layer is still being invented. The Model Context Protocol is becoming the standard for tool integration, and agent-to-agent (A2A) protocols are emerging for cross-framework coordination. Both are moving targets, and building on a moving protocol is a cost that internal platform teams rarely price in.

If you built your own orchestration layer in 2024, you’re rewriting it in 2026. The teams that picked a framework spent those two years shipping.

## The honest case for building

I want to engage the strongest version of the build argument, because there are real reasons to build, and pretending otherwise makes this piece less useful than it should be.

Proprietary data genuinely is a durable competitive moat. Mastercard built a foundation model on its transaction network. Plaid built one on its financial institution coverage. As Morgan Stanley’s analysis from last year made clear, decades of verified historical data with consistent identifiers is both technically challenging and prohibitively expensive for outside players to recreate. If your organization has data like that, you should absolutely build on it.

Regulated industries have legitimate reasons to want control over the full stack. Off-the-shelf AI tools don’t always cleanly map to frameworks like HIPAA, GxP, 21 CFR Part 11, SOX, FFIEC, and PCI DSS, and the cost of a failed audit is measured in business units shut down, not in sprints.

Vendor lock-in at the AI layer is subtler and more dangerous than in traditional software. If your agentic workflows are built on a vendor’s proprietary orchestration layer, switching costs compound rapidly across memory, eval, and integrations simultaneously.

But here’s the distinction that matters: Those are arguments for building agents on top of platform components, not arguments for building the platform components themselves. You can own the data, the domain logic, the evaluation criteria, the governance policies, and the specific behaviors your business needs without owning the memory layer, the orchestration engine, or the trace collection infrastructure underneath them.

Build the things that are specific to your business. Buy the things that are specific to the technology category. That’s the heuristic.

**Five questions before you commit**

If you’re the platform engineer being pulled into this decision, here are the questions worth asking before anyone signs up for the scope.

**Are you building an agent platform or a workflow system?** They’re not the same scope, and conflating them is where most of the cost overruns originate. A workflow system is a reasonable thing to build. An agent platform is four product categories you haven’t staffed for.

**Can you articulate what “done” looks like for each of the four components?** Memory, governance, eval, orchestration. In under three sentences each. If you can’t, you don’t have requirements. You have a vibe. And vibes don’t ship.

**What happens to your platform when you need to swap the underlying model?** Menlo’s December 2025 data shows Anthropic went from 12% of enterprise LLM spend in 2023 to 40% in 2025, while OpenAI fell from 50% to 27%. Enterprises didn’t plan those switches. The capability gaps forced them. If your internal platform hardcoded assumptions about context windows, tool-calling formats, or reasoning styles from one vendor, swapping models isn’t an API key change. It’s simultaneous rewrites across memory, eval, and orchestration.

**What happens when the techniques themselves change?** Eighteen months ago the default pattern was RAG with flat vector retrieval. Now it’s just-in-time context strategies, agent-managed memory tiers, and trajectory-based evaluation. Anthropic’s own follow-up to “Building Effective Agents” explicitly acknowledges the field has moved since they wrote the original. If your platform baked in the 2024 patterns, the 2026 patterns are a refactor, not a config change. Vendor platforms absorb those shifts as releases. Internal platforms absorb them as sprints.

**What happens when the platform team leaves?** This is the tale as old as COBOL, custom ESBs in 2008, or hand-rolled container orchestration in 2015. A small team builds something clever, it works, they move on, and five years later you’re paying premium rates to contractors who can still read the code. Agent platforms are a particularly bad candidate for this pattern because the talent pool is both small and mobile. Here’s the uncomfortable version of the question: Who on your team, today, could rebuild the memory layer if the person who wrote it left tomorrow?

## What this looks like in 2 years

Gartner’s prediction that over 40% of agentic AI projects will be canceled by 2027 isn’t really about the AI. It’s about projects that got scoped before anyone understood the shape of the work. Most of the canceled projects will be internal builds, because internal builds are where the scope estimation error accumulates. Deloitte’s data on two- to four-year AI ROI horizons is the warning shot. If your timeline to value is already long, every month you spend rebuilding a component that exists as a product is a month you don’t have.

The teams that built their platforms around OpenAI in 2023 weren’t wrong. They made a reasonable bet on the market leader at the time. But they spent 2025 porting to a landscape where Anthropic had tripled share and Google had gone from 7% to 21%. The teams that picked model-agnostic platforms spent 2025 shipping. The only durable bet in this space is the one that assumes the bet will change.

The best platform engineering decision you can make this quarter might be to not build the platform.

## Sources

### Primary sources

- Menlo Ventures, *2025: The State of Generative AI in the Enterprise*, December 2025,
 https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/.
- Anthropic, “Building Effective Agents,” December 2024, 
 https://www.anthropic.com/research/building-effective-agents.
- Anthropic, “Effective Context Engineering for AI Agents,” 2025, 
 https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents.
- European Commission, AI Act Regulatory Framework (Regulation EU 2024/1689), 
 https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai.
- Google Cloud, “Evaluate Gen AI Agents,” Vertex AI Documentation, 
 https://cloud.google.com/vertex-ai/generative-ai/docs/models/evaluation-agents.
- McKinsey QuantumBlack, “Evaluations for the Agentic World,” 
 https://medium.com/quantumblack/evaluations-for-the-agentic-world-c3c150f0dd5a.
- LangChain, *State of Agent Engineering 2026*,
 https://www.langchain.com/state-of-agent-engineering.
- Gartner, “Gartner Predicts Over 40% of Agentic AI Projects Will Be Canceled by End of 2027,” June 2025, https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027.
- Grant Thornton, *2026 AI Impact Survey*, April 2026,
 https://www.grantthornton.com/services/advisory-services/artificial-intelligence/2026-ai-impact-survey.

### Secondary Sources

- Mem0, “Mem0 Raises $24M to Build the Memory Layer for AI,” October 2025, 
 https://mem0.ai/series-a.
- Felicis, “Felicis’s Seed in Letta,” September 2024, 
 https://www.felicis.com/blog/letta.
- Vectorize.io, “Mem0 vs Zep,” Benchmark Comparison, 
 https://vectorize.io/articles/mem0-vs-zep.
- Rasmussen et al., “Zep: A Temporal Knowledge Graph Architecture for Agent Memory,” arXiv 2501.13956, 
 https://arxiv.org/abs/2501.13956.
- OWASP, “LLM08:2025 Excessive Agency,” OWASP Top 10 for LLM Applications, 
 https://genai.owasp.org/llmrisk/llm08-excessive-agency/.
- Greshake et al., “Not What You’ve Signed Up For: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection,” arXiv 2302.12173, February 2023,
 https://arxiv.org/abs/2302.12173.
- Model Context Protocol, Official Specification, 
 https://modelcontextprotocol.io.
- PYMNTS, “FinTechs Race to Build Foundation Models on Proprietary Data,” 2026,
 https://www.pymnts.com/artificial-intelligence-2/2026/fintechs-race-to-build-foundation-models-on-proprietary-data/.
- Deloitte, “State of Generative AI in the Enterprise,” Quarterly Reports, 
 https://www.deloitte.com/us/en/insights/topics/digital-transformation/state-of-generative-ai-in-enterprise.html.
