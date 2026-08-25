# n8n

A visual workflow automation platform for technical teams — Zapier/Make.com for people who also want to write code. 400+ integrations, native LangChain-based AI agent support, self-hostable under a fair-code license. 188k GitHub stars, making it one of the most popular automation tools on the planet. TypeScript, runs via Docker or `npx n8n`.

---

## Key Quotes

> "The flexibility of code with the speed of no-code."

The positioning in one line: it's not no-code *or* code, it's a visual builder where any node can drop into JavaScript/Python when the GUI runs out of expressiveness.

> Build AI agent workflows using LangChain with custom data and models.

The AI story is bolted on via LangChain integration — agents as workflow nodes, not agents as the orchestrator. This is a specific architectural choice.

## Key Themes

#tool #pattern #concept

- **Visual workflow automation** — The drag-and-drop integration builder pattern that Zapier popularized, but with escape hatches into real code. The 400+ integrations are the moat.
- **Fair-code licensing** — Not open source (not OSI-approved), but source-available and self-hostable. A deliberate middle ground between proprietary SaaS and open source that lets n8n capture enterprise revenue without losing the self-host community. This is the licensing model that [[AI Killing B2B SaaS]] warns is under threat from vibe-coded alternatives.
- **AI as workflow nodes** — n8n's AI integration treats LLM agents as nodes in a workflow graph, not as the orchestration layer itself. This is the inverse of the pattern in [[Agent Orchestration]] where agents *are* the orchestrator. n8n says: your workflow is the orchestrator, and AI is one tool it can call.
- **Self-hosting as competitive advantage** — Air-gapped deployments, SSO, data sovereignty. The same concerns that drive [[Security and Sandboxing]] for agent systems apply to workflow automation that touches production APIs and credentials. The self-hosting + OIDC/SSO + "no subscription" calculus that sells n8n also sells [[BookOrbit]], which bundles the identical pitch into a personal reading library.
- **Integration breadth as moat** — 400+ integrations and 900+ templates. This is the same play described in [[Building 200+ Integrations with OpenCode]] — integration work is grunt work, but accumulating 400 of them creates a defensible position that's expensive to replicate (though Nango showed agents can close the gap fast).

## Critical Analysis

**What's strong:** n8n occupies an important middle ground. Pure no-code tools (Zapier, Make) frustrate developers when they hit edge cases. Pure code solutions (writing integration scripts) are slow to build and maintain. n8n's "visual first, code when needed" approach is honest about the fact that most workflow steps are simple glue, but some require real logic. The self-hosting option is a genuine differentiator — if your workflows touch sensitive data or internal APIs, running Zapier through a third party is a non-starter for many organizations.

**What's weak:** The "AI-native" branding is marketing-forward. LangChain integration as workflow nodes is useful but doesn't make n8n an AI platform — it makes it a workflow platform that can call LLMs. The fair-code license is pragmatic for n8n's business but creates friction for contributors who expect open-source norms. And 188k stars notwithstanding, the ecosystem is fragile in the way all integration platforms are: every API change in every connected service is a potential breakage, and maintaining 400+ connectors is a treadmill.

**Why it matters for the wiki's themes:** n8n is relevant as a counterpoint to the agent-centric orchestration patterns that dominate this wiki. The wiki's [[Agent Orchestration]] page documents tools where agents coordinate other agents (planner/worker/judge). n8n represents the older, proven pattern: deterministic workflow graphs with human-defined control flow, now adding AI as a capability rather than a controller. The tension between "agent as orchestrator" and "workflow as orchestrator with AI nodes" is one of the central architectural decisions teams face right now. Tools like [[weft]] and [[ralph-ban]] sit in between — kanban boards where humans define the workflow but agents execute tasks.

The fair-code licensing also connects to the [[AI Killing B2B SaaS]] thesis. If agents can generate 200 integrations in 15 minutes for $20, what happens to n8n's 400-integration moat? The answer is probably "it holds for now" — n8n's integrations are maintained and tested, while agent-generated ones are fire-and-forget — but the gap is closing.

---

*Sources: [[summary/n8n]]*
*Last updated: 2026-05-14*
