# Inference Cost Napkin Math

A worked example showing how to estimate LLM serving costs from first principles: GPU specs, model architecture, and basic matrix math. The key insight is that inference is memory-bandwidth-bound — B200 compute cores sit idle 98% of the time — so cost optimization is about matching bandwidth to batch size, not throwing more FLOPs at the problem.

---

## The Central Tension

The B200 has 8 TB/s memory bandwidth and 4500 TFLOP/s compute. That means it "can crunch bytes **562 times faster** than it can load them." Every inference dollar you spend is overwhelmingly paying for memory movement, not computation.

This isn't a B200-specific quirk — it's the defining constraint of transformer inference. The compute-to-memory ratio determines everything downstream: batch size, concurrency, cost per user.

## Key Quotes

> "the compute cores are idle 98% of the time"

This is the number that should reshape how you think about GPU utilization. High GPU "utilization" in monitoring dashboards typically measures compute — but for inference, memory bandwidth is the real bottleneck. A GPU at 100% compute utilization might still be wasting most of its potential if memory isn't saturated.

> "Inference engines will cache the K,V pairs for reuse"

The KV-cache isn't an optimization — it's existential. Without it, a single 200k-context token requires 26 trillion FLOPs and 1.7 billion memory accesses. The cache collapses this by three orders of magnitude. This is why [[KV Cache Locality]] matters so much: if your load balancer sends a returning user to the wrong GPU, you eat the full prefill cost again.

> "For non-chat apps, measuring duty-cycles is not optional"

The author's ~300-800 users per B200 assumes 80% idle time (users reading). Agentic loops, code generation, and continuous reasoning invert this — they're high-duty-cycle workloads where the user is never idle. If you price for chat but ship for agents, your margin disappears.

## The Math, Step by Step

### 1. Matrix multiplication cost

For A(N×d) × B(d×M): 2NMd memory accesses, 2NMd FLOPs. With tiling, memory drops to ~d(N+M). This ratio — ops per byte loaded — is the fundamental lever.

### 2. KV-cache changes the game

Without cache, one 200k-token forward pass: 26 trillion FLOPs, 1.7 billion memory accesses. With cache: ~52.4M ops, ~26.2M memory accesses. The ratio becomes 2×B operations per memory access, where B is batch size.

### 3. Solve for B

Memory bandwidth (8 TB/s) ÷ FP8 bytes per op = 562× imbalance. Set 2B = 562 → ideal batch size is **~331 concurrent users** — the theoretical ceiling before compute becomes the bottleneck.

### 4. VRAM brings it back down

A 32B model = 32GB weights. KV-cache for 200k context = 210GB without GQA, ~26GB with GQA. With 160GB remaining: **~6 concurrent users** at full context.

### 5. PagedAttention buys headroom

Incremental allocation + flushing cold sessions → 40-60 users per chip. Add 80% idle time → 300-800 users.

### 6. Dollar cost

- Owned B200 ($40K): $133/user lifetime at 300 users
- Rented B200 ($4/hr): $0.013/user, **$9.36/month** per user

The author calls this conservative. For agentic workloads with near-100% duty cycles, divide by 5.

## Key Themes

- **#concept** Memory-bandwidth-bound inference — compute is cheap, memory movement is expensive
- **#concept** KV-cache as the critical economic variable — cache hit rate IS your margin
- **#concept** Duty cycle as the hidden multiplier — chat pricing assumes 80% idle; agents don't idle
- **#tool** Napkin math as a bullshit detector — you can sanity-check vendor claims with high-school algebra
- **#pattern** Batching as bandwidth-compute balancing — the ideal batch size falls out of the GPU spec sheet

## Critical Analysis

**The napkin math is the real product.** The specific numbers (331 theoretical users, $9.36/month) will drift as hardware and architectures change. What's durable is the method: start with the memory bandwidth ÷ compute ratio, solve for batch size, then constrain by VRAM. Anyone building or buying inference infrastructure should be able to reproduce this derivation in five minutes. If you can't, you're trusting vendors to price honestly — and they won't.

**The 98% idle compute number deserves more attention.** It means GPU utilization metrics are actively misleading for inference workloads. If your ops team is optimizing for high GPU utilization, they're optimizing the wrong thing. The right metric is tokens-per-second-per-dollar, which maps to memory bandwidth saturation, not compute utilization. This is the same insight behind [[Smart Models Dumb Pipes]]: judgment (compute) is abundant; execution (memory movement) is the bottleneck.

**Duty cycle is the elephant in the room.** The difference between 300 users (80% idle) and 60 users (0% idle) is 5x — and that's the difference between a viable business and a money furnace. Every AI startup that builds on top of LLM APIs is implicitly betting on user idle time. Agentic loops, code generation, and "thinking" models all eat that margin. The author is right that "measuring duty-cycles is not optional" — but most teams aren't measuring them at all.

**The convergence with local inference economics is striking.** [[Datacenter GPU in a Gaming PC]] found that a GBP 200 V100 pays for itself in months against cloud pricing. This article finds a B200 serving 300 users costs $133/user lifetime. The crossover point where local beats cloud keeps moving downmarket. As [[Local Models in Mid-2026]] notes, the models are finally good enough to run at home — and this napkin math explains why the economics work even at small scale.

**Multi-GPU breaks the napkin.** The author is honest that multi-GPU setups require moving beyond back-of-envelope math. Tensor parallelism, pipeline parallelism, and cross-GPU KV-cache routing add communication overhead that doesn't appear in single-GPU arithmetic. This is where [[KV Cache Locality]] becomes critical — prefix-aware routing on multi-GPU fleets can recover 22% throughput with zero hardware changes. The napkin math tells you the ceiling; cache-aware routing gets you closer to it.

**The architectural escape hatches are proliferating.** [[Recent Developments in LLM Architectures]] documents four different attacks on KV-cache cost: cross-layer KV sharing (Gemma 4), attention budgeting (Laguna XS.2), sequence-dimension compression (DeepSeek V4), and constrained residual streams. Each chips away at the memory-bandwidth bottleneck from the model side rather than the hardware side. The napkin math framework adapts — you just plug in the new memory-per-token numbers — but the specific ratios keep improving.

## See Also

- [[KV Cache Locality]] — Why round-robin load balancing kills your inference economics, and how prefix-aware routing fixes it
- [[Datacenter GPU in a Gaming PC]] — The same napkin math applied to a GBP 200 eBay V100
- [[Self-Hosted LLMs]] — GPU memory calculator and inference performance estimator with the same bandwidth/batch/formula approach
- [[Local Models in Mid-2026]] — Five architectural advances attacking the memory-bandwidth bottleneck
- [[Recent Developments in LLM Architectures]] — Cross-layer KV sharing, attention budgeting, and compressed attention as cost-reduction strategies
- [[MiMo-V2.5-Pro-UltraSpeed]] — What happens when you codesign model and system for extreme throughput: 1000+ tok/s
- [[DS4 (DwarfStar 4)]] — antirez's single-user inference engine that makes different tradeoffs (disk KV cache, asymmetric quantization)
- [[Smart Models Dumb Pipes]] — The load balancer as "dumb pipe" is exactly the KV-cache-locality problem
- [[Computer Use is 45x More Expensive Than Structured APIs]] — The other side of the inference-cost coin: architecture cost gaps that better models can't close
- [[The Advisor Strategy]] — Cost-efficient orchestration that composes cheap and expensive models
- [[AI Pricing]] — Per-token pricing API for cross-provider cost comparison
- [[Step 3.7 Flash]] — 97% of Opus 4.6 at 1/9th the cost via advisor-executor architecture
- [[Cohere North Mini Code]] — 30B MoE (3B active) on a single H100, sovereign-developer economics

---
*Sources: [[raw/napkin-inference-cost]]*
*Last updated: 2026-06-21*
