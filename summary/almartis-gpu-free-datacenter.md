---
url: "https://almartis.xyz/gpu-free-datacenter.html"
title: "AI Datacenters Were Built for GPUs. What Happens When You Remove the GPUs?"
author: "Alassane Sakandé, Kevin Simpore"
date_fetched: 2026-05-31
date_published: 2026-05
source_type: blog
publication: Almartis
category: Infrastructure
topics:
  - ai-infrastructure-and-hardware
---

# AI Datacenters Were Built for GPUs. What Happens When You Remove the GPUs?

By Alassane Sakandé and Kevin Simpore, Almartis blog, May 2026 (~10 min read, Infrastructure category).

## Summary

The article traces how AI datacenter networking evolved from traditional north-south bursty traffic to GPU-driven east-west all-to-all patterns, then argues that both InfiniBand and Ultra Ethernet solve a problem that's downstream of computational assumptions — the need to synchronize thousands of GPUs. Almartis proposes an alternative: associative memory systems with structured retrieval that eliminate GPUs entirely, enabling a 1-tier non-blocking full mesh architecture. The goal shifts from maximizing throughput to minimizing retrieval latency.

## Full Content

### Introduction: Traditional Datacenter Networking

Datacenter building was historically "a well-understood, predictable exercise in utility engineering," mixing compute, storage, and networking. Traffic was mostly north-south and bursty; TCP/IP handled drops with retransmission, and minor delays were acceptable.

### The AI Shift

The network's role fundamentally changed with AI training. The network "directly determines accelerator utilization." Modern AI clusters operate as "a massive, distributed supercomputer where thousands of GPUs must continuously swap parameters." Traffic becomes east-west, with all-to-all and all-reduce patterns carrying a small number of extremely large elephant flows. When accelerators run at 800 Gb/s, "the critical metric flips from average latency to Job Completion Time (JCT) and tail latency." Because training executes in synchronized steps: "One delayed packet can stall thousands of GPUs."

### Solving Packet Loss Created Head-of-Line Blocking

RDMA via RoCEv2 enables GPU-to-GPU direct memory access but is "highly sensitive to packet loss." Priority Flow Control (PFC) — a pause mechanism instructing upstream devices to stop transmitting when switch buffers fill — creates "head-of-line blocking, where unrelated traffic becomes trapped behind congested flows." Congestion spreads, queue depths grow, and "GPUs remain idle while waiting for retransmitted packets or congested flows to clear."

### The Incumbent: InfiniBand and Rail Optimization

NVIDIA's InfiniBand is "a native lossless fabric designed specifically for high-throughput, low-latency clustering." Three vectors:

- **Scale Up:** High-speed interconnectivity within a single chassis/node
- **Scale Out:** Horizontal expansion connecting multi-GPU nodes across a data hall
- **Scale Across / DCI:** Linking clusters across sites when power and cooling limit single-site expansion

"we're entering the end of scale-up" as NVIDIA ships complete racks with NVLink and NVSwitch. Rail-optimized topologies map each of 8 GPUs per node to a dedicated NIC, splitting the fabric into "8 parallel, isolated physical switch planes" — reducing congestion and improving failure containment.

### ECMP and Elephant Flows

Traditional ECMP hashes headers to assign flows to paths. Works for many small independent flows but fails with AI's massive elephant flows, causing "collisions where multiple large flows become pinned to the same physical links while alternative paths remain underutilized." Modern AI switches respond with Dynamic Load Balancing (DLB) and packet-spraying mechanisms that break elephant flows apart and schedule based on real-time congestion.

### Ultra Ethernet Consortium

Ultra Ethernet represents "a comprehensive re-architecture of Ethernet designed specifically to challenge InfiniBand on AI workloads." Key features:

- Replacing PFC with transport-layer intelligence
- Packet Spraying across all available links rather than hashing flows
- Hardware-level packet reordering at the NIC layer
- Virtual Output Queueing (VOQ) to buffer by destination

Comparison table:
| InfiniBand | Ultra Ethernet |
|---|---|
| Native lossless | Open Ethernet ecosystem |
| Proprietary | Multi-vendor interoperability |
| Vendor lock-in | Transport-layer intelligence |
| PFC-based | Packet spraying + VOQ |
| High cost | Economies of Ethernet scale |
| Closed ecosystem | — |

### GPU-Free AI Datacenters

"The complexity of modern AI infrastructure is not accidental. It is downstream of the computational assumptions the models themselves impose."

Almartis's alternative: "associative memory systems built around explicit, addressable, and deterministic memory structures" rather than large-scale tensor optimization. The architecture emphasizes "structured retrieval and compositional memory operations." This enables "a GPU-free, non-blocking, 1-tier full mesh architecture built around high-density CPU nodes and 51.2Tb silicon switching fabric," where storage and compute share the same physical domain.

The limit for a 1-tier rail-only cluster with NVIDIA's latest generation: **216 Blackwell Ultra GPUs**, consuming more than twice the power of their GPU-free cluster. "Insignificant for training capable LLM models."

**Claim:** Their "150-kW cluster can train a system from scratch to common sense" — understanding of objects, physical world context, and ability to learn anything from that foundation.

### From Throughput to Retrieval Latency

AI networking has been defined by "how to scale synchronization between accelerators efficiently enough to keep increasingly large GPU clusters utilized." The new objective: "minimizing retrieval and coordination latency across structured memory systems." Central question: "what happens when the architecture itself reduces the need for synchronization in the first place?"

## Footer

"A GPU-free, 1-tier, non-blocking full mesh architecture built for associative memory." © 2026 Almartis.
