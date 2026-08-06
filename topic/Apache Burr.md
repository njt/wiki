# Apache Burr

Apache Burr (incubating) is an open-source Python framework for building AI agents and applications as **explicit state machines**. Instead of chaining LLM calls through opaque abstractions, you decorate plain Python functions as `@action`s that read and write typed state, wire them into transitions with `ApplicationBuilder`, and get persistence, observability, replay, and a debugging UI for free. Built by DAGWorks Inc. and donated to the Apache Foundation.

---

## Key Quotes

> "Pure Python, no magic."

This is the thesis and the marketing. Every other agent framework ships a DSL, a YAML config, or a labyrinth of chains and callbacks. Burr's pitch is: write Python functions, declare what state they touch, and the framework handles the rest. Whether "no magic" holds up at production scale is unproven, but the instinct is correct — agent code should look like normal code, not ritual incantation.

> "Moving from LangChain to Burr was a game-changer! Burr provides a more robust framework for designing complex behaviors."

TaskHuman's DS Architect pivoted their entire codebase. Multiple testimonials name LangChain specifically as the thing they escaped. The LangChain backlash is real, and Burr is one of its beneficiaries — positioned as the un-LangChain: explicit over implicit, decorators over chains, state over hidden prompts.

> "Their UI which makes debugging a piece of cake."

Watto.ai's founder on the Burr UI. Real-time state inspection during execution is Burr's second-best feature after state persistence. Most agent frameworks give you log lines; Burr gives you a visual state machine you can step through. This matters more than most framework authors admit — when your agent does something wrong, you need to see *which state transition* went sideways, not grep through JSON logs.

> "State management part is really helpful for creating state snapshots and build debugging, replaying and even building evaluation cases."

Provectus's architect on the persistence layer. State snapshots aren't just for debugging — they're eval data. Every run produces a replayable trace of state transitions. This turns agent testing from "did it work?" into "did it follow the right path through the state machine?".

---

## Key Themes

#tool #agent-architecture #state-machine #python

- **State machine as the organizing principle** — Burr doesn't treat state as an afterthought the way LangChain does. Actions declare `reads=[...]` and `writes=[...]` on state keys, making data flow explicit and auditable. This is the same insight as [[Swamp Club]]'s typed, versioned data, but with a developer-authored rather than agent-authored workflow.

- **Transitions are the API** — Instead of abstract "chains" or "pipelines," Burr uses explicit transition tuples: `("current_action", "next_action")`. This is finite state machine 101, but applied to LLM applications it becomes a genuine design tool. You can *see* the possible paths. Compare to LangChain where the path through chains is emergent from prompt outputs.

- **Persistence as first-class** — State serializes automatically to disk, Postgres, or custom backends. Applications resume from where they left off. This addresses the [[Context Rot]] problem directly: agent state survives session boundaries because it's structured data, not conversation history.

- **Observability as product, not afterthought** — The Burr UI shows a live state machine graph with current state highlighted, history of transitions, and state snapshots at each step. This is what [[Harness Engineering]] calls feedforward control: make the system's internal state visible so you can debug before things break.

- **Apache incubation as credibility signal** — Donating to Apache signals governance, community, and longevity. It also means the project moves at foundation pace, not startup pace. For enterprises evaluating agent frameworks, the Apache brand matters more than the feature list.

---

## Critical Analysis

**What's good:** The state machine model is the right abstraction for agent applications. Most "agent frameworks" are actually prompt pipelines with extra steps. Burr forces you to think about state transitions explicitly, which is what you should be doing anyway. The decorator API (`@action(reads=[...], writes=[...])`) is clean and Pythonic. The built-in observability UI solves a real problem — debugging agents without visibility into state transitions is guessing.

**What concerns me:** "No magic" is a promise, not a guarantee. The `ApplicationBuilder` pattern, the transition tuples, the halt conditions — this is still a framework with opinions. The question is whether its opinions are the right ones, not whether it has fewer of them than LangChain. Also, Burr is DAGWorks's product before it's Apache's project. The company built it, used it, then donated it. That's a fine model ([[Swamp Club]]'s System Initiative pedigree is similar), but it means the roadmap reflects DAGWorks's needs first.

**The LangChain comparison is revealing but limiting.** Yes, Burr is less abstraction-heavy than LangChain. But "better than LangChain" is the lowest bar in agent frameworks — LangChain's architecture is widely acknowledged as over-engineered. The real question is whether Burr's state machine model is better than: (a) [[Swamp Club]]'s agent-authored DAGs, (b) [[Slate]]'s thread-and-episode context routing, (c) [[Elysia]]'s decision tree over tool selection, or (d) just writing a Python script with good logging. The testimonials don't address this — they're all LangChain refugees, not comparative evaluations.

**The state machine model has a ceiling.** Finite state machines work beautifully for workflows with known states and transitions — customer service bots, approval pipelines, structured data extraction. They break down when the state space is genuinely open — creative tasks, research agents, anything where the agent discovers the next step rather than following a pre-designed graph. Burr's parallelism and sub-application features try to address this, but they're bolt-ons to a fundamentally deterministic model. For [[Agent Orchestration]] patterns where multiple agents coordinate dynamically, Burr's fixed transition graph becomes a constraint, not a feature.

**Who should use this:** Teams migrating off LangChain who want cleaner architecture without giving up observability. Python shops building structured AI applications (approval workflows, RAG pipelines, multi-step extraction) where the state transitions are known in advance. Teams that value replayability and audit trails — Burr's state snapshots make compliance easier than any other agent framework I've seen.

**Who should skip it:** Teams building open-ended research agents where the agent discovers its own path. TypeScript/JavaScript shops (Burr is Python-only). Teams that want a visual workflow builder (use [[n8n]]). Teams that think the entire concept of agent frameworks is overrated and you should just write Python.

**The Apache factor cuts both ways.** Incubation means the project has governance, a code of conduct, IP clearance, and a path to graduation. It also means the project can't move fast, can't have a commercial edition, and can't prioritize features that only paying customers need. For DAGWorks, Apache is a credibility play. For users, it's a trade-off between stability and velocity.

---

## Related Pages

- [[Swamp Club]] — Agent-authored DAG workflows with immutable versioned data; Burr is developer-authored state machines with replayable state snapshots. Same insight (structure over improvisation), opposite authorship model.
- [[Elysia]] — Decision tree over tool selection; shares Burr's philosophy of constraining the action space rather than dumping everything into context. Burr constrains via state transitions, Elysia constrains via tree routing.
- [[Process Flow]] — Choreography over orchestration: each stage designates its successor. Burr is the opposite: central state machine definition with explicit transitions. Same problem domain, inverted architecture.
- [[Slate]] — Thread-and-episode architecture with context routing as the core primitive. Burr's state machine is more rigid but more debuggable; Slate's routing is more flexible but harder to reason about.
- [[Semantic Kernel]] — Microsoft's enterprise agent middleware. Burr is lighter-weight, Python-only, and state-machine-first; SK is polyglot, enterprise-grade, and function-calling-first.
- [[Elements of Agentic Systems Design]] — Burr maps cleanly to the taxonomy: State persistence = Memory, @action decorator = Agency, transitions = Reasoning, sub-applications = Coordination, state snapshots = Artifacts, replay = Evaluation.
- [[Harness Engineering]] — The harness matters more than the model. Burr's harness (state machine, persistence, UI) is its product; the model is a component.
- [[celld]] — Deno's self-hosted Durable Objects runtime that uses the same state-machine pattern as Burr, but for distributed infrastructure coordination rather than AI agents. `celld-logic` is a pure, zero-dependency state machine that's replayable under deterministic simulation — the same insight, different domain.
- [[Agent Orchestration]] — Hub page for multi-agent coordination patterns. Burr's sub-application model is one approach to agent composition.
- [[n8n]] — Visual workflow automation; Burr is programmatic where n8n is visual. Different audiences, same underlying DAG concept.

---
*Source: [burr.apache.org](https://burr.apache.org/), fetched 2026-06-11*
