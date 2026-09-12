# Datacenter GPU in a Gaming PC

Oscar Molnar's field report on cramming a used Tesla V100 SXM2 (16GB) datacenter GPU into a gaming rig alongside an RTX 4080: £200 for 32GB total VRAM, ~32 tok/s on Qwen3.6-27B at 128K context. The hardware hacking (82 dB fan → motherboard PWM header) is the entertaining part; the NixOS driver archaeology (kernel 6.6, legacy_535, CUDA 12.2 backport) is the part that matters for anyone trying this themselves. The conclusion — that a bedroom GPU running Qwen3.6-27B now sits in the same conversation as Sonnet 4.6 — is provocative and probably directionally correct for certain benchmarks, but the gap on actual coding tasks (SWE-Bench) tells a more qualified story.

---

## Key Quotes

> "the model you run in your bedroom is in the same conversation as the ones that charge you per token"

This is the money quote and the one that'll get cited. Molnar backs it with specific benchmark comparisons: Qwen3.6-27B ties Sonnet 4.6 on Artificial Analysis's Agentic Index, beats it on MMMU-Pro and Terminal-Bench 2.0, trails on GPQA and SWE-Bench Verified. The framing is careful — "same conversation," not "equal to." That's the right hedge. But it's worth noting the benchmark selection: Agentic Index and Terminal-Bench are the two where local models have been catching up fastest. SWE-Bench Verified is where the gap remains wide. The model you run in your bedroom is in the same conversation *on the benchmarks that favor it*.

> "it never goes above 50C even at full load"

At 10% PWM. After replacing the 82 dB stock fan with a £2 motherboard adapter cable. This is the hardware hacker's version of "and it works perfectly." The thermal headroom on the V100's heatsink is absurd — designed for 1U server racks with forced airflow, now sitting in open air with barely any fan speed. The practical takeaway: datacenter GPUs are over-cooled for desktop use, and that's a feature, not a bug. You can run them near-silent.

> "services.xserver.enable = true;"

An entire line item in a NixOS config, carrying the weight of a driver quirk that would take hours to debug if you didn't know about it. NVIDIA kernel modules won't load on NixOS without X server enabled — even on a headless system. This is the kind of institutional knowledge that blog posts preserve better than documentation. The whole NixOS section (kernel 6.6 pin, legacy_535, CUDA 12.2 backport) is a survival guide for anyone who buys a Volta card and discovers NVIDIA dropped support in driver branch 560.

> "the 7900 XTX costs £700+ and ROCm support for LLM inference is still rough compared to CUDA"

The AMD tax, quantified. The 7900 XTX has more raw bandwidth (960 GB/s vs. the V100's 900 GB/s) and more VRAM (24GB vs. 16GB) but costs 3.5x more and comes with software friction. CUDA's moat isn't just performance — it's that every inference framework, every quantization, every optimization targets CUDA first. ROCm support is "rough," and Molnar doesn't bother to elaborate because if you've tried both, you already know.

## Key Themes

#local-llm #hardware #homelab #nixos #gpu #cost-optimization

## Architecture & Hardware

**The build.** Tesla V100 SXM2 16GB (eBay, ~£150) + SXM2-to-PCIe adapter (~£50) + PWM fan cable (~£2). Total: ~£200. The V100 is a 2017 Volta card with 5,120 CUDA cores and 900 GB/s memory bandwidth over a 4,096-bit HBM2 bus — that bandwidth number is the key stat. At 900 GB/s, the V100 sits between the RTX 4080 (736 GB/s) and the RX 7900 XTX (960 GB/s) in raw throughput, but at a fraction of the cost. The tradeoff is 16GB VRAM — the V100 32GB variant exists but costs more.

**The adapter board.** SXM2 is NVIDIA's mezzanine connector for servers — it's not PCIe. The ~£50 adapter PCB translates SXM2 to a standard PCIe x16 slot. This is the enabler. Without it, datacenter GPUs are paperweights outside a server chassis. The adapter market is small and specific: SXM2-to-PCIe for V100, different adapters for different generations. It's a cottage industry built on eBay sellers and AliExpress listings.

**Fan hack.** The V100's cooler is designed for server chassis with high-static-pressure fans. In open air, the stock fan runs at 100% PWM (82 dB — garbage disposal territory) and can't be controlled via software. Molnar's fix: trace the pinout with a 9V battery, confirm it's a standard PWM fan on a JST PH2.0 connector, wire it to a motherboard header. At 10% PWM, the card stays under 50°C at full load. This is the kind of hardware debugging that separates "I read about this on Reddit" from "I did this and here's what happened."

**Dual-boot via USB.** NixOS runs from an NVMe drive in a USB-C enclosure. Unplug = Windows gaming PC (4080 only). Plug in + reboot = LLM inference rig (4080 + V100). Crude, but effective. The V100 occasionally disappears from `lspci` after warm reboots — ACPI enumeration bug, cold boot always fixes it. Another "you only learn this by doing it" detail.

## Software Stack

**NixOS as the OS of choice for janky hardware.** Molnar's NixOS config is a masterclass in pinning specific versions to support deprecated hardware. The constraints cascade: Volta support was dropped in NVIDIA driver branch 560 → must use branch 550 (`legacy_535`) → requires kernel 6.6 → only supports CUDA 12.2 → CUDA 12.2 must be backported from nixpkgs 24.05. Each constraint is a potential footgun, and NixOS's declarative config makes the whole chain explicit and reproducible. If he'd done this on Ubuntu, none of this would be documented — it'd be a series of `apt-mark hold` commands lost to shell history.

**The X server hack.** `services.xserver.enable = true` on a headless system because NVIDIA's kernel modules refuse to load without it. This is the kind of incantation that takes hours to discover if you don't have a blog post to crib from. It's not in NVIDIA's documentation. It's not in NixOS's documentation. It's in Oscar Molnar's blog post.

**Models over NFS.** The models live on TrueNAS, mounted via NFSv4. llama.cpp is configured to wait for the mount before starting (`_netdev`, `auto`, `nofail`). This is practical: models are large, SSDs are expensive, NAS is already there. The cost optimization extends to storage.

**Inference performance.** Qwen3.6-27B-MTP Q5_K_M (~19GB) split evenly across both GPUs (`-ts 1.0,1.0`): ~32 tok/s generation, ~133-160 tok/s prompt processing. With Multi-Token Prediction, 50-60 tok/s on code. At 32 tok/s, you're reading comfortably — it's not instant, but it's not waiting. At 50-60 tok/s on MTP-friendly output, it's faster than most cloud APIs.

## Critical Analysis

**The benchmark claim needs scrutiny.** Molnar says Qwen3.6-27B "ties with Claude Sonnet 4.6 on Artificial Analysis's Agentic Index." Artificial Analysis's Agentic Index is a composite — it aggregates across multiple benchmarks with weightings that matter. A tie on the composite doesn't mean parity on any individual task. The model trails Sonnet on GPQA and SWE-Bench Verified — the two benchmarks most relevant to the coding agents this wiki covers. For the use case this wiki cares about (agentic coding), the gap is real.

**But the trajectory is the point.** Molnar isn't claiming Qwen3.6-27B is better than Sonnet 4.6. He's claiming it's in the same conversation. That's a qualitatively different claim, and it's supported by the data. A year ago, no local model was in the same conversation as frontier cloud models on any benchmark. Now one is, for £200 in hardware. The question isn't "is local inference as good as cloud inference today" — it's "how long until the gap closes on the benchmarks that matter to me." The successor model sharpens the question: [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] argues the "~32 tok/s is slow" read is answering the wrong question — its ~33 tok/s `tg` still delivered a three-repo bug fix in ~10 minutes unattended, because wall-clock to a correct result, not tokens-per-second, is the metric a daily driver actually cares about.

**The CUDA moat is structural.** AMD's 7900 XTX has more bandwidth and more VRAM than the V100, costs £700+, and gets ruled out because ROCm is "rough." This isn't a temporary state. NVIDIA's software advantage compounds: every framework targets CUDA first, every optimization is CUDA-tested, every quantization pipeline assumes CUDA. ROCm is catching up but it's chasing a moving target. For local inference hardware, the practical choice is NVIDIA or Apple Silicon — AMD exists in theory but not in practice.

**The 16GB ceiling is real.** The V100 16GB can run Qwen3.6-27B at Q5_K_M quantization (~19GB) only because it's split across two GPUs (16GB + 16GB = 32GB). A single V100 couldn't run this model. The P40 alternative (24GB, no tensor cores) is slower. The V100 32GB variant is the sweet spot but costs more. At £200 total, the 16GB V100's constraint is acceptable. But anyone looking to run 70B+ models locally is still looking at multi-thousand-pound setups.

**The NixOS config is the real artifact.** The blog post is entertaining hardware hacking wrapped around a NixOS configuration that solves real driver compatibility problems. Anyone with a Volta GPU and a modern Linux distribution will hit the same issues. The `legacy_535` + kernel 6.6 + CUDA 12.2 combo is the key unlock. The X server workaround is the hidden gem. This config will age — eventually kernel 6.6 will be EOL, CUDA 12.2 will be ancient, and the whole stack will need a rethink. But for now, it works.

**This is the homelab spirit.** Molnar's setup is janky: USB-booted OS, eBay GPU on an adapter board, fan hacked with a £2 cable, models served over NFS from a NAS. It's not clean. It's not supported. It works. The entire project cost less than a single month of Claude Max. That's the economic reality that makes local inference compelling for enthusiasts: the hardware pays for itself in months, and you get to keep it.

**What's missing from "the same conversation."** Molnar frames Qwen3.6-27B vs. Sonnet 4.6 as a model comparison. The August 2026 launch of [[Pokee-Isaac 28B]] — a commercial, closed-source model built partially on Qwen3.6-27B that claims 10M-token context on a single RTX 4090 — shows the same base model being productized in the opposite direction: not "run it yourself for £200," but "pay us $0.15/M input for our secret sauce." The existence of both paths from the same Qwen base validates Molnar's thesis that these models are in the conversation, while the closed/open split shows the conversation has more than one answer. But the conversation isn't just about model quality — it's about the entire stack. Sonnet comes with tool use, streaming, structured outputs, prompt caching, and an API that 100,000 developers know. Qwen3.6-27B comes with... llama.cpp flags. The model might be competitive, but the developer experience is not. For the audience of this wiki — people building agentic systems — that gap matters as much as benchmark scores.

---

## Related Pages

- [[Local and Open Source Inference]] — Hub page: what's solved (voice, docs), what's not (reasoning), the cost/convenience tradeoff
- [[Self-Hosted LLMs]] — GPU memory calculator and inference performance estimator; the "what can my hardware run?" companion
- [[A Few Words on DS4]] — antirez crosses the local inference threshold: using a local model instead of Claude/GPT for serious work
- [[DS4 (DwarfStar 4)]] — The C inference engine: 65K lines, Metal/CUDA, GGUF, disk KV cache
- [[QMD]] — Tobi Lutke's local-first CLI search engine; same "run it on your own hardware" ethos
- [[GPU-Free AI Datacenters]] — The opposite approach: associative memory instead of GPU clusters
- [[Step 3.7 Flash]] — 196B model at 1/9th the cost of Opus 4.6; the cloud-side of the cost-performance equation
- [[Honey I Shrunk the Coding Agent]] — 9B local model jumps from 19% to 46% by redesigning the harness; empirical proof that the harness matters more than the model
- [[MiniMax Models]] — Full model lineup with local MLX deployment option
- [[Granite 4.1]] — IBM's open-source family; another data point in the local inference landscape
- [[Qwen3.8 27B Hardware Tests]] — Hardware Corner's benchmark of the Qwen 3.6 successor: same VRAM-first profile (a 24 GB card tops out at 64k context), RTX 3090 as the value pick, and the finding that Qwen3.8 is no faster than Qwen3.6 — Molnar's "same conversation" claim extends to the next point-release without changing the hardware story.

---

*Sources: [[summary/v100-local-llm]]*
*Last updated: 2026-06-02*
