---
url: https://tailscale.com/blog/making-tailscale-faster
title: "Making Tailscale Faster"
author: The Tailscale team
date_fetched: 2026-09-25
date_published: 2026
topics:
  - distributed-systems
  - developer-tools
---

Tailscale walks through a batch of data-plane performance work: cutting memory overhead for small packets on Linux/Android by unpacking large GRO reads in place instead of copying each packet into its own 64 KiB buffer (~5% speed-up), shortening packet queues, a multi-queue system landing in the second half of 2026 that gives subnet routers, app connectors, and exit nodes parallel lanes scaled to machine resources rather than peer count, and `writev` support to hand scattered packet data to the Linux kernel without combining it first.

Separately, netmap caching (currently behind a feature flag, defaulting in v1.104) lets devices start and connect using a disk-cached copy of the network map when the control plane is slow or unreachable — warm starts were one to two orders of magnitude faster in the field — with honest caveats about persistent disk requirements and SD-card wear.

The post closes by naming a gap: existing performance tooling is point-to-point, rigid, lacks QUIC/HTTP-3 support, and above all is not "Tailscale-aware" — it cannot tell you whether a connection is direct or going through DERP, or whether a peer relay would help. Tailscale is exploring a Tailscale-native monitoring and testing toolkit.
