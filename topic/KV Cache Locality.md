# KV Cache Locality

Misdirected LLM requests waste 20–40% of GPU compute on redundant prefill. When a load balancer routes a request to a GPU that doesn't hold the relevant KV cache, the system recomputes work that already exists on a different card — and standard round-robin routing achieves just 12.5% cache hit rate on 8 GPUs. Prefix-aware routing flips this to 97.5%, delivering 22% more throughput and 85% lower P99 latency with no additional hardware.

---

## Key Quotes

> Your load balancer doesn't know. It can't know. It's counting connections, not tokens.

The central insight, stated bluntly. Standard load balancing (round-robin, least-connections) operates at the wrong level of abstraction for LLM serving. Connections are cheap; prefill is expensive.

> KV cache locality is not a tuning knob. It's a multiplier on your existing hardware.

This reframes the problem from optimization to architecture. You're not tweaking — you're leaving a 1.2–1.4x multiplier on the table. The comparison to a hardware multiplier is deliberate: this isn't a software tradeoff, it's physics.

> With round-robin, nearly 9 in 10 requests need full prefill, so the queue is perpetually full of expensive work.

This is the mechanism behind the P99 improvement, not just the average. Cache misses don't just slow individual requests — they clog the queue for everyone. Fixing locality drains the queue, benefiting even the remaining cache misses.

> "Balanced" and "efficient" are not the same thing.

The slogan version of the entire argument. Load balancers optimize for balance; inference serving needs locality. These goals conflict, and most deployments haven't noticed.

---

## Key Themes

- #concept **KV cache locality** — Per-GPU KV caches mean routing IS performance. A request landing on the wrong GPU pays the full prefill cost as if the cache didn't exist.
- #concept **Prefix-aware routing** — Consistent hashing on token prefixes ensures repeated system prompts, conversation histories, and RAG contexts hit the same GPU. Even at 50% prefix sharing, hit rates reach 91%.
- #pattern **The P99 story** — Cache misses under load create queue pileup. Fixing locality improves tail latency (85%) more than average throughput (22%) because it drains the queue.
- #pattern **Load-aware fallback** — Strict affinity creates GPU hot spots when one system prompt dominates. Diverting requests when in-flight count exceeds 2x median trades 5 points of cache hit rate for 45% better P99.
- #tool **vLLM Prometheus metrics** — `vllm:gpu_prefix_cache_hit_rate` is the diagnostic. A P99/P50 TTFT ratio above 5x signals cache thrashing.
- #concept **The 8B floor and 70B ceiling** — Below 8B parameters, routing overhead eats prefill savings. Above 70B, GPUs are compute-saturated so aggregate throughput doesn't improve — but per-request latency drops 44%, which users feel.

---

## Critical Analysis

This is an excellent piece of infrastructure pragmatism — the kind of finding that's obvious once stated but invisible to most operators because load balancers and inference engines live in different mental compartments. The benchmarks are thorough and the failure modes are honestly reported (the 8B and 70B null results, the load-imbalance tradeoff).

The article's weakness is that it treats prefix-aware routing as a binary — either you have it or you don't — without exploring the middle ground. A simple sticky-session approach (hash on session ID) would capture some of this benefit with zero infrastructure changes. The consistent-hashing fallback for unseen prefixes is clever but only works when the system prompt is the dominant shared prefix, which is the common case but not the universal one. For applications with diverse system prompts, a content-based sharding approach (e.g., locality-sensitive hashing on prompt embeddings) might work better, but the article doesn't mention it.

The most important unstated implication: **this makes the case for smaller GPU fleets.** If prefix-aware routing gets you 22% more throughput per GPU, you can serve the same load with fewer GPUs — or serve more load with the same GPUs. That's a recurring cost reduction, not a one-time optimization. The $1,200–$1,800/month figure for an 8-GPU node understates the case because it only counts wasted prefill, not the throughput you *could* be getting from those cycles.

The load-imbalance fix (divert when in-flight > 2x median) is pragmatic but hand-tuned. A dynamic approach based on queue depth or estimated prefill cost would likely outperform a static threshold — the article acknowledges the tradeoff is real but doesn't explore whether a smarter threshold exists.

Ranvier itself (the open-source project behind this post) is ambiguous from the article alone — it appears to be a load balancer or proxy that implements prefix-aware routing for LLM serving, but the post doesn't describe its architecture. The follow-up on tokenizing 50K req/s suggests it operates at the request level, parsing tokens to make routing decisions — which would explain the ~10ms overhead figure.

---

## Related

- [[Smart Models Dumb Pipes]] — The load balancer as "dumb pipe" is exactly the problem: counting connections when it should be counting tokens
- [[DS4 (DwarfStar 4)]] — Concrete KV cache implementation with three-tier design (raw→compressed→disk), showing what's at stake per GPU
- [[Local and Open Source Inference]] — The infrastructure layer where these routing decisions play out
- [[Self-Hosted LLMs]] — GPU sizing calculators; this article argues utilization matters as much as capacity
- [[Distributed Systems]] — Consistent hashing, routing, and the load-balancing-vs-locality tension
- [[Dapper Performance Trap]] — Same genre: a hidden variable (NVARCHAR casts, KV cache locality) silently destroying performance at scale
- [[PgDog]] — Connection pooling and load balancing for Postgres; the same "routing matters" insight in a different domain
- [[Model Routing Is Simple Until It Isn't]] — IBM Research's empirical proof that cache economics dominate per-token pricing in agent workloads, validating the thesis that caching is the dominant cost variable in LLM systems
- [[In-House LLM Serving at Netflix]] — Netflix's production confirmation from the other side: Triton's built-in metrics bridge surfaces only 9 of 40+ vLLM metrics, hiding KV cache hit rates and prefix cache hit rates — the exact metrics this article argues are existential

---
*Sources: [[summary/kv-cache-locality]]*
*Last updated: 2026-07-18*
