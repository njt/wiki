---
url: https://arxiv.org/html/2607.05188v1
title: "Latent Programming Horizons in Coding Agents"
author: André Silva, Han Tu, Martin Monperrus
date_fetched: 2026-07-18
date_published: 2026-07
---

Silva et al. (KTH) probe the residual streams of two open-weight coding-agent
models (Qwen3.6-35B-A3B and Laguna-XS.2) to decode what the model internally
represents about the program being edited. They collect 22,714 trajectories
across SWE-Bench-Verified and SWE-Bench-Pro using mini-swe-agent, training
logistic-regression probes on four binary program properties: whether the code
parses, passes its full test suite, reduces failing tests, or introduces
regressions.

Probes decode correctness at AUC up to 0.83 and partial correctness up to 0.84.
Well-formedness (parsing) probes collapse to near-chance due to heavy label
imbalance — over 92% of edits already parse. Encoding strength follows an
inverted-U pattern across layers (weakest early, peaking mid-model), and
Qwen3.6 encodes roughly 0.10 AUC stronger than Laguna. Probes transfer across
benchmarks with only small drops (0.04–0.09 AUC).

The paper's headline finding is a *latent programming horizon*: probes
trained to predict program correctness *k* steps ahead remain above chance for
at least 25 steps, plateauing above randomness out to k=50. The authors frame
this as "the first evidence of long-term horizon by coding agents" and suggest
it opens directions for monitoring and steering agents from within the latent
space. Limitations include the correlational (not causal) nature of probing,
label-imbalance effects, and the narrow two-model scope.
