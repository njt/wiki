# Making Tailscale Faster

Tailscale's engineering post on a wave of data-plane performance work: zero-copy handling of small packets inside large GRO reads, multi-queue processing for subnet routers/app connectors/exit nodes, `writev` batching, and netmap caching for fast startup without a reachable control plane — closing with an argument that network performance tooling needs to be topology-aware.

---

The post is a nice example of infrastructure performance work that is mostly about *memory layout and copying discipline* rather than exotic algorithms. Most packets are ~1 KiB, but Linux's efficient path (GRO) delivers up to 64 KiB at a time, and wireguard-go only offered a 64 KiB buffer to unpack into — so every small packet got copied into a huge buffer. The fix ("leave packets where they landed") is the container-shipping inversion: stop unpacking containers, just index into them. A ~5% speedup for that is a reminder that the boring per-packet copy is often the bottleneck.

The multi-queue change is the architecturally interesting one: subnet routers previously ran all streams through one ordered, single-thread pipeline because per-stream ordering must be preserved — the fix is to assign each stream a lane and parallelize *across* streams, not within them. Lanes scale to machine resources rather than peer count, which matters because tailnets range from homelabs to cloud deployments with hundreds of peers.

Netmap caching is the most user-visible piece and the most philosophically loaded: a device can start and connect with a disk-cached network map before ever reaching the control plane. Warm starts were one to two orders of magnitude faster than cold. The caveats are stated honestly (needs prior connection, needs persistent disk, SD-card wear, big-tailnet write amplification). It's a graceful-degradation design: control-plane availability is treated as an optimization, not a precondition.

The closing section on tooling is quietly the most important claim: general-purpose performance tools can't tell you whether a connection is direct or DERP-relayed, or whether a peer relay would help. Performance in an overlay network is a property of the *path*, and tools blind to the path can only measure symptoms.

## Key quotes

> "It's a bit like container shipping: the ports, ships, and trucks are built for one container shape, however full it happens to be."

The framing for why 1 KiB packets pay for 64 KiB buffers — allocation size coupled to the efficient receive path, not to payload size. Commentary: a genuinely good analogy for a class of bug (buffer-sizing dictated by hardware/protocol efficiency) that shows up everywhere in systems code.

> "A single lane was shared across many connections, because a receiving application must never see its own packets arrive out of order."

The precise statement of why the old design was single-threaded: ordering is per-stream, so parallelism must also be per-stream. Commentary: a textbook case of an invariant (per-stream ordering) being over-applied as a global serialization constraint.

> "Bad network conditions — that's really the space where people can get a lot of utility out of netmap caching."

Claus Lensbøl on where the feature pays off: airplane Wi-Fi, hotel filtering, far-from-control-plane devices. Commentary: the feature is essentially "assume the last known topology until told otherwise," which is the right default for intermittent connectivity.

## Themes

#concept — zero-copy packet handling, per-stream parallelism
#pattern — caching for graceful degradation when the control plane is unreachable
#tool — Tailscale data plane, wireguard-go, planned Tailscale-aware perf toolkit

## Critical take

The most credible thing about this post is what it admits: the improvements are Linux/Android-only for now, multi-queue is still unreleased, and the netmap cache has real failure modes (SD wear, large-tailnet write amplification). It reads as an engineering report, not marketing — though the framing of agentic workflows and CI as the beneficiaries is clearly aimed at the current AI-tailwind moment. The unresolved tension: multi-queue parallelism helps aggregate throughput, but the per-stream latency guarantees that forced the single-lane design still bind — the post asserts lower delay without showing distributions. And the tooling section, while correctly diagnosing that performance measurement must be path-aware, is a promise rather than a product.

## Related pages

This source strengthens [[Headscale]]'s picture of the Tailscale ecosystem: Headscale reimplements the control plane this post treats as an availability-optimizable component — netmap caching is exactly the kind of client behavior a self-hosted control server must stay compatible with. It nuances [[Domenic Denicola's Agentic Coding Setup]], which uses Tailscale as undifferentiated plumbing for YOLO agents; this post explains why that plumbing is getting faster and why agentic workloads (many short-lived connections, latency-sensitive startup) are precisely the ones the multi-queue and netmap-caching changes target. It complicates any assumption in [[Postgres Transactions Are a Distributed Systems Superpower]]-style local-first thinking that the network layer is a fixed cost: here the network itself is being optimized with the same locality-and-copying discipline database people apply to buffer pools.

---
*Sources: [[raw/making-tailscale-faster]], [[summary/making-tailscale-faster]]*
*Last updated: 2026-09-25*
