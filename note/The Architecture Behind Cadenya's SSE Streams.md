# The Architecture Behind Cadenya's SSE Streams

Robert Ross's end-to-end write-up of the event pipeline behind Cadenya's agent platform: the thesis is that Server-Sent Events are the easy part, and the real engineering is everything upstream — durable event storage (Postgres), fan-out (Pub/Sub), orchestration (Temporal), and live tailing (Redis streams) — before a message ever reaches an HTTP connection.

---

## What It Argues

Most SSE tutorials stop at "use Redis." Ross refuses that myopia and traces one event's full journey: an API write lands in Postgres via a Repository, publishes `ObjectiveCreated` to Pub/Sub, a subscriber starts a Temporal workflow that runs the agent loop (objective → iteration → parallel tool calls), each tool call emits an `ObjectiveEventCreated` through the same repository pattern, a lightweight Pub/Sub subscriber XADDs it into a Redis stream keyed by objective ID, and only then does a gRPC stream service tail it and forward bytes to a thin HTTP SSE server. The diagram is worth more than any summary of it: SSE connections are the last hop of a five-system chain.

## Key Quotes

> "The hard part is getting a message to that stream in the first place."

The entire piece in one line. Streaming to HTTP clients is a solved, packaged problem; the architecture that produces the message is not.

> "If you take one thing away from this blog, make it 'Use protobufs in Temporal workflow and activity parameters.'"

A strikingly narrow takeaway for a broad architecture post, but it makes sense: when the same protobuf messages flow through repositories, Pub/Sub envelopes, and Temporal parameters, the serialization layer becomes the unifying contract of the whole system rather than a per-hop concern.

> "Fanning out 'subscribers' from a Temporal workflow is an anti-pattern... Instead, repositories (almost) always push messages to Pub/Sub, and if a Temporal workflow needs to be started, it is started from a Pub/Sub subscriber."

A crisp opinionated rule about where orchestration belongs: workflows are consumers, never producers of fan-out. He also honestly defers the Outbox Pattern ("when it needs to") — a refreshing admission that a production system ships without the theoretically-correct guarantee.

> "Abusing the protobuf `Any` type felt like a sin in this case. We should *always* know the type of event in the message."

Rejecting `Any` in favor of a giant `oneof` discriminant on `ObjectiveEvent` trades extensibility for exhaustive type knowledge — a defensible call for an internal event bus where the compiler (and Go's generated `oneof` switch helpers) can catch unknown event kinds.

## Key Themes

#concept #pattern — CQRS as a discipline enforced by package structure (`internal/repositories/*/repository.go`), where each repository interface mirrors its gRPC service and every write is "store in database, publish to Pub/Sub."

#concept — Event replay via `Last-Event-ID`: ULID-sorted IDs make `id > lastEventId` a sufficient (if imperfectly ordered) replay query, and a deduplication set guards the gap between replay end and live-tail start — the kind of subtle race condition that separates demo SSE from production SSE.

#tool — The stack is Go + protobufs + Temporal + Pub/Sub + Redis streams + gRPC + Envoy, with topic names derived from proto descriptors via Go generics — type-safe messaging where the message type *is* the topic.

## Analysis

This is the most useful kind of systems post: written from production, with real code, including the ugly parts (ping events to keep load balancers alive, `StripLargeEventContent` to avoid blowing up Redis, ordering keys keyed per-event so ordered publishes don't add sequencing latency to other groups). The three-layer Temporal workflow design is a clean answer to the durable-agent-loop problem — but note the honest asymmetry: the *whole architecture* exists because an agent's progress must be observable in real time AND replayable after disconnect. That requirement — full event sourcing of agent runs with live tailing — is becoming the standard shape of agent platforms, and this is one of the few public, detailed blueprints of it.

The quibbles: the ULID replay is admittedly not perfectly sorted ("acceptable here"), which is the kind of shortcut that works until it doesn't under clock skew; and the Redis stream's 200-entry cap plus 30-minute expiry means SSE replay beyond that window depends entirely on the Postgres path — a coupling between two replays sources that the post gestures at but doesn't stress-test. Still, as a blueprint it's unusually honest about what's hard.

## Related Pages

- [[Your Backend Is Full of Hidden Workflows]] — the same diagnosis from a different angle: agent platforms accumulate hidden workflow machinery; Cadenya's Temporal-under-Pub/Sub rule is a concrete instance of the pattern, with an opinion about where workflows should be started from.
- [[Event-Driven Does Not Mean Decoupled]] — this pipeline is heavily event-driven yet tightly coupled by protobuf schemas; a nuance to that page's argument that events don't buy the decoupling people assume.
- [[Hash Slots and the Redis Batching Tax]] — another production Redis architecture deep-dive; complements Cadenya's Redis streams usage with the cluster-level costs Redis users hit.
- [[Change Is Inevitable — Versioning Event-Driven Systems (Bougria)]] — Cadenya's proto-envelope-as-topic design raises exactly the versioning questions Bouria's talk addresses; the oneof envelope here is one answer to schema evolution for event streams.

---
*Sources: [[raw/the-architecture-behind-our-sse-streams]], [[summary/the-architecture-behind-our-sse-streams]]*
*Last updated: 2026-11-09*
