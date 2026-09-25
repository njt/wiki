---
url: https://www.media.mit.edu/articles/what-is-a-multi-agent-system/
date_fetched: 2026-09-25
---

# The Identity Crisis

The multi-agent systems field faces an identity crisis. As LLMs dominate AI discourse, researchers rebrand orchestrated applications as "Multi-Agent Systems." But most miss what makes systems genuinely multi-agent: **how agents interact matters more than how smart individual agents are**.

- Ayush Chopra, PhD Candidate MIT (media.mit.edu/~ayushc)

## The Agent-as-Object Problem

Most frameworks today follow an agent-centric design pattern: start with individual agents as objects, give each sophisticated capabilities (reasoning, planning, memory), then compose systems by orchestrating these agent objects through workflows. This leads to architectures that look like object-oriented programming with natural language interfaces—Action Agents, Planning Agents, Orchestrator Agents—where interaction becomes an afterthought implemented through APIs between independent components.

Current "MAS LLMs" treat multi-agent systems as collections of smart agents coordinated by orchestrators, but genuine distributed systems exhibit coordination that emerges from agent interactions—coordination strategies that couldn't be designed by any central planner and had to be discovered through distributed decision-making dynamics.

## Interaction Patterns Drive Intelligence

Real distributed coordination emerges from interaction patterns, not sophisticated individual reasoning. Supply chain disruptions cascade through global networks not because individual logistics managers are brilliant, but because of how thousands of local decisions interact across multiple scales. Financial markets exhibit coordination through how trading decisions interact via price signals and information flow. Social movements emerge from how individual actions influence others through networks. The intelligence is in the interaction patterns themselves, not in any individual decision-maker.

## The Multi-Scale Composition Challenge

When control, data, and expertise are distributed, agents coordinate across multiple scales simultaneously. Local interactions aggregate into network patterns, which influence global behavior, which feeds back to constrain local decisions. Modern distributed AI systems interact across diverse protocols—APIs, message queues, blockchain networks—each with different characteristics for latency, reliability, and privacy. The fundamental challenge: How do you compose interaction patterns across different scales, protocol types, and temporal dynamics while maintaining adaptability?

## Interaction-First Design

What if we inverted the design paradigm entirely? Instead of designing systems as collections of agent objects, what if interaction patterns were the primary abstraction? You would compose systems by defining how agents influence each other, with agents becoming participants in interaction patterns rather than independent entities. This abstraction shift enables emergent rather than orchestrated coordination, naturally handles multi-scale dynamics, and opens possibilities for genuinely adaptive distributed systems where intelligence emerges from how agents influence each other rather than from sophisticated individual components.

This abstraction shift enables capabilities impossible with agent-centric approaches. Instead of pre-programming coordination strategies, you can discover emergent coordination patterns that adapt to changing conditions. Instead of managing complex agent hierarchies, you compose interaction substeps that operate across multiple scales simultaneously. Frameworks like AgentTorch demonstrate how to make this practical—treating interaction patterns as differentiable primitives that can be composed, optimized, and deployed at scale. The result: genuinely adaptive distributed systems where intelligence emerges from how agents influence each other.

**The right abstraction for multi-agent systems isn't the agent—it's the interaction.**

**Key Takeaway**: Current MAS frameworks build "smart objects with workflows." Real distributed intelligence requires "interaction patterns that enable emergent coordination." The fundamental design shift is from agent-centric to interaction-centric abstractions.

- How to build interaction-centric MAS?: github.com/AgentTorch/AgentTorch
