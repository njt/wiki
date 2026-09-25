# The Right Abstraction Is the Interaction

Ayush Chopra (MIT Media Lab) argues that today's LLM "multi-agent systems" are mostly agent-centric mislabels: smart objects orchestrated through workflows, which he compares to object-oriented programming with natural language interfaces. Genuine multi-agent intelligence, he contends, lives in interaction patterns — emergent, multi-scale, no central planner — and the field should invert its design paradigm to make the interaction, not the agent, the primary abstraction.

---

## What it says

- **The identity crisis.** LLM-era frameworks rebrand orchestrated apps as "Multi-Agent Systems" while missing the classical MAS insight: *how agents interact matters more than how smart individual agents are*.
- **Agent-as-object.** Most frameworks start with individual agents as objects, give them reasoning/planning/memory, then compose via orchestrators. Interaction becomes "an afterthought implemented through APIs between independent components."
- **Interaction patterns drive intelligence.** Supply chains, markets, and social movements coordinate not because individual decision-makers are brilliant but because of how thousands of local decisions interact. "The intelligence is in the interaction patterns themselves."
- **Multi-scale composition.** Real distributed systems coordinate across scales and protocols (APIs, message queues, blockchains) simultaneously — local interactions aggregate into network patterns that feed back to constrain local decisions.
- **Interaction-first design.** Compose systems by defining how agents influence each other; agents become participants in patterns rather than independent entities. AgentTorch (his framework) treats interaction patterns as differentiable primitives that can be composed and optimized.

---

## Key quotes

> "architectures that look like object-oriented programming with natural language interfaces—Action Agents, Planning Agents, Orchestrator Agents—where interaction becomes an afterthought"

A genuinely cutting diagnosis. It names the exact shape most 2025-26 agent frameworks have: classes with LLM brains and a router on top. The OOP analogy is the strongest single line in the piece because it explains *why* the designs feel brittle — they inherit object thinking wholesale.

> "genuine distributed systems exhibit coordination that emerges from agent interactions—coordination strategies that couldn't be designed by any central planner and had to be discovered through distributed decision-making dynamics"

This is the classical-MAS pedigree talking (Chopra comes from the pre-LLM distributed AI tradition), and it's the part practitioners orchestrating Claude/Codex fleets will resist: almost every production multi-agent setup today *is* a central planner. His claim is that this is a phase, not an endpoint.

> "The right abstraction for multi-agent systems isn't the agent—it's the interaction."

The thesis in one sentence. It's a design-abstraction argument, not an implementation recipe — which is both its strength (portable) and its weakness (nothing here says what to do on Monday morning short of "try AgentTorch").

---

## Themes

#concept #pattern #comparison

## Analysis

The essay is short, manifesto-shaped, and pointedly dismissive of the current mainstream — which makes it useful precisely because the mainstream is what this wiki mostly documents. Nearly every orchestration source here ([[Multi-Agent AI Systems Are Organizations]], [[Swarm Skill]], [[Agent Swarm Model Economics]]) describes agent-centric systems: planner/worker trees, orchestrators, queues, boards. Chopra's claim is that all of that is the wrong abstraction, and the interesting question is whether he's right or merely nostalgic for pre-LLM MAS.

The strongest steelman of agent-centric design is that LLM agents are unlike classical MAS agents: their value *is* individual reasoning quality, and interactions between prompt-driven components are hard to make differentiable or even stable. Chopra's counterweight — emergent coordination discovered through distributed dynamics — has an empirical basis in markets and supply chains, but "emergence" is also the word engineers reach for when they want to skip specifying behavior. The honest reading: he's right that interaction structure is under-theorized in current frameworks (most "protocols" are just message passing), but the leap from that to differentiable interaction primitives is asserted via AgentTorch rather than demonstrated here.

Where it lands for practitioners: as a lens, not a blueprint. When you build an orchestrator and it fails, the failure is usually in the interaction pattern (message formats, handoffs, feedback loops) — not in how smart each agent is. That diagnosis is this essay's real gift, even if the prescribed cure is research-grade.

## Related pages

This source sharpens the [[Agent Orchestration]] topic's framing: nearly everything filed there is agent-centric (orchestrators, hierarchies, queues), and Chopra names that as the field's core mistake — a standing challenge to every page in that section.

It complicates [[Multi-Agent AI Systems Are Organizations]]: that source maps multi-agent systems onto organizations with central task division and allocation; Chopra argues the genuinely intelligent part is exactly what no central planner designed, so the org analogy captures the orchestration layer while missing the emergent one.

It nuances [[Swarm Skill]] and the swarm-orchestration tradition: those 14 hard rules are hand-engineered coordination patterns for agent-centric systems — which, in Chopra's terms, are precisely the "smart objects with workflows" he says must give way to emergent interaction patterns.

It also converses with [[Deterministic Spine, Agentic Leaves]]: both locate multi-agent failure in the connective tissue rather than the individual agents, though one prescribes deterministic control and the other emergent coordination — opposite remedies for the same diagnosis.

---
*Sources: [[raw/what-is-a-multi-agent-system]], [[summary/what-is-a-multi-agent-system]]*
*Last updated: 2026-09-25*
