# Behavior-Centric AI Design

Adam Jacob's manifesto against the default architecture of the AI era: stop letting the LLM drive control flow, and instead compile intent into reusable, typed, composable "behaviors" that call frontier intelligence only where a job genuinely needs it. The agent-centric design, he argues, is why agentic AI keeps failing to show ROI.

---

Jacob traces agent-centric design to ChatGPT and the mid-2023 function-calling API: the LLM sits at the heart of the program and decides, probabilistically, what happens next. Everything since — RAG, skills, MCP, gateways, model routers, sub-agents, memory schemes — has been an add-on that leaves the core decision in place. The consequences are structural, not incidental:

> The more an LLM is responsible for the control flow, the more the correctness of the resulting system relies on it making the right decision at every point - which cuts directly against the nature of the LLM.

His trip-planner example is well chosen precisely because it's mundane: the user expects a hotel and a flight every time, but a planning agent might decide not to call the hotel agent. The standard fix — validator agents checking other agents — compounds the problem, since each agent spends tokens on its own context-gathering and reasoning. He also lands a sharp social observation: these systems are tolerable to their designers, who know why they fail and just retry, but at organizational scale the papercuts become unacceptable and the deployment/observability/security footprint balloons. That's the mechanism behind the ROI death spiral he cites Gartner (40%+ of agentic projects cancelled by 2027) and McKinsey (37% of enterprises showing financial impact) for.

The alternative: behaviors with typed, validated inputs and outputs, versioned structured data, isolated credentials — enforced by the platform at runtime, not by convention. A workflow composes them; at most a few scoped LLM calls sit inside behaviors, never in the orchestration path. The agent-facing surface reduces to one tool call. His own benchmark: 14x fewer tokens, 6x cheaper per trip plan versus a LangChain agent. Then the sales turn: Swamp is the proof, and "instead of using frontier intelligence at runtime to decide what to do, we use it at build time to encode our intent into behaviors we can rely on."

## Themes

- #concept — control flow is the scarce resource; probabilistic control flow is the anti-pattern
- #pattern — behaviors as typed, runtime-validated, composable building blocks with a shared corpus
- #tool — Swamp as the reference implementation of the pattern
- #comparison — agent-centric vs behavior-centric as the framing device

## Analysis

This is the strongest single-artifact articulation of a thesis that is quietly becoming consensus: the LLM should be a component, not the program. Jacob's contribution is naming the failure mode precisely — it's not that models are dumb, it's that putting them in charge of orchestration makes correctness a probabilistic property of every decision point, and robustness then gets bought back with tokens. The designer-forgiveness point is genuinely insightful and explains why demos impress and deployments disappoint.

But read it as what it is: a vendor essay whose benchmarks are the vendor's own. The 14x comparison pits an "unoptimized" LangChain agent against his own framework, and the production claims (8x tokens on code review, "hundreds of deploys a week") come with no methodology. The strawman risk is real too — mature agent-centric practice already wraps agents in deterministic harnesses, typed tool schemas, and validation gates; "behavior-centric" is in danger of being a rebrand of "good engineering around the model." The honest version of the claim is about degree: how much of the control flow you take away from the model. On that, his answer (all of it, at build time) is more maximalist than most practitioners, and the truth is probably that some workflows want a model in the loop for branching and some don't.

Still, the core move — enforce contracts at runtime, spend tokens on extraction not orchestration, treat behaviors as an organizational corpus — is right, and the essay states it better than most.

## Relations

- [[Reducing Token Spend with Deterministic Workflows]] — Jacob's own earlier field report (8x token cut from the same move); this essay is that anecdote grown into a general theory and a platform pitch, strengthening it with a named pattern and a mechanism (runtime-enforced contracts).
- [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]] — Codemanship's tiered tool-choice rule says the same thing about allocation but stops short of Jacob's maximalism; this essay complicates it by arguing the deterministic tier should be a first-class, platform-enforced product, not just a default preference.
- [[Swamp Club]] — the tool page for the platform this essay exists to promote; this note supplies the design rationale the tool summary lacks, while also being the source to read skeptically for the benchmarks.
- [[Stop Asking the Reasoning Model to Decide Everything]] — Jeremy Daly's routing taxonomy reaches the same destination (bounded judgments for models, deterministic paths for code) from the cost side; Jacob's version is the organizational-scale argument for the same routing decision.

---
*Sources: [[raw/behavior]], [[summary/behavior]]*
*Last updated: 2026-10-18*
