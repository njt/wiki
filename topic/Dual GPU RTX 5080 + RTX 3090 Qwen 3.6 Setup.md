# Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup

iMil's detailed field report on running Qwen 3.6 27B at Q8 quantization across mismatched NVIDIA GPUs (RTX 5080 Blackwell + RTX 3090 Ampere) on an Asus X570-Pro motherboard, achieving 80–91 tok/s through tensor-split speculative decoding with MTP and ngram hints. The article is a soup-to-nuts guide: BIOS configuration, kernel driver archaeology for mixed-generation GPUs, a llama.cpp build targeting both architectures, and the exact launch flags that make it work. The real contribution isn't the performance numbers — it's the documentation of every sharp edge smoothed along the way, from "CSM must be disabled or dual-GPU won't boot" to "NCCL is counterproductive, disable it."

---

## Key Quotes

> "you CAN'T boot the OS in BIOS/MBR mode"

This appears mid-way through the BIOS section but it's the article's most critical finding. If CSM is enabled, the second GPU simply doesn't work. No error message, no warning — just a card that's invisible and an afternoon lost to debugging. The entire setup turns on this single BIOS toggle. The author buries it in a paragraph without fanfare, which is exactly how these discoveries feel when you make them: obvious in hindsight, invisible from the outside.

> "-DGGML_CUDA_NCCL=OFF"

The author's commentary on this is the article's sharpest technical judgment: "I've noticed NCCL can sometimes be counterproductive even though llama-server logs may claim otherwise." Four words of technical reasoning backed by empirical results. The entire AI industry has spent years building NCCL as the standard multi-GPU communication library, and here's a single practitioner turning it off and getting better performance. The lesson isn't "NCCL is bad" — it's "measure everything, trust nothing, and don't let conventional wisdom override what your hardware tells you."

> "I also saw no visible difference between ngram-mod and draft-mtp speculative decoding, so I would recommend enabling both for a combined effect"

A pragmatic observation that cuts through the speculative decoding taxonomy. Most discussion of speculative decoding focuses on choosing the right method (MTP vs. ngram vs. EAGLE vs. Medusa). iMil's approach is simpler and more honest: try both, see if the combination helps, move on. The 77% draft acceptance rate and 80+ tok/s results suggest the combined approach works. This is engineering as practiced, not engineering as theorized.

> The draft acceptance rate is ~77%, with ngram-mod handling the repetitive/structured output and MTP handling the creative/unpredictable passages.

Not a direct quote from the article, but the pattern visible in the statistics. ngram-mod generates far fewer drafts (1,169 vs. 42,477) but with near-perfect acceptance (1,169/1,169). MTP generates the volume but at ~85% acceptance. Together they cover different failure modes: ngram catches the easy wins on structured text; MTP takes the harder shots on free-form generation.

---

## Key Themes

#tool **llama.cpp dual-GPU inference** — The full configuration surface: tensor split (`-sm tensor -ts 2,3`), multi-architecture CUDA builds, dual speculative decoding with ngram-mod + draft-mtp, and the NCCL-off discovery.

#pattern **Mixed-generation GPU setups** — Running Ampere and Blackwell cards together is explicitly unsupported by NVIDIA's open-gpu-kernel-modules. The standard `nvidia-open` driver works, but you lose p2p DMA between cards. The article documents exactly what breaks and what doesn't.

#concept **Consumer hardware as inference platform** — At ~$2,000 total GPU cost for 80+ tok/s on a 27B model at Q8 with 229K context, this setup delivers inference performance that competes with cloud APIs on raw speed. The 16GB + 24GB VRAM split is awkward (the 5080's 16GB is the bottleneck), but tensor-split strategy makes it work.

#tool **BIOS as the hidden failure domain** — CSM disabled, Above 4G Decoding enabled, ReSize BAR enabled, PCIe Gen 4 forced for both slots. Five settings, each individually documented as "you need this." The BIOS is the part of the stack nobody writes blog posts about, and it's where multi-GPU setups fail most often.

---

## Critical Analysis

**This is the article that [[Datacenter GPU in a Gaming PC]] should be read alongside.** Molnar's piece is about making it work with janky hardware (eBay V100, adapter boards, fan hacks). iMil's piece is about making it work *well* with current-generation consumer hardware. Molnar gets 32 tok/s; iMil gets 80+. Molnar uses NixOS to pin a deprecated driver branch; iMil uses the standard nvidia-open driver and documents the BIOS settings that make it possible. Together they map the full spectrum of local inference hardware from "can I even run this?" to "can I run this fast?"

**The NCCL-off finding deserves more investigation than the article gives it.** iMil says NCCL is "counterproductive" and disables it, and the results (80+ tok/s) speak for themselves. But *why* is NCCL counterproductive on a two-GPU setup with mismatched cards? The likely answer is that NCCL's overhead (peer-to-peer setup, topology detection, collective communication) costs more than the benefit for a two-card setup where tensor parallelism doesn't need cross-GPU synchronization during generation — each card independently processes its share of layers. But this is speculative. Someone should benchmark this properly.

**The tensor split ratio (2:3) is doing a lot of work.** The RTX 3090 has 24GB, the RTX 5080 has 16GB. A 27B Q8 model is roughly 27GB, and q8_0 KV cache for 229K context is another ~12GB. The 2:3 ratio (3090 gets ~60% of compute) roughly maps to the VRAM ratio (24:16 = 60:40). But the author doesn't explain how they arrived at 2:3 — trial and error? Memory profiling? The ratio determines whether the model fits and which card becomes the bottleneck. A few sentences on the tuning process would make this reproducible rather than cargo-cultable.

**The speculative decoding numbers are genuinely impressive.** 77% draft acceptance rate across 44K+ drafts is not cherry-picked. The ngram-mod's near-perfect acceptance (1,169/1,169) on 74K generated tokens suggests the model is producing highly structured output some of the time — likely code or repetitive text. The MTP acceptance (~85%) on creative passages makes the case for dual-strategy speculative decoding: no single method covers all output distributions.

**What the article doesn't cover matters too.** No power consumption numbers. No thermal data under sustained load. No discussion of whether the 5080's 16GB becomes a bottleneck at 229K context (almost certainly yes — the 3090 is using 23.6GB of 24GB while the 5080 is at 15.9GB of 16GB). No comparison to a single RTX 4090 or 5090, which would simplify the setup considerably. The article documents what worked, not what was tried and failed — which means readers will rediscover the same dead ends.

**The broader significance: consumer dual-GPU is becoming viable.** Between this article and [[Datacenter GPU in a Gaming PC]], a clear pattern emerges: running large models on consumer hardware is no longer a curiosity. It's reproducible, well-documented, and delivers performance that competes with cloud APIs. The remaining barrier isn't model quality ([[Local Models in Mid-2026]] shows the gap is closing) or inference speed (80+ tok/s is faster than most cloud APIs) — it's setup complexity. You need to understand BIOS configuration, kernel drivers, CUDA architecture codes, cmake flags, and llama.cpp's speculative decoding taxonomy just to get started. The day someone wraps all of this in a single docker-compose.yml, cloud inference gets competition.

---

## See Also

- [[Datacenter GPU in a Gaming PC]] — The companion piece: £200 V100 in a gaming rig running the same model at 32 tok/s. Read together for the full spectrum from "janky and cheap" to "current-gen and fast."
- [[Local Models in Mid-2026]] — Five engineering advances (MTP, MoE, sparse attention, KV compression, FP4) that make models like Qwen 3.6 viable at home. The MTP discussion directly explains why speculative decoding works so well here.
- [[Inference Cost Napkin Math]] — Why inference is memory-bandwidth-bound: B200 compute cores sit idle 98% of the time. iMil's setup demonstrates the same principle on consumer hardware — the dual GPU config is about aggregate memory bandwidth, not compute.
- [[Local and Open Source Inference]] — Hub page: what's solved, what's not, the cost/convenience tradeoff.
- [[Self-Hosted LLMs]] — GPU memory calculator and inference performance estimator; the "will my hardware run this?" companion.
- [[MiMo-V2.5-Pro-UltraSpeed]] — The extreme end: 1000+ tok/s on commodity GPUs via FP4 + speculative decoding + persistent kernels. What happens when the techniques iMil uses are pushed to production scale.
- [[JetBrains Mellum2]] — MTP head as dual-use speculative decoding on a 12B MoE model. The same technique iMil uses, in a different context.
- [[DS4 (DwarfStar 4)]] — antirez's inference engine making different tradeoffs (disk KV cache, single-user focus) on the same class of hardware.
- [[Qwen3.8 27B Hardware Tests]] — Hardware Corner's llama.cpp benchmark of the Qwen 3.6 successor: the same VRAM-first profile (24 GB → 64k, 32 GB → 128k), a dual RTX 5060 Ti "capacity-per-dollar" result that echoes this page's dual-GPU thesis, and the finding that Qwen3.8 is *slower* than Qwen3.6 at long context.

---

*Sources: [[summary/imil-rtx-5080-rtx-3090-qwen-setup]]*
*Last updated: 2026-07-05*
