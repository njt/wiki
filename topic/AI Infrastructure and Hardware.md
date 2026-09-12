# AI Infrastructure and Hardware

The wiki's evidence on the physical layer converges on one claim: AI serving is a memory-bandwidth problem, not a compute problem. A modern accelerator can crunch bytes hundreds of times faster than it can load them, so the price of a token is set by how many bytes must move per token and how much of that movement is wasted. The consequence is that compute cores sit idle the overwhelming majority of the time, and nearly everything the field does below the model layer — bigger memory, prefix-aware routing, speculative decoding, even onsite gas turbines — is an attempt to move fewer bytes, or to keep expensive silicon from sitting idle. If decode were compute-bound, or if memory bandwidth stopped being the scarce input, most of this corpus would be beside the point.

---

## The Argument

### Decode is a memory-bound problem, and everything follows from that

[[Theoretical LLM Inference Bottlenecks]] states the accounting identity that governs the whole theme: execution time is bounded by the maximum of compute, memory, and communication time. On an H100 the "ridge point" sits at roughly 295 FLOPs/byte. Prefill, at ~2,048 FLOPs/byte for a 2K prompt, is comfortably above it and saturates tensor cores. Decode is the opposite pathology: roughly 1 FLOP/byte at batch size one, two-and-a-half orders of magnitude below the ridge. A 70B model in fp16 needs ~140GB of weights read per token, so an H100's 3.35 TB/s caps single-stream decode at ~24 tokens/sec per GPU — "and that's ignoring KV cache reads… You could double the TFLOPS and gain exactly nothing at batch 1."

[[Inference Cost Napkin Math]] reaches the same place from the dollar side. A B200 "can crunch bytes 562 times faster than it can load them," which sets a theoretical ceiling of ~331 concurrent users; PagedAttention and real-world idle stretch that to ~40–60 per chip (or ~300–800 counting user reading time). The punchline is stark: "the compute cores are idle 98% of the time," and the cost works out to roughly $9.36 per user per month at rental rates. [[The Tokens You Can't Wait For]] names the operational consequence: batching is the fix — read the weights once, compute many requests at once — but latency-bound work such as code completion and real-time document parsing cannot wait to fill a batch, "leaving expensive hardware severely underutilized."

These three sources do not conflict; they triangulate. One supplies the physics, one the economics, one the operational failure mode. Together they establish that inference cost is a bandwidth and scheduling problem rather than a raw-throughput problem.

### The chip response: memory first, and the power bill follows

[[NVIDIA B300 vs H200 GPU Analysis]] shows NVIDIA's answer is memory leadership, not FLOPS alone. The Blackwell Ultra B300 doubles the H200's memory to 288GB HBM3e and holds 8 TB/s bandwidth; the defining selling point is that a single card can host a 70B model at FP16 "while still leaving over 100GB available for KV Cache." Compute rises to 7,000 FP8 TFLOPS, but the cost lands elsewhere: 1,400W per card, mandatory direct liquid cooling, and an 8-GPU box drawing ~14kW. The article itself frames this as something enterprises may prefer to outsource — "delegating power and thermal challenges to the cloud provider." The "8–20×" uplift figures carry the usual vendor discount (the byline is a marketing team), but the memory arithmetic is the part that matters and it matches the bandwidth thesis.

[[Self-Hosted LLMs]] is the practical inverse, and a secondary source here: a memory calculator that maps concrete GPUs to which models fit and how many concurrent requests they can hold, by dividing available VRAM by KV-cache-per-request. It grounds the same arithmetic in hardware a practitioner actually owns, and it records the caveat that multi-GPU scaling decays to ~65% efficiency at 8 GPUs once communication overhead is counted.

### The CUDA moat is eroding, but slowly

[[Performance per dollar is getting faster and cheaper]] is the strongest counter-narrative to NVIDIA's framing. Wafer's team ran GLM-5.2 on AMD's MI355X and reported roughly 80% of B200 throughput while costing about 2.75× less per GPU than the B300, using MXFP4 quantisation they describe as "essentially lossless" against FP8. The telling detail is what it took: no custom kernel writes, "only framework-level bug fixes" — a single `#ifdef USE_ROCM` guard and a module-prefix mismatch. The conclusion that "SOTA on AMD is becoming more a matter of support, not software" and "the CUDA moat is eroding in real time" is a claim about friction, not silicon. Set against [[NVIDIA B300 vs H200 GPU Analysis]]'s marketing, the two bracket a real contest: NVIDIA holds the memory-and-ecosystem high ground; AMD is closing the price-performance gap, and the remaining obstacle is engineering effort rather than hardware capability.

### Serving is where the waste is — and where the wins are

If decode is bandwidth-bound, the largest recoverable loss is recomputation. [[KV Cache Locality]] demonstrates it with a single benchmark: on an 8-GPU node, round-robin routing achieves a 12.5% KV-cache hit rate, while prefix-aware routing reaches 97.5%, yielding a 22.3% throughput gain and cutting P99 time-to-first-token from 6,800ms to 1,000ms. The mechanism is that KV caches are per-GPU, and "your load balancer doesn't know. It can't know. It's counting connections, not tokens." The framing that KV-cache locality "is not a tuning knob. It's a multiplier on your existing hardware" lands exactly on the theme: the same silicon is worth a fifth more or less depending on how traffic is routed.

[[In-House LLM Serving at Netflix]] shows a mature operator making the same class of decision at production scale: vLLM over TensorRT-LLM, chosen for iteration speed and debuggability rather than raw performance; Triton as the model-scoring substrate; Red-Black deploys to keep rollback cheap; and a constrained-decoding pipeline that had to be rewritten from a per-request Python loop (GIL-serialised) into a C++ batch-level logits processor to keep tail latency flat as batch size grew. The note that Triton's metrics bridge "surfaced only 9 of 40+ vLLM metrics" is a small, honest signal of how much of this stack remains hand-assembled.

[[MiMo-V2.5-Pro-UltraSpeed]] is the extreme end of the same curve: Xiaomi and TileRT claim 1,000+ tokens/sec decode on a one-trillion-parameter model from a single 8-GPU commodity node, via FP4 quantisation of MoE experts, a block-parallel speculative decoding method (DFlash), and TileRT's persistent-kernel execution model. It matters to the theme less as a speed record than as proof of model-system codesign: the algorithmic choices and the system choices were made together, not layered. The honesty is in the release mechanics — 3× the price for ~10× the speed, rationed through an application window "due to limited high-speed inference resources" — a reminder that extreme speed is currently scarce and expensive to supply.

[[The Tokens You Can't Wait For]] offers a platform-level alternative for one quadrant of the problem: for low-value, decode-heavy, latency-bound work, replace autoregressive generation with text diffusion, which is compute-bound even at batch size one. Inception Labs' Mercury reported over 1,100 tokens/sec on H100s. The trade is explicit — diffusion does "more total work per useful token" and lands at 85–95% of strong autoregressive baselines — which is why the authors frame it as a routing rule rather than a replacement.

One further claim is worth holding apart: Modal's survey of coding-agent platforms argues that general-purpose cloud is the wrong substrate for agent workloads, and its most telling observation is that "CPU-based execution is the primary sandbox workload," with GPU "on-demand when needed, not the default." That reframes much of the above: a large share of agent traffic may never touch a GPU at all, which is why the sandbox layer is CPU-centred.

### Energy is now the binding constraint

[[How AI Labs Are Solving the Power Crisis]] moves the bottleneck up a layer. SemiAnalysis projects AI power demand growing from ~3GW in 2023 to 28GW+ by 2026, against interconnection queues that stretch to five years and a speculative-request prisoner's dilemma clogging the queue. At $10–12B of AI cloud revenue per GW per year, six months of earlier grid access is worth billions — which is why xAI's Colossus deployed 500MW+ of truck-mounted turbines and bypassed the grid entirely. The supply chain is the real story: gas turbines carry 12–36 month lead times, blade casting is concentrated in four firms still recovering from an aerospace bust, and the alloys depend on rhenium, cobalt and tantalum. The energy problem is as much about castings and interconnection paperwork as about GPUs.

The model labs are responding by fronting power and site capacity directly. [[Muse Spark]] mentions Meta's Hyperion data centre as the strategic investment underpinning a ground-up rebuild of its AI stack — and separately claims its new pretraining regime reaches "the same capabilities with over an order of magnitude less compute" than Llama 4 Maverick, a compute-efficiency claim that belongs to this theme even though the post is mostly about the model. ([[Muse Spark and the Rough Edges Admission]] covers the Apollo "evaluation awareness" finding, which sits outside infrastructure.)

### Is the GPU the right substrate at all?

[[GPU-Free AI Datacenters]] is the outlier that makes the theme's hidden premise explicit. Almartis's argument is that all of the above — InfiniBand, Ultra Ethernet, packet spraying, rail-optimised topologies — is "downstream of the computational assumptions the models themselves impose," namely that thousands of GPUs must synchronise. Remove that assumption, via "associative memory systems built around explicit, addressable, and deterministic memory structures," and you can build a "GPU-free, non-blocking, 1-tier full mesh" cluster. The networking history it recounts — RoCEv2's sensitivity to packet loss, Priority Flow Control's head-of-line blocking, the InfiniBand-versus-Ultra-Ethernet contest — is a useful map of why AI networking exists at all, even though its own proposal is unverified.

## Where the Sources Disagree

**GPUs: substrate or detour.** [[GPU-Free AI Datacenters]] claims the complexity is accidental and eliminable, and that a "150-kW cluster can train a system from scratch to common sense." Every other source treats GPUs as the substrate to be optimised rather than removed: [[MiMo-V2.5-Pro-UltraSpeed]] pushes commodity GPUs to 1,000 tokens/sec, [[NVIDIA B300 vs H200 GPU Analysis]] assumes continued GPU scaling, and [[The Tokens You Can't Wait For]] proposes a different model on the same GPUs. Almartis is a vendor blog making an extraordinary, unverified claim, so the burden of proof sits with it — but the disagreement is worth stating because it makes visible an assumption the rest of the corpus never questions.

**How to fix single-stream latency: speculative decoding or diffusion.** [[MiMo-V2.5-Pro-UltraSpeed]]'s DFlash keeps autoregressive generation lossless via rejection sampling and attacks the sequential dependency from within; [[The Tokens You Can't Wait For]] argues that for low-value, decode-heavy traffic this is the wrong fight, and one should switch paradigm to diffusion and accept 85–95% quality. [[Theoretical LLM Inference Bottlenecks]] sharpens the difference: speculative decoding has an Amdahl ceiling set by the draft model's acceptance rate, while diffusion is compute-bound at batch one but does more total work. The two sources do not refute each other; they disagree about which cost — accuracy or compute — is cheaper to spend.

**Whether the CUDA moat is actually breaking.** [[Performance per dollar is getting faster and cheaper]] says "the CUDA moat is eroding in real time," citing a port that required no custom kernels; [[NVIDIA B300 vs H200 GPU Analysis]] implies NVIDIA's memory and ecosystem leadership is intact. The measurable middle — AMD at ~80% of B200 throughput for about 2.75× less money, at the price of engineering effort — is the only part of this dispute the evidence actually settles.

## What's Missing

- No neutral benchmarks. The chip numbers come from AMD's and NVIDIA's respective vendor corners; nothing in the evidence is an independent third-party measurement.
- Training is nearly absent. The serving and economics sources all cover inference; only [[GPU-Free AI Datacenters]] addresses training's networking, and no source examines training-scale compute economics.
- The energy economics are truncated — the TCO analysis in [[How AI Labs Are Solving the Power Crisis]] sits behind the paywall, so the "bring your own power" cost comparison is not in the evidence.
- Cloud platforms are asserted against rather than examined: Modal's claim that AWS/GCP/Azure is the wrong substrate for agent workloads is stated, not tested.
- The evidence is directional on the biggest numbers: it cannot say how much of the projected 28GW of demand has actually materialised, or what aggregate supply of GPUs and gas turbines exists.

## Also on This Theme

- [[Tesla V2L Discharger]] — a portable 5kW vehicle-to-load adapter; peripheral to the theme, relevant only as a footnote to how far the "energy" filing reaches.
- [[USB Type-C and Power Delivery Architecture]] — TI's USB-C and USB Power Delivery app note (240W Extended Power Range); power delivery at the device edge, outside the data-centre scope.
- [[Well-Read Students Learn Better]] — a 2019 arXiv paper showing pre-trained compact models are competitive with compression; the earliest evidence here that smaller, cheaper-to-serve models are viable.

---

*Compiled from 16 sources: [[summary/almartis-gpu-free-datacenter]], [[summary/best-infrastructure-platforms-coding-agents]], [[summary/glm52-amd]], [[summary/how-ai-labs-are-solving-the-power-crisis]], [[summary/in-house-llm-serving-at-netflix]], [[summary/kv-cache-locality]], [[summary/mimo-tilert-1000tps]], [[summary/muse-spark]], [[summary/napkin-inference-cost]], [[summary/nvidia-b300-vs-h200-gpu-specs-performance-analysis]], [[summary/self-hosted-llms]], [[summary/tesla-v2l-discharger]], [[summary/theoretical-llm-inference-bottlenecks]], [[summary/ti-usb-c-pd-ebook]], [[summary/tokens-you-cant-wait-for]], [[summary/well-read-students-learn-better]]*
*Last compiled: 2026-09-12*
