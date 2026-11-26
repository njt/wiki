# Building Always-On Agents (Anthropic)

Anthropic's (William Hu) reference architecture for always-on agents: the agent loop is unchanged, but the surrounding scaffolding — event inbox, timers, memory, permissions, outbox — is what makes an agent that sleeps, wakes on events, and acts unattended feel "always on." It's the most complete first-party blueprint yet for the durable-state, event-driven agent harness.

---

## Key quotes

> An always-on agent is not actually "always on." This is just a description of the user experience.

Honest demystification: the magic is a run/session cycle plus durable state, not a persistent process. Everything that matters lives in what carries over between runs — which is why four of the five primitives are storage problems.

> The inbox should be the source of truth. When the worker is woken up, it should pull the latest events from the inbox first. That way, if the worker dies mid-run, a new one can pick up the same event and retry.

This is straight distributed-systems practice — at-least-once delivery, idempotent workers, one active worker per inbox — applied to agents without apology. The article quietly argues that building always-on agents is mostly *re-learning* queueing and durability discipline, not inventing new AI.

> As a rule of thumb, memory should only hold what the agent can't look up again. If something already lives in another system, such as an issue tracker, you should save a pointer instead.

A memory discipline that resists hoarding: instructions, user notes, past-message summaries, per-task progress files, index-only loading at run start. It makes memory a cache with an eviction philosophy rather than a transcript.

> Permissions are where always-on agents carry the most risk. A run holds the user's access and, by design, operates unattended.

The out-of-environment credential proxy is the standout pattern: the agent never holds real credentials; a sidecar injects them into outgoing requests. Combined with classifier-gated auto mode and deferred tool calls that park a mid-call session until the user's reply arrives as an inbox event, it's a concrete answer to the unattended-authority problem.

> In practice, it's better for the agent to start quiet and then earn more of the user's attention over time.

Notably identical advice to Rubric Labs' "start out shy" — an emerging consensus that notification discernment is a calibrated, eval-tuned property, not a prompt line.

## Key themes

#concept #pattern #agent-ux #distributed-systems

## Analysis

This is a product-company how-to, and its bias shows in useful ways: every pattern is named after something Anthropic already ships (Claude Code's auto memory, auto mode, hooks, the Messages API memory tool). The five primitives are deliberately boring — database tables, workers, schedulers, dispatchers — and that's the thesis: the hard problems of always-on agents are **durable state and control**, not intelligence. The cost framing (Opus 5.5 at 78% less per solved long-horizon task) reveals the actual unlock: always-on was always architecturally possible; it was economically absurd until long-horizon tasks got cheap.

What the piece underplays: it treats the inbox/worker model as settled when real multi-source deployments will hit harder questions (per-event security verification deferred into the run is a prompt-injection surface, not just a latency choice), and it barely addresses multi-user tenancy. It's also silent on what the agent should *do* — discernment, the hard part per Rubric Labs, gets one sentence about message volume.

## Relations

- [[Always-On Agents (Rubric Labs)]] — the field-formulation this article industrialises: Rubric Labs' heartbeat and shy-first discernment map onto timers and the outbox here, but Anthropic adds the durable-inbox and permission machinery Rubric Labs hand-waves.
- [[Agent Memory]] — this article's four memory types (instructions, notes, past messages, task progress) are a pragmatic slice of Jones's seven-type taxonomy, with the same pointer-not-copy discipline.
- [[Scaling Long-Running Agents]] — both argue continuity lives in the harness, not the model; this piece supplies the concrete stores (inbox, timers, memory files, outbox) that long-running-agent discussions usually gesture at.
- [[Devin Memory and Dreaming]] — the optional "dreaming" pass curating memories across sessions strengthens the case that offline memory consolidation is becoming a standard agent component.

---
*Sources: [[raw/building-always-on-agents]], [[summary/building-always-on-agents]]*
*Last updated: 2026-11-26*
