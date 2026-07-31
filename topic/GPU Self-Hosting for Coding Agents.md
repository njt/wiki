# GPU Self-Hosting for Coding Agents

A benchmark-driven reality check on what you actually get when you move coding agent inference from frontier APIs to your own hardware. The short answer: you probably won't save money, but you might need to do it anyway.

---

## The Benchmark

aistack (imec) ran **64 real coding tasks** through three tiers: self-hosted GPUs, rented cloud hardware, and commercial frontier APIs. The goal was to measure what self-hosting actually delivers — not what the spec sheets promise.

## Key Findings

### Cost: Self-Hosted ≈ Rented (When Utilization Is Honest)

> "Your own box, priced at the hours it actually spends working lands in the same ballpark as renting one."

The critical variable is **utilization rate**. GPUs sitting idle between coding sessions destroy the economics. The authors note that utilization rates "paint a very different picture for the setups we tested" — the implication being that most self-hosted setups have far lower utilization than their owners assume.

This rhymes with [[Inference Cost Napkin Math]], where duty cycle is "the 5× multiplier nobody measures."

### Quality: The Single-GPU Gap Is Real

A smaller model that fits on a single consumer GPU solved roughly **a third** of the 64 tasks. The frontier model (accessed via API) solved **40 out of 64**. The delta isn't marginal — it's the difference between a tool you can rely on and one you can't.

### Quality: Multi-GPU Can Match Frontier (Barely)

The largest open-weight model **matched the frontier result** — but only on an **8×B200 node**, and only supporting "a couple parallel sessions at most." That's a six-figure hardware commitment to serve maybe two developers simultaneously. See [[NVIDIA B300 vs H200 GPU Analysis]] for why the B200's 288GB HBM3e and 1,400W TDP make this a data-center-scale proposition, not a homelab one.

## Critical Analysis

### The Utilization Trap Is the Real Story

The finding that self-hosted costs roughly equal renting *at honest utilization* is devastating for the "buy a GPU and save on API bills" pitch. Most individual developers and small teams cannot keep a GPU saturated. The math only works if you're running inference continuously — which coding agents, with their bursty human-in-the-loop workflow, structurally are not.

This is the same dynamic that killed on-premise datacenters in the first cloud era: utilization is the invisible tax that makes "cheaper to own" pencil out as "more expensive in practice."

### The "Data Can't Leave the Building" Argument Is Doing a Lot of Work

The article's two non-cost justifications — data residency and rate-limit freedom — are real but narrow. Most teams don't have data that literally can't touch a frontier API. The rate-limit argument is stronger: when your entire engineering workflow depends on a model that can throttle you, you don't control your own throughput. But that's a scale problem, and most teams aren't there yet.

### This Validates the Hybrid Approach

The benchmark effectively validates what [[Local Qwen Is Not a Worse Opus]] argues: local models are a *different tool*, not a worse version of frontier models. Use local for analysis, classification, and non-critical tasks; pay for frontier when correctness matters. The article's data supports this — the single-GPU model solving 1/3 of tasks is useful for the right 1/3, not as a replacement.

### The Missing Benchmark: Model Improvement Rate

The article measures a snapshot, but the real question for GPU buyers is depreciation risk. If open-weight models improve 8–10 months behind frontier (per [[Open models lag state-of-the-art closed models by 4 months]]) and GPU hardware depreciates on a 2–3 year cycle, the buyer is racing both clocks. The article doesn't model this, and it's probably the most important variable for anyone writing a purchase order.

## Key Themes

- `#economics` — Cost parity between self-hosted and rented at realistic utilization
- `#hardware` — Single GPU vs. 8×B200: the hardware spectrum for coding agent inference
- `#benchmark` — 64 real coding tasks as a more honest metric than standard evals
- `#tradeoff` — Quality vs. control vs. cost: pick two

## Recommendations

1. **Don't buy a GPU to save money on API bills.** At honest utilization rates, you won't.
2. **Buy a GPU because you need data residency or throughput guarantees.** Those are the defensible reasons.
3. **If you do self-host, measure utilization obsessively.** It's the number that determines whether you're saving money or burning it.
4. **Consider the hybrid model first:** local for high-volume/low-stakes, frontier for the hard problems.

---

*Sources: [[raw/gpu-self-hosting]]*
*Last updated: 2026-08-01*
