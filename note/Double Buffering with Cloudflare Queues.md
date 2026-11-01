# Double Buffering with Cloudflare Queues

A terse engineering note describing a double-buffered telemetry pipeline on Cloudflare: two Queues alternate active/inactive on UTC-hour boundaries, producers route by modulo math on the event timestamp, and a half-hourly consumer drains the now-inactive queue, compresses the batch, and writes hourly partitions to R2 — explicitly designed around Cloudflare's pricing.

---

The author collects telemetry events and needs them segmented into UTC-hour groups. Rather than one queue plus a scheduler racing ingestion, they use *two* queues. The producer evaluates the payload's timestamp and routes to whichever queue is "active"; a consumer task triggers every hour on the half-hour, looks back one hour to determine which queue is now inactive and safe to consume, then compresses the batch and writes it to R2 with a simple partitioning strategy. Modulo arithmetic does the routing; UTC avoids daylight-saving headaches.

The payoff, in the author's words: the pattern "transforms chaotic streams of telemetry into a compressed hourly archive," yielding "a partitioned, serverless data lake optimized for downstream analytics, while keeping your cloud storage bills incredibly low." The stack is Workers, Queues, Workflows, and R2.

## Key quotes

> "Modulo math is used to route the payload to the active queue and again to drain the inactive queue. UTC is used to avoid DST handling."

The whole mechanism in two sentences. Double buffering here is not a memory-management metaphor but an operational one: the "swap" is time-driven, and the invariant is that a queue is only drained after it has provably stopped receiving writes.

> "The consumer task triggers every hour on the half-hour to drain the inactive queue... looks back one hour to determine which queue is currently inactive and safe to consume."

The half-hour offset is the subtle part — it gives the just-retired queue a full grace period so nothing written near the boundary is lost or double-read. A tiny amount of idleness bought for a large amount of correctness.

> "You get a partitioned, serverless data lake optimized for downstream analytics, while keeping your cloud storage bills incredibly low."

The stated objective is pricing-shaped architecture: batch sizes and cadences chosen to fit the billing model, not just the data model.

## Themes

#pattern #tool #concept

## Analysis

This is a sketch, not a tutorial — the body describes producer and consumer behavior but the accompanying code is absent from the fetched text, so the reader has to trust the invariant reasoning. Still, the pattern itself is sound and nicely old-fashioned: double buffering is a systems-classic technique re-invented at the serverless layer, where the "buffer" is a managed queue and the swap is the clock rather than the GPU.

What's genuinely interesting is the honesty about the driving constraint. Most pipeline designs claim to be shaped by data volume or latency; this one says out loud that it is shaped by *pricing*. Aligning work to provider billing buckets (hourly batches, storage-partitioned archives) is a legitimate and under-documented form of architecture — the cost model is part of the spec.

Caveats a reader should hold: a UTC-hour boundary is a global thundering-herd point if traffic is spiky; "looks back one hour" is safe only if producer clocks are trusted; and R2 partitioning by hour is only as good as the event timestamps embedded in payloads (late-arriving events will land in the wrong partition). None of these are addressed in the note.

## Relation to the wiki

This note strengthens [[Cloudflare K2 — Serverless Event Streams]]: both are Cloudflare-native takes on using object storage and queues as the durability layer, but where K2 rebuilds a partitioned log in R2 for general event streaming, this source shows the humble end-user version — a two-queue rotation for telemetry — a nice complement showing the same primitives at hobby scale.

It nuances [[Patreon Notification Fanout]] as another instance of queue-based fan-out/batching under a serverless pricing model, though fanout pushes work out while double buffering pulls it in on a fixed clock.

It also connects to [[Queues Don't Fix Overload]] from the opposite direction: that piece warns queues are not a capacity fix, while this pattern uses queues purely as temporal buffers with guaranteed drainage — an example where the queue is sized by the clock, not the load.

---
*Sources: [[raw/double-buffering-with-cloudflare-queues]], [[summary/double-buffering-with-cloudflare-queues]]*
*Last updated: 2026-11-02*
