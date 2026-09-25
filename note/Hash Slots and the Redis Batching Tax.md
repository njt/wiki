# Hash Slots and the Redis Batching Tax

A gig-economy delivery engineer discovers Redis cluster hash slots the hard way — through traces showing one logical read exploded into scores of single-key `MGET` round trips — and iterates to a fix (brute-forced hash tags, slot-aware batching, `EVAL`-based expiry, CSV values) that turns scattered single-key traffic into a handful of legal multi-key commands. The essay's real payload, though, is methodological: build the benchmarking harness first, because in the post-Claude era the excuse for not building one is gone.

---

## The shape of the problem

The service matches couriers to delivery offers, and its latency budget is dominated by a routing engine — "when the bucket overflows, everybody gets wet." Route estimates are highly repetitive at a busy scale: couriers a block apart produce nearly identical requests, restaurants don't move. The cache has to be *shared* (process-local caches don't survive worker churn), keyed on H3 hexes rather than raw coordinates, with hex resolution as the accuracy/hit-rate lever.

## What went wrong: keys that never share a slot

The naive key `<origin hex>:<dest hex>:<resolution>` is exactly the shape Redis clustering punishes. Redis distributes 16,384 hash slots via `CRC16(key) mod 16384`; multi-key commands are only valid within one slot, and two keys differing by a character land in unrelated slots. A well-built client does the correct thing with such input — groups keys by slot and puts every group on its own wire — which turns your one batched read into hundreds of serialised single-key ones. The author found this in traces, not docs:

> "A single read was showing up as scores of individual MGET spans for a single key each."

That's the invisible tax of clustered Redis: the client *rescues* you from an error by converting it into latency.

## The fixes

- **Hash tags.** Redis hashes only the text inside `{...}` in a key, ignoring the rest. The team treats the tag as a template (`{routing:v1:<n>}`) and brute-forces integers at startup — explicitly likened to Bitcoin miners grinding for a hash with a property — until every node owns enough tags to spread load.
- **Slot-aware fan-out.** Each Redis node is single-threaded, so 20 concurrent requests to one node are a queue. Computing slots locally and grouping before fan-out means "the fan-out is over nodes rather than arbitrary chunks, and every request is doing useful work."
- **Expiry via `EVAL`.** `MSET` has no expiry argument; pipelined `SET ... EX` multiplies commands, and `MSET`+`EXPIRE` leaves a crash window that strands immortal keys. A Lua script does it in one shot.
- **Ditching JSON.** "The cached value started as JSON, because that's what I've always done." At scale the serialization overhead was real CPU; CSV fixed it. A good reminder that "small payload" intuitions don't survive `n`.

## The methodological coda

The author leans on Pike's rules 2 and 3 — measure before tuning, and don't get fancy until you know `n` is big — and admits the trap was realising too late that *his `n` was reliably large*, so the simple approach was wrong from the start. The sharper point is organisational: he skipped benchmark harnesses because startup culture priced a 2-point ticket into a 5. That pricing is dead:

> "I think in the post-Claude era of programming, where such a benchmarking harness is a matter of providing a spec and waiting 10 minutes, this is no longer acceptable."

## Themes

#concept #tool #pattern

## Analysis

This is a well-observed war story about a failure mode that documentation renders nearly invisible: Redis clusters don't reject your scattered keys, they *absorb* the cost for you, and only distributed tracing reveals the tax. The hash-tag trick is old, under-documented Redis lore, and the brute-force-mining framing is the most memorable presentation of it I've seen.

The essay's two halves are less integrated than they could be — the Pike-rules reflection is somewhat bolted on — but the coda is the part that matters to this wiki. The claim that AI collapses the cost of *verification infrastructure* (harnesses, benchmarks) specifically, not just code production, is a quietly important variant of the "agents make measurement cheap" argument. The author's own admission — he measured the bottleneck correctly but still skipped the harness — shows that knowing Pike's rules isn't the barrier; the organisational accounting around ticket estimation was. Agents removing the excuse doesn't automatically remove the accounting, which is why the author has to make it a *habit* rather than a policy.

There's also an irony worth noting: an article about eliminating redundant work via caching was itself produced by discovering, one expensive surprise at a time, things the documentation already knew. Traces and benchmark harnesses are the general answer to "why didn't anybody tell me" — they tell *you*, without waiting for somebody.

## Relation to the wiki

- [[How We Made Claude AI Faster]] — same thesis from the other end: Anthropic's speed sprint began with measurement and benchmark ratchets; this source shows the harness-first habit as the individual practitioner's version of the same discipline.
- [[Just Brute Force Your Embeddings]] — a kindred "measure the simple thing before reaching for the fancy system" essay; here the lesson runs in the opposite direction (the simple thing *failed* because `n` was large), which nuances both.
- [[Making Tailscale Faster]] — another performance-engineering field report where the wins came from understanding the system's actual dispatch behaviour (batching, single-threaded queues) rather than from any exotic technique.
- [[The New Software Lifecycle]] — the post-Claude pricing claim (spec → harness in 10 minutes) is a concrete instance of its "verification is cheap now" argument.

---
*Sources: [[raw/why-didnt-anybody-tell-me-about-hash-slots]], [[summary/why-didnt-anybody-tell-me-about-hash-slots]]*
*Last updated: 2026-09-25*
