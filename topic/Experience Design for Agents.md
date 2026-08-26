# Experience Design for Agents

Kurtis Kemple argues that experience design -- not model capability -- is the single biggest factor in whether an agent gets adopted or abandoned. He proposes a four-layer responsibility model (human, agent, workflow, tool) with aligned autonomy levels, and identifies three enabling conditions: context management, authority alignment, and progressive trust.

---

## Key Quotes

> "The single biggest factor in whether an agent gets adopted or abandoned is experience design."

> "Ship with narrow, inspectable, reversible defaults. Make the authority structure visible."

> For agents' authority to remain aligned with human judgment, the agent's actions have to stay "transparent, steerable, and resilient under failure."

## Key Themes

#agent-design #ux #autonomy #trust #human-agent-interaction

The four-layer model (human owns intent/judgment, agent owns planning/outcomes, workflow owns automation, tool owns execution) is a clean taxonomy for thinking about where responsibility lives. The key insight: judgment cannot be delegated. The human always owns intent, even when the agent has significant autonomy over execution.

The "progressive authority" concept -- autonomy expands as agents demonstrate competence -- connects directly to [[Chief of Staff]]'s graduated autonomy model (three levels with measurable graduation criteria, rolling 90-day trust window). Kemple provides the theory; De Jesus provides the implementation.

[[AI UX Patterns — User Transparency]] is the concrete UI layer for this: Nanz enumerates what "make the authority structure visible" means in practice — granular permissions (read vs. send, reference vs. delete), a revocable memory ledger, up-front cost estimates, and a visual marker for when the agent is acting autonomously. It strengthens Kemple's "transparent, steerable, resilient" principle by turning it into a buildable checklist, though its assumption of a careful, consenting user is exactly what [[How We Contain Claude]]'s 93% approval rate undermines.

The "drift" failure mode (context management lapses, agent's understanding becomes stale) is the UX manifestation of what [[Elements of Agentic Systems Design]] calls the Context element. It's also the core problem [[Cord]] tries to solve with runtime task decomposition -- maintaining coherent context across complex multi-step work.

## Critical Analysis

Strong: the responsibility model is immediately useful for anyone designing agent interfaces. The emphasis on reversibility and inspectability echoes [[yolo-cage]]'s approach of deferring decisions to PR review. "Narrow, inspectable, reversible defaults" is a design principle worth internalizing.

Missing: no discussion of how to handle disagreements between agent and human (what happens when the agent's plan is better than what the human requested?). The framework assumes aligned interests, but real agent deployments surface genuine conflicts between user intent and optimal outcomes. Also no consideration of multi-agent scenarios where responsibility models need to compose.

The Signal → Decision → Response pattern from [[AI-Powered Gamification for the Web]] is a concrete, buildable instantiation of Kemple's principles at the feature level rather than the system level. Where Kemple says "make authority visible and reversible," Yatteau operationalises this as: capture one observable behaviour (signal), let AI make one narrow call (decision), and visibly change the UI so the user feels the effect (response). The AI is deliberately invisible — "the user really does not care that there's some kind of model happening in the background" — which aligns with Kemple's argument that experience design, not model capability, determines adoption.

---
*Sources: [[summary/experience-design-for-agents]]*
*Last updated: 2026-05-14*
