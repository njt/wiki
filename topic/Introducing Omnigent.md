# Introducing Omnigent

Databricks open-sourced a *meta-harness* — a layer *above* individual coding agents that makes them interoperate. Rather than building Yet Another Agent Framework, Omnigent wraps existing harnesses (Claude Code, Codex, Pi) in a uniform API so they can be composed, sandboxed with contextual policies, and shared via URL. Apache 2.0, alpha.

## Key Quotes

> "The frontier of agent engineering is moving up a level."

This is the thesis in eight words. Not "build a better agent" — build a better layer *above* agents. It's the same move that took infrastructure from hand-rolled servers to Kubernetes: stop optimizing individual units, start orchestrating them.

> "Each harness is its own silo, with its own context, its own controls."

A sentence that will resonate with anyone who's had Claude Code open in one terminal and Codex in another, copying text between them. The tool-switching tax is real.

> "Messages and files in, text streams and tool calls out."

Omnigent's bet: the user-facing interface of every coding agent is already standardized whether anyone designed it that way. The input/output contract is universal enough to build a common wrapper around.

> "A policy can say: 'After downloading an npm package, require human approval before pushing to git.'"

Not "allow/deny git" — *contextual* policy that chains events. The policy engine knows what the agent just did and gates the next action accordingly. This is a level up from prompt-level guardrails.

## Key Themes

- **#meta-harness** — Abstraction above agents, not beside them. Omnigent doesn't compete with Claude Code; it wraps it.
- **#agent-composition** — Combine agents across models and harnesses with minimal code changes. The three pillars: Composition, Control, Collaboration.
- **#contextual-policy** — Stateful guardrails that chain events, not static allow/deny lists. Cost budgets that pause and ask.
- **#agent-sandboxing** — OS-level isolation that can intercept/transform network requests. A complement to things like [[cco]].
- **#agent-collaboration** — Share live agent sessions via URL with real-time commenting and steering. Agents as multiplayer documents.
- **#open-source-agent** — Apache 2.0, from Databricks. Matei Zaharia (Spark creator) is a co-author.

## Critical Analysis

**The meta-harness insight is correct.** The industry has been building better silos — better prompts, better models, better harnesses — and completely ignoring the inter-harness problem. Omnigent names a real gap: nobody's working on making agents *compose* across toolchains.

**But it's absurdly early.** The meta-harness pitch assumes a world where you run multiple agent harnesses simultaneously. Most teams are still figuring out how to run *one* effectively. The audience for this is the people who hit the ceiling of a single harness — a real but small population in mid-2026.

**The Kubernetes analogy cuts both ways.** K8s solved a real problem but added immense complexity. A meta-harness that's harder to configure than the agents it wraps would be a net loss. The YAML-based authoring and policy system needs to be dead simple or it'll be dead.

**The sandbox story is the sleeper.** Omnigent's contextual policy engine ("after X, require approval for Y") is more interesting than the composition layer. Most agent sandboxes today are binary — in or out. Stateful, event-chaining policies are genuinely new. This part of the architecture could outlive the meta-harness wrapper.

**Open source from Databricks is a flex.** Zaharia's involvement signals this isn't a side project. Apache 2.0 means it's absorbable — if the ideas are good, they'll show up in other harnesses within months. The MCP integration on the roadmap would make it even more portable.

**The collaboration feature is either transformative or a toy.** Sharing agent sessions via URL implies a world where agents are social objects — reviewed, steered, and debated by teams. That's either the future of agent UX or a feature nobody asked for. The answer depends on whether agents become reliable enough that *reviewing their process* is higher-leverage than *running your own instance*.

## Related

- [[Agent Orchestration]] — hub page; Omnigent is orchestration infrastructure
- [[Harness Engineering]] — the harness matters more than the model; Omnigent is harness engineering one level up
- [[Components of a Coding Agent]] — Omnigent wraps the harness component
- [[Security and Sandboxing]] — contextual policies as a new sandboxing primitive
- [[Agent-Native Architectures (Every)]] — composability as a first principle
- [[Loop Engineering]] — designing systems that prompt agents; Omnigent is loop engineering infrastructure
- [[The Agentic Product Standard v2.0]] — composition patterns
- [[Apache Burr]] — agent state machines; Omnigent's policies could be modeled this way
- [[cco]] — OS sandboxing for coding agents; Omnigent does it in-architecture
- [[Traycer]] — Also a meta-harness wrapping 17+ coding agents, but as a full Electron desktop application with real-time Yjs CRDT collaboration, agent-to-agent debate, and a versioned RPC protocol that enables the open-source client and closed-source host to ship independently
- [[bb — The Agent Orchestrator as Normalizer]] — Convergent design at a lower layer: bb wraps harnesses by speaking their native protocols (JSON-RPC, ACP, SDK) through per-provider adapters rather than wrapping at the user-facing I/O layer. Where Omnigent bets on abstraction ("messages in, streams out"), bb bets on adaptation — each harness keeps its full capabilities but produces a shared event model on the way out. The trade-off is fidelity vs. adapter engineering cost

---
*Source: Databricks blog, 2026-06-13. Authors: Matei Zaharia, Kasey Uhlenhuth, Corey Zumar. Last updated: 2026-08-08.*

## Updates

- **2026-08-08:** [[Managing AI Coding Costs at Scale]] (from the same Databricks team) frames Omnigent as cost infrastructure: the meta-harness preserves model independence, which is the #1 cost lever — being able to switch spend to cheaper models without forcing developers to switch tools. Paired with Unity AI Gateway for central cost observability and model menu management, it's the toolchain half of their dual-mandate (broad access + predictable costs) architecture.
