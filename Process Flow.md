# Process Flow

Process Flow is a SaaS service orchestration platform built on a single sharp idea: **workflows shouldn't need a central definition**. Instead of a DAG or state machine declared upfront, each stage is an HTTP endpoint that returns JSON naming the next stage, when to run it, and what state to pass along. The workflow *emerges* from the chain of stages. It's choreography-as-a-service.

The platform handles scheduling, state persistence, retries, and visibility. You get a web dashboard to inspect any stage, pause/reschedule/cancel, and branch from any point. Free tier gives 500 stage executions/month; paid tiers scale to 50,000/month at £500.

## Key quotes

> "No restrictive contracts; each stage defines the following stage URL, execution time, and state information."

This is the core architectural bet. Rather than a workflow engine that must know the full graph, each service just knows its own successor. This makes workflows composable at the service level — you don't need to update a central definition when you add a step.

> "Build dynamic workflows with a simple JSON exchange. Each stage receives workflow state, then returns updated state, the next service location, and when the next stage should run."

The protocol is intentionally minimal. A stage is just an HTTP endpoint that accepts and returns JSON. No SDK required, no state machine DSL to learn. This is the [[Smart Models Dumb Pipes]] philosophy applied to workflow: the stages are smart, the orchestration layer is dumb.

> "Jump into any workflow and examine any stage. Review full state and execution history. Pause, reschedule, cancel or edit stages from the web interface."

Observability isn't bolted on — it's the product. Every stage run is recorded with its full state, making debugging workflows trivial. This is what Temporal and Cadence promise but with a fraction of the complexity.

## Key themes

- #tool — SaaS platform for service orchestration
- #pattern — Choreography over orchestration: stages designate successors, no central DAG
- #pattern — HTTP as workflow protocol: no SDK, no DSL, just JSON over HTTP
- #concept — Emergent workflows: the workflow is the trace of stage executions, not a pre-declared graph

## Critical analysis

**The good**: The choreography pattern is genuinely underrated. Central workflow engines (Temporal, Camunda, Step Functions) impose a coordination tax: you must declare the full graph upfront, which means every workflow change requires updating the central definition. Process Flow's approach — each stage just knows its successor — means workflows compose naturally. Adding a notification step between "booking_confirmed" and "send_reminder" doesn't require touching a central DAG; you just change what URL `booking_confirmed` points to.

**The trade-off**: The flip side of no central DAG is no central visibility into *possible* paths. You can see what *did* happen (full execution history), but you can't statically analyze what *might* happen. For compliance-heavy workflows (SOC2, HIPAA), that's a real gap — auditors want to see the possible state space, not just the traces. This isn't a bug; it's the price of the architectural choice.

**The pricing tells a story**: £500/month for 50,000 executions is ~£0.01 per stage execution. That's cheap enough to run customer journeys (booking → reminder → follow-up → survey) at scale, but expensive enough that you'd think twice before putting *every* microservice call through it. The pricing suggests they're targeting business process orchestration (customer workflows, onboarding, approvals) rather than high-frequency service mesh traffic.

**The competitive landscape**: This sits between [[n8n]] (visual workflow builder, 400+ integrations, self-hostable) and Temporal (durable execution with strong guarantees). Process Flow is simpler than both — no visual builder, no SDK, just HTTP. That's either a feature (zero lock-in, any language) or a limitation (no type safety, no compile-time checks), depending on your team.

**Unanswered questions**: Who built this? No author, no team page, no GitHub org. The pricing is in GBP, suggesting UK. The SvelteKit + Material Dashboard stack is polished but not distinctive. The domain was not indexed by search engines as of June 2026, which at this pricing tier suggests an indie project or early-stage startup rather than a venture-backed company.

## Related pages

- [[Agent Orchestration]] — Multi-agent coordination patterns; Process Flow's choreography model applies directly
- [[All Your Agents Are Going Async]] — HTTP may be the wrong transport for agents that outlive connections, but Process Flow's scheduled-execution model sidesteps this by never holding connections open
- [[n8n]] — Visual workflow automation with 400+ integrations; Process Flow is the anti-n8n: no visual builder, no integrations, just HTTP
- [[Smart Models Dumb Pipes]] — The same architectural philosophy: the endpoints are smart, the orchestration layer is dumb
- [[Swamp Club]] — Agent-first workflow framework with DAG execution; Process Flow is choreography where Swamp Club is orchestration
- [[Event-Driven vs Polling Architectures]] — Process Flow's scheduled execution model is polling-by-design, avoiding the webhook reliability problems that guide describes

---
*Source: [processflow.tech](https://processflow.tech/), fetched 2026-06-09*
