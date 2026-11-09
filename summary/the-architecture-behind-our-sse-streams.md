---
url: https://cadenya.com/handwritten/the-architecture-behind-our-sse-streams
title: "The Architecture Behind Our SSE Streams"
author: Robert Ross
date_fetched: 2026-11-09
date_published: 2026-09-17
topics:
  - distributed-systems
  - agent-architecture
---

Robert Ross's end-to-end technical write-up of how Cadenya's agent platform streams objective events to browsers over Server-Sent Events. His thesis is that the SSE part itself is trivial — the hard problem is everything upstream: how a message gets produced, durably stored, fanned out, and made replayable before it ever reaches an HTTP connection.

The pipeline: an API request (POST /create objective) flows Cloudflare → Envoy (JSON-to-gRPC transcoding) → a thin gRPC service that delegates to a per-domain Repository. Repositories follow one rule — persist the event to Postgres, then publish it to a Google Pub/Sub topic named after the protobuf message itself (`topics.v1.ObjectiveCreated`), using Go generics so the topic name is derived from the proto descriptor. Pub/Sub subscribers — never the CQRS layer directly — start Temporal workflows. The agent loop runs as three nested workflows (`AgentObjectiveWorkflow` → `IterationWorkflow` → `CallToolWorkflow` in parallel), each recording events through `objectiveevents.Repository`, which publishes `ObjectiveEventCreated` to Pub/Sub. A lightweight SSE subscriber appends every event to a Redis stream keyed `objective_events:<objectiveID>` (XADD, max ~200 entries, 30-minute expiry), stripping oversized payloads.

The read path: a gRPC streaming RPC tails the Redis stream with XREAD (5-second block windows), while a separate "stupid simple" HTTP SSE server proxies the gRPC stream to the browser using go-sse. `Last-Event-ID` reconnect is handled by replaying Postgres events with `id > lastEventId` (ULIDs are sortable-enough), deduplicating against the live tail via a set so the gap between replay end and tail start doesn't double-send. Only "durable" event IDs advance the client's reconnect cursor.

Recurring themes: protobufs everywhere (repository arguments, Pub/Sub envelopes, Temporal workflow parameters — "use protobufs in Temporal workflow and activity parameters"), a single `oneof`-discriminated ObjectiveEvent envelope instead of protobuf Any, CQRS with repositories as the one persistence interface, and the rule that Temporal is started from Pub/Sub subscribers, never from the write path directly.
