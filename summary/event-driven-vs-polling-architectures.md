---
url: https://agentblueprint.substack.com/p/event-driven-vs-polling-architectures
title: "Event-Driven vs. Polling Architectures for Agent Triggers"
author: Michel Tricot
date_fetched: 2026-06-05
date_published: 2026-05-14
publication: Agent Blueprint (Substack)
topics:
  - distributed-systems
  - agent-orchestration
---

# Event-Driven vs. Polling Architectures for Agent Triggers

**Author:** Michel Tricot
**Publication:** Agent Blueprint (Substack)
**Published:** May 14, 2026

## Summary

The piece argues that trigger architecture for agents is "load-bearing infrastructure" that most teams treat as an afterthought. The author contends that framing the choice as a binary between webhooks and polling is the root cause of brittle production systems, and proposes four distinct mechanisms and a decision framework per source.

---

## Why Agent Triggers Break in Production

The author describes two common camps: teams that chose webhooks for "real-time" and now face "duplicate writes, out-of-state updates, and silent gaps," and teams that chose polling for simplicity and now "burn through rate limits while their agent runs minutes behind reality."

The deeper issue is framing the choice as binary — push vs. pull — which produces architectures "brittle in exactly the ways agents can't tolerate."

---

## The Four Trigger Mechanisms and Their Delivery Contracts

1. **Webhooks** — HTTP POST callbacks. Delivery is "at-least-once, unordered, and best-effort across most major providers." Source-side retention measured in hours to days.

2. **Log-based CDC** — Reads database transaction logs via tools like Debezium. "Ordered per partition, durable, captures deletes." Tied to primary database, requires operational investment.

3. **Message-bus subscriptions** — Connects to durable pub/sub systems (Kafka, Pulsar, AWS EventBridge, Salesforce Platform Events, Google Pub/Sub). "Replayable, support fan-out, ordering depends on partitioning."

4. **Scheduled polling with conditional GET** — Client-initiated pull on a schedule, described as "the universal fallback." Cost scales with interval and endpoint count; misses intermediate states between polls.

CDC and webhooks have very different ordering and durability properties. A message bus is not a webhook. Conditional polling is not the same as naive polling.

---

## "Real-Time" Webhooks Are Mostly Marketing

The author argues that across major SaaS webhook systems, the same caveats recur: duplicates are possible, order is often not guaranteed, and delivery under stress can be delayed.

**Stripe:** Attempts delivery for up to three days in live mode. Events may arrive out of order. Stripe "explicitly states that it doesn't guarantee event delivery order." Every event carries a unique `event.id` for deduplication.

**Shopify:** Does not guarantee ordering within a topic or across different topics for the same resource — a products/update webhook might arrive before the products/create webhook for the same product. Retries up to 8 times over a 4-hour window. Retried webhooks carry the original payload from trigger time, not freshly fetched data, so retries can deliver stale data.

**HubSpot:** Retries up to 10 times over roughly 24 hours. Workflow-based webhooks have a longer retry schedule of about three days. The payload includes an `attemptNumber` field starting at 0, signaling consumers should expect the same event more than once.

**GitHub:** The outlier — documents handling for redelivery but not the same automatic retry model. Any agent trigger system using GitHub webhooks must either "poll the deliveries API on a schedule and replay failed deliveries via the REST redeliver endpoint, or run a parallel polling sweep against the resources of interest."

The author notes that guaranteeing delivery order forces a choice between blocking the entire pipeline on a single failure or maintaining order only during success — either way, consumers need out-of-order handling code. The key insight: "A webhook tells me something changed. It does not tell me the truth about what changed, or in what order."

---

## Polling Hits a Ceiling Faster Than Teams Expect

Polling feels cheap at first — one customer, a few endpoints, a 30-second interval. Then the math breaks.

**GitHub:** Caps authenticated REST requests at 5,000 per hour. Polling 100 endpoints every 30 seconds generates 12,000 requests per hour — 2.4x over budget. Even 42 endpoints consumes the entire hourly allocation. The mitigation is conditional GET with ETags or If-Modified-Since — a 304 response from an authenticated conditional GET doesn't count against the primary rate limit. But the author notes GitHub "is the exception."

**HubSpot:** Uses a per-app burst limit of 190 requests per 10 seconds on Professional plans for privately distributed apps, applied per app rather than shared across tenants. At 10 tenants, each gets ~19 requests per 10 seconds. At 100 tenants, it's 1.9. "The per-tenant budget degrades linearly with scale."

**Salesforce:** Enterprise starts at 100,000 requests per rolling 24-hour period per org. Running 10 tenants at 5 endpoints per tenant on a 30-second interval generates 144,000 requests per day — 44% over allocation.

---

## The Source Picks the Pattern

Teams spend days debating webhook vs. polling abstractly, but "the source system has already made much of the decision for them."

Many SaaS APIs don't support webhooks at all or only for a subset of objects: Odoo, Sage Accounting, and Pennylane don't expose outgoing webhooks. Even Xero uses a lightweight notification model triggering a separate API fetch for full resource data.

**Salesforce example:** Rather than a single generic webhook, it offers three overlapping event-driven mechanisms:

- **Outbound Messages** — Older SOAP-based, retries failed deliveries for up to 24 hours with no replay window.
- **Platform Events** — Developer-defined custom messages. High-volume offers a 72-hour replay window; standard-volume retains for 24 hours. Can be published as part of a transaction, supports multiple subscribers, carries an `EventUuid` field for deduplication.
- **Change Data Capture** — Automatically monitors enabled Salesforce objects with full coverage of UI, API, and bulk operations, delete tracking, and per-stream ordering via Replay IDs. Also a 72-hour replay window.

The author warns that "Event-driven" is not one thing even inside a single platform. CDC change events don't carry an `EventUuid` like Platform Events. Replay IDs are "positional stream markers rather than globally unique identifiers." The real risk: when Salesforce maintenance moves an org to a new instance, "the retained event stream can be reset entirely, and the replay IDs that worked yesterday no longer point anywhere."

The conclusion: "The right trigger architecture isn't the one I prefer. It's the best one the source will let me have."

---

## Log-Based CDC Reads the Source of Truth

When owning the database, CDC reads the transaction log directly — every insert, update, and delete captured, including intermediate states polling would miss.

**Debezium on Postgres:** Uses a logical replication slot to consume the Write-Ahead Log. The slot is durable and tracks consumer position. Requires `wal_level = logical` and works only against the primary, not standby replicas. Default delivery is "at-least-once" — in failure and restart scenarios, the same event can be delivered twice. Debezium writes to Kafka with the row's primary key as the message key, so per-row ordering is guaranteed within a partition. Cross-row ordering across partitions is not.

**Biggest failure mode:** WAL accumulation. When Debezium stops consuming, the replication slot holds WAL segments from recycling until disk is exhausted. Postgres offers `max_slot_wal_keep_size` as a circuit breaker, but if the slot exceeds that limit, it's marked as lost and requires a full re-snapshot.

CDC "reads from the source of truth, the log, not from application-level automation." For internal databases it's the most precise mechanism. For SaaS sources without database access, it's off the table.

---

## The Fine Print on Message Buses

Kafka, Pulsar, EventBridge, and Pub/Sub look similar from a distance but diverge in ways that matter for agent triggers.

**Kafka:** Producer defaults are at-least-once. Consumer semantics depend on commit configuration. With `enable.auto.commit=true`, offsets commit on the next `poll()` call after the previous batch was returned, producing at-least-once delivery in practice. Exactly-once semantics are available via Kafka transactions, "but the guarantee is scoped to Kafka-to-Kafka paths" — once the agent makes an HTTP call or invokes an external API, that guarantee ends.

**AWS EventBridge:** Target retries are configurable with a maximum event age up to 24 hours, retry attempts between 0 and 185. No ordering guarantee. Archive replay re-matches events against current rules rather than the rules at original delivery, so "a rule change between original delivery and replay can route a replayed event somewhere it never would have gone."

**Google Pub/Sub:** Opt-in exactly-once delivery "holds only when the subscription is configured as single-region (not multi-region)" and the subscriber uses streaming pull API with acknowledgments returned inside the ack deadline. Multi-region subscriptions or missed ack deadlines produce duplicates with no error signal. Ordering is also opt-in and key-scoped — redelivering one message "cascades into re-delivery of all subsequent messages for that key, including already-acknowledged ones."

The common thread: "exactly-once delivery is typically unavailable by default, available only with significant constraints, or scoped to a narrower boundary than teams assume." The message bus adds durability and replay but "does not eliminate the need for idempotent handling."

---

## Why Agents Change the Trigger Problem

Traditional trigger design assumes a short, deterministic handler. Agents don't fit that shape — they may "wait on a human approval, block on an external API, call tools recursively, and need to survive across restarts." The trigger has to land in a system that can hold durable state, not just return HTTP 200.

The author references **Vinoth Govindarajan's** piece on agent runtimes, noting that useful agents "often do work that crosses time" — waiting for humans, tools, webhooks, timers, background jobs, and external systems. They fail halfway, retry, resume, and create side effects "that should not repeat by accident."

**Harrison Chase's** concept of "ambient agents" is cited: agents that "listen to an event stream and act on it accordingly, potentially acting on multiple events at a time."

**LangGraph's** trigger taxonomy maps to four categories: event-driven, state-based, time-based, and external-input triggers. The `interrupt()` function freezes execution at a checkpoint, persists the thread, and resumes when human input or a webhook callback arrives — "no compute is consumed while the thread is paused."

The durable execution pattern is what makes long-running agents possible. **Temporal** implements it with workflows and signals — each agent is a long-running workflow that maintains state and waits for Signals. **Inngest** uses memoized steps where `step.run()` caches results, so completed steps return cached results instantly on resume. Without durable execution, stateless retry patterns re-invoke LLM calls that had already completed — "the token re-burning problem."

The author warns that picking a trigger mechanism without a matching runtime "is where agent systems collapse."

---

## Why Mature Systems Go Hybrid

The author's working assumption: "events will be missed. Not might. Will."

**Stripe** documents the duplicate problem directly: endpoints "might occasionally receive the same event more than once" and recommends logging processed event IDs. For terminal payments, during outages "reader action webhooks might be late" and recommends querying the resource directly.

**Merge's** guidance: webhook-plus-polling with a 24-hour safety-net poll at minimum, on top of more frequent polling tuned to the use case. **Svix** offers polling endpoints as a complementary delivery mechanism. The shared logic: "consumers need a reconciliation process to keep the system honest."

"The reconciliation job is what makes the hybrid trustworthy" — it compares local state against source state and backfills gaps. The same idempotency check runs regardless of whether the event arrived via webhook or polling sweep.

The recovery procedure: the provider retry window exhausts, the circuit breaker opens, and the system switches to polling reconciliation. When normal webhook delivery resumes, the idempotency store prevents re-processing of events already caught during polling.

Teams that skip reconciliation "ship agents that look correct until the first webhook outage, and then quietly emit wrong decisions for hours."

### What the hybrid pattern doesn't solve

The hybrid is not magic: it doesn't give exactly-once delivery on its own (still depends on idempotent handlers), doesn't catch events the source never emitted, doesn't help with sources lacking both webhooks and a reasonable polling surface (common in legacy on-prem systems), and "doubles operational cost" — two delivery paths means two failure modes to monitor.

If the source publishes a durable, replayable bus (Kafka, Pulsar, Salesforce's Pub/Sub API), prefer that. The bus gives reconciliation through replay, avoiding a parallel poll loop. "The hybrid is the right default when the source is HTTP-only. It isn't the right default when something better exists."

### Idempotency makes the hybrid honest

Duplicates are implied by the delivery contract. Polling can duplicate across intervals. Webhooks can duplicate on retry. CDC can duplicate on replay. "No trigger is exactly-once on its own. Exactly-once effects come from idempotent handlers at the write boundary."

Agents make idempotency harder because writes following a trigger may be non-deterministic in content — the LLM is a non-deterministic client. If you hash tool parameters into the idempotency key, a retry that generates slightly different parameters produces a different key, making the destination treat the retry as a new write.

The key must derive from stable structural context only: `(agent_run_id, step_id, tool_name, call_index)`, optionally scoped by business identifiers. `call_index` disambiguates cases where a single step invokes the same tool more than once. All four components are assigned before LLM inference runs and do not change on retry.

The author provides a Python example of a `structural_key` function using SHA-256 hashing of the concatenated components.

The warning: "Picking an at-least-once trigger without a deduplication store at the write boundary is how payment duplicates, duplicate CRM contacts, and duplicate outbound emails get into production."

---

## How to Choose the Right Trigger Architecture

Four questions replace the webhook-or-poll binary:

1. **What latency does the workflow actually tolerate?** Most tolerate more than teams admit. A support ticket agent doesn't need sub-second triggers. A payment dispute agent might.

2. **What does the source system actually offer?** Webhooks, events, CDC, polling only. If the source doesn't expose webhooks for the objects needed, the question resolves to polling.

3. **What is the cost of a missed event versus a duplicate?** If duplicates are cheap and misses expensive, use at-least-once plus reconciliation. If duplicates are catastrophic, stronger delivery semantics and idempotent handlers are needed. The author notes in most agent systems, both are expensive — hence the hybrid pattern.

4. **Where does the trigger land?** The delivery mechanism must match a runtime that can hold state for as long as the agent needs. A webhook landing in a runtime with no memory of prior side effects "is fragile by design."

The author includes a comparison table (described in text) showing how the four mechanisms stack up across: latency, ordering, dedup support, replay window, rate limits, state support.

The framework should be applied per source. A multi-source agent system will use different mechanisms for different sources, and "that isn't a design flaw. It's the correct response to heterogeneous source capabilities."

---

## Do This Next

Five ordered steps for starting the trigger layer:

1. **Inventory delivery contracts before architecture.** For every source, document whether it supports webhooks, the retry budget, ordering guarantee, and rate limit. Skip this and you'll discover pattern mismatches in production.

2. **Build the idempotency store before the first integration.** A structural-key dedup store at the write boundary makes every other choice survivable. Without it, the first duplicate webhook ships a duplicate write.

3. **Default to webhook plus reconciliation, not webhook alone.** Stand up the polling backstop on day one, even if webhooks work perfectly in staging. "The first webhook outage is the wrong time to design the recovery path."

4. **Pick a runtime that can hold state across waits.** If your trigger lands in a stateless function, the next fifteen-minute LLM call kills the run. Durable workflow runtimes solve this. Plain HTTP handlers don't.

5. **Treat trigger choice as per-source, not per-system.** A multi-source agent will use different mechanisms for different sources. "Pretending one pattern fits everything is how trigger layers age into rewrites."

---

## What It All Comes Down To

"There is no universal right answer to webhook or poll." In production, the right pattern combines fast-path events, reconciliation polling, idempotent handlers at the write boundary, and a durable runtime that can hold state across gaps.

The author runs the same four questions per source: latency tolerance, what the source offers, cost of miss vs. duplicate, and where the trigger lands. "The answers usually produce a different architecture for every source. That is the point."

### Closing Framework

"The trigger and the write are two halves of the same problem. Both have to be designed for at-least-once reality from the first day." The author references their companion piece on idempotent writes as covering the write boundary, while this piece covers the trigger boundary — together "they bracket the most common failure modes I see in production agent systems."

Final call: "Trigger architecture is the under-invested layer in most agent stacks I look at. It deserves the same rigor as model selection, retrieval, and prompt design. Not as a footnote. As a foundation."

---

## Comments

**SinghCoder** (May 22, 2026): Praised the delivery-contract framing, noting it's the part most agent trigger discussions skip. Mentioned building "Watchline" in this space, and said the article clarified that the contract "probably has to travel with the wakeup itself: dedupe id, cursor/checkpoint, retry/backfill window, ordering assumptions, and whether the event is fresh or reconciled." Otherwise the agent just sees "event happened" and treats it as truth even when the source only promised best-effort delivery.
