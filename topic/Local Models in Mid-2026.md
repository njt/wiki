# Local Models in Mid-2026

Matt Coles surveys the local LLM landscape in mid-2026 and finds that five engineering advances — sparse attention, mixture-of-experts routing, latent KV cache compression, multi-token prediction, and 4-bit quantization — have collectively closed the gap between open-weights models and the closed frontier for everyday coding, writing, and agent work. The article's central observation is that nearly every competitive open model is now sparse/MoE, not dense, and that the ratio of total to active parameters (Qwen 3.6: 27B total / 3B active; Kimi K2.6: ~1T total / 32B active) is the number that actually matters for local hardware. The irony Coles lands on: the models are finally good enough to run at home right as DRAM prices doubled and the memory supply chain pivoted to datacenter HBM.

---

## Key Quotes

> V4-Pro reportedly needs about 75% fewer per-token inference FLOPs and roughly 90% less KV cache than V3.2 at million-token context — the difference between long context being a demo and being buildable.

The performance claim is specific enough to be useful. A 90% KV cache reduction on a million-token context is the difference between "technically possible" and "you can actually ship a product with this." The indexer running on a separate CUDA stream so latency is hidden is the kind of detail that tells you someone actually read the architecture, not just the press release.

> MoE is cheap on compute and bandwidth but very heavy on capacity — almost by accident, something that makes the 'unified memory' devices a very good fit.

This is the article's most useful hardware insight. MoE's asymmetry (all experts in RAM, few activated per token) maps perfectly onto Apple Silicon's architecture. You don't need the bandwidth to feed all 744B parameters through the GPU at once — you need the capacity to hold them and the bandwidth to route through the active subset. This framing makes Apple Silicon feel less like a compromise and more like the natural architecture for sparse models.

> "Normal" autoregressive text generation is memory-bandwidth-bound — the hardware is sitting idle waiting for the weights, not waiting for the compute.

The bandwidth-compute distinction is the article's best pedagogical contribution. Most discussions of inference speed focus on FLOPs and confuse people. Coles clarifies: the bottleneck isn't compute, it's waiting for weights to arrive from memory. This makes speculative decoding (MTP) intuitive rather than mysterious — if the hardware is idle anyway, you might as well have it guess ahead.

> The models are finally good enough to run at home right as the box to run them on got expensive.

The structural irony that makes local inference a rich-person's game for now. PC DRAM passed 100% price increase quarter-on-quarter in early 2026; SK Hynix sold out next year's capacity. The supply crunch isn't a blip — it's a structural shift as wafer allocation tilts toward HBM for datacenters. Relief not expected before late 2027.

> Sparse attention, MoE routing, latent KV compression, multi-token prediction, and FP4 quantization — every one of these is a published paper and a merged commit, not a trade secret.

The article's thesis, stated plainly. The models being good is nice, but the methods staying open is what gives the rest of us options. This echoes antirez's experience with [[DS4 (DwarfStar 4)]] — you can build a competitive inference engine against one model in a week because the architecture is documented and the weights are downloadable.

---

## Key Themes

- #concept **Active vs. total parameters** — With MoE dominance, the parameter count that matters for inference speed is 3–32B (active), not 27B–1T (total). Total params determine RAM requirements; active params determine tokens/second.
- #concept **Sparse attention as the long-context unlock** — DeepSeek's lightning indexer + sliding window makes million-token context buildable rather than demo-ware. The separate CUDA stream trick hides indexer latency.
- #concept **KV cache as the hidden memory hog** — For reasoning models emitting 20K+ chain-of-thought tokens, the KV cache often dominates memory over the weights themselves. Compressed attention + compressed cache together move the wall.
- #concept **Bandwidth-bound, not compute-bound** — Autoregressive generation is memory-bandwidth-bound. MTP exploits idle hardware by guessing ahead. This reframes the entire inference optimization problem.
- #pattern **Unified memory as MoE-native architecture** — MoE's "heavy on capacity, light on bandwidth" profile makes Apple Silicon and upcoming unified-memory devices a natural fit rather than a compromise.
- #tool **RTX Spark** — Nvidia + Microsoft's upcoming 128GB unified memory device (Grace CPU + Blackwell GPU), shipping fall 2026. Pitched for running 120B-param models at million-token context.
- #concept **FP4 quantization goes production** — NVFP4 and MXFP4 formats, Blackwell hardware support, quantization-aware training. For large models, "a sensible default rather than a compromise."

---

## Critical Analysis

**What this article does well:** Coles is an unusually clear technical communicator. The bandwidth-vs-compute distinction, the MoE-unified-memory fit, and the separate-CUDA-stream detail are the kind of specifics that tell you the author actually understands the material rather than summarizing press releases. The article is structured as a coherent argument — five engineering techniques, one hardware crisis, a market survey — rather than a laundry list. The "published paper and merged commit, not a trade secret" line is the right thesis for why local inference matters beyond cost savings.

**What's missing:** The article has zero code or benchmarks the reader can reproduce. When Coles says DeepSeek V4 Pro is "level with Sonnet 4.6," there's no methodology, no task breakdown, no link to the raw data. The Artificial Analysis data is summarized in a chart image — you can't copy the numbers. For an article arguing that local models are competitive, the absence of reproducible evidence is a genuine weakness.

The hardware recommendations section is the weakest part. "Used RTX 3090" and "RTX 5090 if you have the $$$" is advice anyone could give. The RTX Spark discussion is forward-looking but admits we don't have real bandwidth figures or tokens-per-second numbers. The AMD Strix Halo mention is a single bullet with no comparative analysis against Apple Silicon.

The article also doesn't engage with the failure modes of local models. What happens when a local MoE model hallucinates on a coding task? How do you verify output quality without a cloud model to cross-check? The "good enough" thesis is compelling but the article doesn't explore what "not good enough" looks like in practice.

**Compared to related wiki coverage:** [[Recent Developments in LLM Architectures]] covers the same architectural advances (KV compression, attention budgeting, MoE variants) from a researcher's perspective — Raschka's piece is the deep-dive; Coles's is the practitioner's survey. [[How Far Behind Are Open Models]] provides the rigorous gap measurement (8–10 months on private benchmarks) that Coles gestures at with a single chart. [[Open models lag state-of-the-art closed models by 4 months]] is the specific Epoch AI analysis Coles cites. [[Self-Hosted LLMs]] is the calculator for "will this model fit on my hardware?" — the practical companion to Coles's hardware discussion. [[Datacenter GPU in a Gaming PC]] is Molnar's field report doing exactly what Coles recommends: running open models on commodity hardware. [[Muse Glimmer]] is the lab-scale confirmation: Meta distilled its closed frontier reasoning model into an open 30B that ships under 20 GB with a DFlash drafter — every one of Coles's five techniques (quantization, multi-token prediction, distillation) assembled by the model maker and handed over under Apache 2.0.

**The sharp take:** The "models are finally good enough right as the box got expensive" irony is a genuine structural observation, not just a clever line. The wafer allocation shift toward HBM is a multi-year trend, not a quarterly blip. If the memory supply crunch persists through late 2027 as Coles reports, the local inference story shifts from "which model?" to "can you afford the hardware?" — and that makes Apple's unified-memory advantage a moat, not just a nice-to-have.

---

## See Also

- [[Recent Developments in LLM Architectures]] — Raschka's deep-dive on the same architectural advances (KV sharing, attention budgeting, compressed attention)
- [[How Far Behind Are Open Models]] — Ihle's rigorous gap analysis: 8–10 months on private benchmarks, wider than public scores suggest
- [[Open models lag state-of-the-art closed models by 4 months]] — Epoch AI's specific measurement Coles cites
- [[Self-Hosted LLMs]] — Hardware calculator: which models fit on which GPUs, what tokens/sec to expect
- [[Datacenter GPU in a Gaming PC]] — Molnar's field report: £200 V100 running Qwen3.6-27B at 32 tok/s
- [[DS4 (DwarfStar 4)]] — antirez's inference engine for DeepSeek V4: asymmetric quantization, disk KV cache, Metal/CUDA backends
- [[KV Cache Locality]] — Prefix-aware routing flips cache hit rate from 12.5% to 97.5%; the infrastructure side of the KV problem
- [[MiMo-V2.5-Pro-UltraSpeed]] — Xiaomi's 1T MoE model at 1000+ tok/s via FP4 + speculative decoding + persistent kernels
- [[FreeToken — Edge-Native MoE Serving]] — the systems layer that operationalises Coles's bandwidth insight: edge-native MoE serving that continuously remaps expert residency and CPU–GPU execution to each machine's actual resource balance, claiming a 753B GLM-5.2 on a single workstation GPU
- [[JetBrains Mellum2]] — 12B MoE coding model with MTP head for speculative decoding; "focal model" concept
- [[Cohere North Mini Code]] — 30B MoE (3B active) agentic coding model on a single H100
- [[Bonsai 27B]] — PrismML takes quantization to its logical extreme: ternary/binary Qwen 3.6 27B at 3.9 GB, the first 27B-class model to fit on a phone. Retains ~90% of baseline quality with no FP16 escape hatches anywhere in the network
- [[Qwopus3.6-27B-v2]] — Jackrong's community fine-tune of Qwen3.6-27B: Trace Inversion reconstructs Claude-4.7-Max's reasoning from compressed outputs via a surrogate inverter model, then trains through a three-stage curriculum. Adds a distillation dimension to the local-inference story — you're not just running the base model, you're running a model trained on reconstructed frontier reasoning traces
- [[Poolside Laguna S 2.1]] — The latest data point in the MoE trend: 118B total / 8B active, runs on a single DGX Spark, beats DeepSeek-V4-Pro-Max (1.6T) on DeepSWE by 4.5× with 1/200th the active parameters. Poolside's "behavior over intelligence" framing — that RL post-training is persistence engineering, not capability injection — adds a new dimension to the local-inference story beyond pure architecture
- [[Local and Open Source Inference]] — Hub page: voice is solved, documents are close, reasoning still needs cloud
- [[Subquadratic 12M Context Window]] — Unverified sparse attention claim in the same problem space
- [[GPU-Free AI Datacenters]] — The networking problem created by distributed training synchronization
- [[Muse Spark]] — What the closed frontier is doing while open models chase: Meta's first proprietary reasoning model
- [[2025 in LLMs]] — Simon Willison's annual landscape survey

---

*Source: [coles.codes](https://coles.codes/posts/local-models-mid-2026), Matt Coles, 2026-06-12. Fetched 2026-06-15.*
