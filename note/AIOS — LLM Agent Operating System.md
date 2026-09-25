# AIOS — LLM Agent Operating System

AIOS (arXiv 2403.16971, Yongfeng Zhang et al., first posted March 2024) is an academic architecture paper that asks: what if agents ran on an operating system designed for them? It proposes pulling LLM-specific services — scheduling, context management, memory, storage, access control — out of agent applications and into an "AIOS kernel", with an SDK on top. Reported result: up to 2.1x faster execution serving agents built with various frameworks.

---

The framing is the interesting part, more than the numbers. Most agent infrastructure treats each agent as an application that owns its own loop, its own context, and its own tool calls. AIOS says that is the same mistake computing made before OSes: agents fighting over an LLM is agents fighting over a CPU, and ad-hoc per-agent management of context and memory is every program hand-rolling its own virtual memory. The kernel abstraction — agents as processes, the LLM as a scheduled resource — is a genuinely structural claim about where agent infrastructure should live.

It is also a claim that has aged oddly well and oddly badly. Well: everything since — serving layers, agent sandboxes, orchestration planes — has drifted toward AIOS's picture of centralized resource management for fleets of agents. Badly: the paper predates the modern coding-agent world, where the "agent" is one developer's single process and the scarce resource is context window, not throughput. The concurrent-multi-tenant story it tells is a datacenter story, not a laptop story.

One caveat on provenance: the ingested source is the arXiv abstract page, not the full paper. The architecture details (what exactly the scheduler schedules, how access control works) are not verified here — only what the abstract asserts.

## Key quotes

> "Allowing unrestricted access to LLM or tool resources can lead to inefficient or even potentially harmful resource allocation and utilization for agents."

The "harmful" is doing quiet work — this is a resource-management paper that smuggles in containment. An agent with unrestricted tool access is a [[Security and Sandboxing]] problem as much as a scheduling one.

> "The absence of proper scheduling and resource management mechanisms in current agent designs hinders concurrent processing and limits overall system efficiency."

The core thesis, stated as an OS analogy: agents are processes, the LLM is the processor, and someone has to own the scheduler. This anticipates the fleet-serving questions that [[Agent Substrate]] answers with checkpointed worker pods — AIOS names the layer, Agent Substrate builds it on Kubernetes.

> "It introduces a novel architecture for serving LLM-based agents by isolating resources and LLM-specific services from agent applications into an AIOS kernel."

Kernel isolation is the paper's real contribution claim. It splits agent *applications* from agent *infrastructure*, which is exactly the boundary later systems redraw — harness below, agent above.

> "Experimental results demonstrate that using AIOS can achieve up to 2.1x faster execution for serving agents built by various agent frameworks."

The 2.1x is a serving-throughput number from a multi-agent workload, not a single-agent quality number. Read it as evidence about the concurrency problem, not about making any one agent smarter.

## Themes

#concept — agents as OS processes; the LLM as a schedulable resource
#tool — AIOS kernel and SDK as a concrete system
#pattern — kernel/application isolation, borrowed from classical OS design

## Analysis

The opinionated take: AIOS is right about the layer and probably wrong about who builds it. The paper imagines an OS for agents emerging as a research artifact with its own kernel and SDK. What actually emerged is messier — serving engines, sandbox runtimes, orchestration planes, and harnesss each owning a slice of what AIOS wanted to centralize, with no single vendor winning the "agent OS" title. That is not a refutation; the resource-management problems it named (concurrency, context scheduling, access control) are exactly the ones being solved piecemeal.

It also undersells what turned out to be the hardest resource: context. The abstract lists "context management" as one kernel service among five, but post-2024 the context window is the scarce resource that scheduling actually fights over — see [[Agentic Context Management]]. AIOS's scheduler story is about requests; the lived problem is about tokens.

## Related pages

- [[Agent Substrate]] — AIOS names the kernel layer; Agent Substrate is a production realization of the same insight (agents as schedulable, checkpointable workloads), with a deliberately dumb scheduler instead of AIOS's rich one.
- [[Agentic Context Management]] — complicates AIOS's list of kernel services: the resource that actually needs managing is the context window, not just requests and memory.
- [[The Case Against Building Your Own Agent Platform]] — the counterweight: if AIOS's layer exists, someone has to maintain it, and most teams should not.
- [[Components of a Coding Agent]] — the single-agent view AIOS abstracts away: for one developer's agent, the OS is the harness, not a multi-tenant kernel.

---
*Sources: [[raw/2403-16971]], [[summary/2403-16971]]*
*Last updated: 2026-09-25*
