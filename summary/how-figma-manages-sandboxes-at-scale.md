---
url: https://www.figma.com/blog/how-figma-manages-sandboxes-at-scale/
title: "How Figma manages sandboxes at scale"
author: Pratik Sarode and Jordan Poulton (Figma Sandbox Platform team)
date_fetched: 2026-11-06
date_published: 2026-11-06
topics:
  - security-and-sandboxing
  - distributed-systems
---

Figma built Nimbus, a centralized sandbox platform, because its AI features (Figma Make, the design agent) needed tens of thousands of sandboxes running untrusted code with varying memory and network permissions. This is the first post in a three-part security engineering series on sandboxing at scale.

Nimbus works on a Kubernetes-operator-like contract: a product team declares a **namespace** (container image, resources, health checks, warm-pool bounds, egress policy), and the Nimbus control plane handles provider selection, resource allocation, security policy, and lifecycle. Three APIs — `GetOrCreate`, `Get`, `Delete` — hide everything provider-specific. Every sandbox is provisioned from a shared base image and supervised by `sandboxd`, which owns the process tree and reports CPU/memory/network observability.

The security design is the most distinctive part. A container boundary alone isn't enough: instead of a blunt network allowlist, Nimbus treats reads differently from writes — fully trusted hosts from the namespace get free access, while everything else permits only bodyless `GET`/`HEAD`/`OPTIONS` (enough for web search and page fetching, never `POST`/`PUT`). Where a provider supports egress inspection the policy is installed there; otherwise a proxy sidecar evaluates each outbound HTTPS request. Credentials are stored encrypted inside the sandbox and swapped for real values only at egress time, and only to their intended destination. If the sidecar can't attach or the policy can't be evaluated, the request fails closed.

On reliability, Nimbus maintains warm pools of pre-provisioned sandboxes (90% latency cut, availability up from 99% to 99.95%), claims sandboxes exactly-once via Postgres transactions, sizes pools from claim velocity rather than fixed targets, and routes around degraded providers — weighting on recent outcomes, treating ambiguous timeouts separately from confirmed failures. A simulator runs the control-loop logic against synthetic failure conditions; it caught a real bug where slow-but-successful provisioning batches kept draining pools while the controller waited. Launched internally in November 2025, Nimbus is now shared infrastructure across product surfaces, with automatic provider failover replacing on-call pages.
