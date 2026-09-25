---
url: https://jyn.dev/tokens-too-cheap-to-meter/
title: "Tokens Too Cheap to Meter"
author: jyn
date_fetched: 2026-09-25
date_published: 2026-09-25
topics:
  - ai-infrastructure-and-hardware
  - ai-product-and-business
---

A data-dense essay arguing that the per-task cost of AI intelligence is falling ~2.5 orders of magnitude per year, driven by GPU efficiency gains (~2x/2yr), model architecture improvements (Mixture-of-Experts, Mamba hybrids), and inference-engine software gains (10-50%/yr in vLLM, NVIDIA, Intel MLPerf results). The author projects LLMs becoming infrastructure embedded everywhere, frontier-quality local inference on commodity hardware within 3-6 years, and quality/access — not token supply — becoming the binding constraint.

The evidence is layered: hardware efficiency curves, pareto-frontier charts showing 2026 cost-per-task ~100x cheaper than 2025, MoE models 7x smaller at equal quality, and Mamba-Transformer hybrids (Nemotron-H-47B holding 1M+ tokens in 32 GB vs ~120 GB for a comparable dense model). Specialized classifiers like TypeSafe's Jev ($42/billion tokens, output "too cheap to meter") and the open-weight Laya push another 1-2 orders of magnitude down, enabling tools like `jgrep` that call a model from inside a Unix pipe.

The forward-looking half is the more interesting one: tokens become cheaper than tool calls (grep on a MacBook already costs 4.5 orders of magnitude less than an LLM turn), so models get embedded *in* tools; supply-side and demand-side Jevons paradoxes mean efficiency gains translate into more compute built and new uses found; and software gains a fourth option — "tell an LLM to build it" — which attacks incumbents' moats. The author expects a barbell market: frontier labs sell the hardest tasks, everything else competes with open weights.
