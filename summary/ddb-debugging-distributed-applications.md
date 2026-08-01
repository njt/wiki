---
url: https://arxiv.org/abs/2607.06107
title: "DDB: Source-Level Interactive Debugging for Distributed Applications"
author: Yibo Yan, Junzhou He, Seo Jin Park
date_fetched: 2026-07-11
date_published: 2026-07-07
---

A 2026 USENIX ATC paper from USC that extends interactive source-level debugging (GDB-style breakpoints, call stacks, variable inspection) to distributed applications — a workflow long considered impractical across process boundaries.

Three mechanisms address the barriers that break interactive debugging at scale. **Distributed Backtrace (DBT)** embeds compact caller-context metadata in every RPC payload and reconstructs unified call stacks across RPC boundaries, letting a developer paused in service C inspect variables in the caller frames of services B and A. **Intent-preserving control plane** propagates breakpoints to dynamic process sets by targeting logical groups rather than physical processes, so breakpoints survive auto-scaling, rolling restarts, and computation migration. **Pause-Erased Time (PET)** virtualizes each process's clock via LD_PRELOAD shims, decoupling logical time from physical pauses so that stopping the world for inspection doesn't trigger leader elections, transaction aborts, or timeout cascades.

Integration requires only 10–60 lines of code per RPC framework (demonstrated with gRPC, ServiceWeaver, Nu, and Quicksand). On deployments up to 122 processes, `dbt` backtrace latency averages 30 ms, PET keeps perceived time jumps under 5 ms, and throughput overhead is 1–5%.

In a user study with 9 participants debugging three cross-service faults in a Raft cluster, DDB achieved 100% fault localization (vs. 44% for GDB and 33% for OpenTelemetry), with a median localization time around 8 minutes. Participants using baseline tools failed to localize cross-service faults in 61.5% of cases and often exceeded the 20-minute time limit. PET's ≤5 ms drift provides an order-of-magnitude safety margin against the most aggressive surveyed timeout thresholds (50–250 ms in LogCabin, RAMCloud, and Nu).

The prototype runs on x86_64 Linux (~22k LoC: 15k Rust coordinator, 1.4k Python per-process agent, ~2k C/C++/Go connector), assumes GDB as the underlying debugger, and virtualizes only POSIX time APIs (not bare-metal rdtsc or NIC timestamps).
