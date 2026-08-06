# Event-Driven vs Polling Architectures

Michel Tricot's definitive piece on trigger architecture for agents — the load-bearing infrastructure most teams treat as an afterthought. Published in Agent Blueprint, May 2026.

The core argument: framing trigger choice as webhooks-vs-polling is the root cause of brittle production systems. Tricot identifies **four distinct mechanisms** (webhooks, log-based CDC, message-bus subscriptions, scheduled polling) with fundamentally different delivery contracts, then walks through the real-world guarantees of Stripe, Shopify, HubSpot, GitHub, and Salesforce to show that "real-time webhooks" are mostly marketing. The conclusion is architectural, not ideological: **the source picks the pattern**, and the right answer for a multi-source agent system is different mechanisms per source.

## Key Quotes

> "A webhook tells me something changed. It does not tell me the truth about what changed, or in what order."

This is the thesis in one sentence. Webhooks are signals, not facts. Every major SaaS provider — Stripe, Shopify, HubSpot — documents that events may arrive out of order, may be duplicated, and may carry stale payloads on retry. Tricot's forensic walk through each vendor's actual delivery contract is the most valuable part of the piece. The Shopify detail is particularly damning: a products/update webhook can arrive before products/create for the same product, and retries carry the original trigger-time payload, not current state.

> "Picking an at-least-once trigger without a deduplication store at the write boundary is how payment duplicates, duplicate CRM contacts, and duplicate outbound emails get into production."

This is where the piece moves from diagnosis to prescription. The idempotency discussion is the strongest section — Tricot identifies the **LLM non-determinism problem**: if you hash tool parameters into the idempotency key, a retry that generates slightly different parameters produces a different key. The solution is structural keys — `(agent_run_id, step_id, tool_name, call_index)` — assigned before inference runs and invariant across retries. This is a genuinely non-obvious insight specific to agent systems.

> "events will be missed. Not might. Will."

The working assumption that makes the hybrid pattern necessary. Tricot's case for webhook-plus-reconciliation as the default (not webhook alone) is backed by Stripe and Merge's own guidance. The hybrid isn't magic — it doubles operational cost and doesn't help with sources that emit no events at all — but it's the least-bad default for HTTP-only sources.

> "The right trigger architecture isn't the one I prefer. It's the best one the source will let me have."

The anti-ideology conclusion. Tricot's per-source decision framework (four questions: latency tolerance, source capabilities, cost of miss vs. duplicate, where the trigger lands) replaces the webhook-or-poll binary with something actually useful. The Salesforce deep-dive — three overlapping event mechanisms with different semantics inside a single platform — is the proof that "event-driven" isn't one thing even within one vendor.

## Key Themes

#trigger-architecture #idempotency #durable-execution #webhooks #CDC #message-bus #polling #at-least-once-delivery #agent-infrastructure #production-engineering #reconciliation #structural-idempotency

## Critical Analysis

**What makes this piece exceptional:** Tricot doesn't write generic architectural advice. He names specific vendors, specific retry windows, specific rate limits, and specific failure modes. The Shopify detail about stale retry payloads, the GitHub observation that conditional GET 304s don't count against rate limits, the Salesforce warning about replay IDs getting nuked during instance migrations — these are details you only get from having fought these systems in production. This is the piece you hand to someone who just said "let's just use webhooks."

**The structural idempotency key is the most important original contribution here.** Everyone knows webhooks can duplicate. The insight that LLM non-determinism breaks naive parameter-hashing approaches — and that the fix is keys derived from structural invariants assigned pre-inference — is specific to agent systems and not obvious from traditional distributed systems literature. This alone justifies the piece.

**What's missing:** Tricot mentions Temporal and Inngest as durable runtimes but doesn't dig into the trade-off between them. The piece also stops at "pick a durable runtime" without addressing the practical question of which one and why — a companion piece on runtime selection would be valuable. The rate-limit math is excellent for GitHub/HubSpot/Salesforce but doesn't address what happens when you have dozens of sources each with different rate limit models.

**The hidden implication:** If per-source trigger diversity is correct (and Tricot makes a strong case that it is), then agent platforms that abstract away the trigger layer behind a uniform interface are hiding the wrong thing. The diversity IS the architecture. A platform that presents all sources as "events" with uniform semantics is lying to you in exactly the way that produces the production failures Tricot documents. This is the same argument as [[Smart Models Dumb Pipes]] applied one layer down: the trigger layer needs to expose source-specific delivery contracts, not paper over them.

**The reconciliation-is-not-optional argument is correct but uncomfortable.** Standing up a polling backstop on day one means building and maintaining two delivery paths from the start. Most teams won't do this. The ones that do will have more reliable systems. This is the kind of advice that's obviously right and rarely followed — like "write tests first" or "use database transactions."

**Connection to the wiki's themes:** This piece fills a gap in the wiki's agent architecture coverage. Where [[All Your Agents Are Going Async]] argues that HTTP is the wrong transport, Tricot argues that even within HTTP, webhooks alone are the wrong pattern. Where [[SQLite is All You Need for Durable Workflows]] covers the runtime side, Tricot covers the ingestion side. Together they bracket the durable-agent problem: how events get in, and how state survives while the agent works.

**Tricot argues webhooks need reconciliation; [[The Valley of Webhooks]] goes further and argues they're the wrong category of tool entirely.** The valley article reframes webhooks as a local optimum — an entire industry of excellent tooling (Svix, Hookdeck, Fivetran) built on a primitive that was designed to trigger side effects, not transfer datasets. Where Tricot prescribes webhook-plus-reconciliation as the least-bad default, the valley article asks whether the consumer-pull change log (a la Stripe's `/v1/events` or WorkOS's Events API) should be the primitive instead. The two pieces together form a spectrum: fix the pattern vs. replace the primitive.

## See Also

- [[All Your Agents Are Going Async]] — HTTP is the wrong transport for agents that outlive connections
- [[SQLite is All You Need for Durable Workflows]] — durable execution doesn't require durable infrastructure
- [[Smart Models Dumb Pipes]] — expose real semantics, don't paper over them
- [[Agent Orchestration]] — hub page for multi-agent coordination patterns
- [[Elements of Agentic Systems Design]] — ten-element taxonomy including coordination
- [[How Hightouch Built Their Long-Running Agent Harness]] — context management is the real engineering challenge
- [[Scaling Long-Running Agents]] — the runtime side of the durable agent problem
- [[Building Agents for Production Systems with MCP]] — production integration layer
- [[Guardrails and Feedback Loops]] — idempotency as a guardrail, not an afterthought
- [[Agent-Native Architectures (Every)]] — five principles including composability
- [[Structural Backpressure Beats Smarter Agents]] — architectural constraints beat model intelligence

---

*Source: [Event-Driven vs. Polling Architectures for Agent Triggers](https://agentblueprint.substack.com/p/event-driven-vs-polling-architectures), Michel Tricot, Agent Blueprint, 2026-05-14. Ingested 2026-06-05.*
