# Recent Developments in LLM Architectures

Sebastian Raschka surveys four open-weight LLM releases from early 2026 — Gemma 4, Laguna XS.2, ZAYA1-8B, and DeepSeek V4 — tracing how each attacks the same problem from a different angle: making long-context inference cheaper without simply shrinking the model. The transformer block isn't being replaced, but it's being surgically modified at every pressure point: KV cache size, attention computation cost, residual-stream mixing, and per-layer capacity allocation.

---

## Key Quotes

> Later layers "reuse key-value states from earlier layers" instead of computing their own. Each layer still computes its own Q projection, so attention patterns remain layer-specific.

Raschka on Gemma 4's cross-layer KV sharing. The insight is that attention patterns can diverge even when the K and V tensors are inherited — Q alone provides enough degrees of freedom. The ~50% KV cache savings is a direct cost reduction for the reasoning/agent workloads where context stays resident for many turns.

> The expensive transformer blocks stay closer to the smaller "effective" size. Additional capacity is stored in per-layer embedding tables.

On Gemma 4's PLE mechanism. This is the architectural equivalent of putting fast-but-expensive RAM in the CPU and slower-but-cheaper storage off-chip — except here the "cheap" parameters are per-layer embedding lookups. Raschka rightly notes this hasn't been rigorously ablated: we don't know if PLE beats just making the model bigger.

> Full-attention layers are more expensive (looking across the whole context), so they get fewer query heads. Sliding-window layers get more capacity since they're cheaper per token.

Laguna XS.2's per-layer attention budgeting. This is the cleanest idea in the article: budget attention capacity where it costs least, not where it's most uniform. It's surprising more models don't do this — the per-layer config is trivial to implement compared to custom attention kernels.

> Where MLA compresses the per-token KV representation, CSA and HCA compress along the sequence dimension — summarizing groups of tokens into fewer compressed KV entries.

The DeepSeek V4 distinction. MLA saves cache space per token; CSA/HCA save by representing fewer tokens altogether. The three-branch architecture (sliding window + mild compression + heavy compression) is a bet that different context horizons need different resolution.

> These new tweaks "probably 10x the code complexity" — not inherently bad since they reduce runtime costs, but making it increasingly difficult to understand individual components and their interactions.

Raschka's closing warning. This is the quiet thesis of the piece: architecture progress is real but it comes at the cost of comprehensibility. A 100-line transformer is grokable in an afternoon. These variants take weeks.

---

## Key Themes

- #concept **KV cache compression** — The common thread across all four models. KV cache size is the binding constraint on long-context inference, and each model attacks it differently: reuse (Gemma 4), sequence compression (DeepSeek V4), latent-space compression (ZAYA1-8B).
- #concept **attention budgeting** — Laguna XS.2's per-layer query-head allocation treats attention as a spendable resource. Full-attention layers get fewer heads because each head costs more.
- #concept **residual-stream engineering** — DeepSeek V4's mHC is the first major rethinking of residual connections since Pre-LN. The manifold constraints (doubly stochastic matrices) make parallel residual streams safe to scale.
- #concept **effective vs. nominal parameters** — Gemma 4's E2B/E4B naming convention reflects a real design pattern: separate the transformer compute budget from the embedding-table capacity budget.
- #pattern **compression in latent space vs. sequence space** — MLA compresses per-token; CCA compresses Q/K/V and operates in compressed space; CSA/HCA compresses along the sequence dimension. Same goal, different axis.
- #comparison **dense vs. MoE vs. per-layer capacity** — Gemma 4 uses PLE, ZAYA1-8B uses extreme MoE, DeepSeek V4 uses parameter-sparse MoE. The old dense-vs-MoE binary is fracturing.

## Critical Analysis

**What this article does well:** Raschka has an unusual talent for making architecture papers legible without dumbing them down. His visual-dense approach (diagrams referenced throughout) and consistent framing (every model as an answer to "how do we make long context cheaper?") turn what could be a laundry list into a coherent survey. The explicit decision to skip benchmarks, training recipes, and RL focuses the piece on what's actually new in the transformer block.

**What's missing:** The lack of comparative benchmarks is honest (Raschka says it upfront) but leaves a genuine gap — we don't know whether cross-layer KV sharing or compressed attention produces better real-world results. The piece doesn't engage with the question of whether these architectural gains compound or conflict. Would Gemma 4's KV sharing work alongside DeepSeek's mHC? No one knows yet.

**The sharp take:** The 10x code complexity observation is the most important line in the article and it's buried near the end. If basic transformers are legible in 100 lines and these variants take 1,000, then architectural progress is actively working against the understandability that makes the field accessible. This isn't just an aesthetic complaint — incomprehensible architectures are harder to debug, harder to align, and harder to build tooling for. The pattern matches what happened to compilers (from simple recursive-descent to multi-million-line optimization frameworks) and operating systems. We're watching the transformer go from hobby project to industrial artifact in real time.

**Compared to related wiki coverage:** Pokee AI's [[Pokee-Isaac 28B]] (August 2026) claims a "non-decoder-only architecture" with 10M-token context on a single GPU, partially derived from Qwen3.6-27B — but unlike the four architectures Raschka surveys, it's closed-source with no published paper, no independent benchmarks, and no technical detail about what "non-decoder-only" means. The contrast between Raschka's transparent, reproducible architectures and Pokee's marketing-forward launch is itself a data point about where the field is heading. [[Components of a Coding Agent]] is Raschka's practitioner piece — this article is his researcher hat. Together they form a complete picture: the harness matters more than the model for coding, but the model architecture determines the cost envelope the harness operates within. [[DS4 (DwarfStar 4)]] is the concrete implementation of DeepSeek V4's CSA/HCA in C — you can read the compression code at `ds4.c:411-415`. [[KV Cache Locality]] covers the routing side of the same problem: even the best KV compression doesn't help if requests land on the wrong GPU. [[Poolside Laguna S 2.1]] is the follow-up to the Laguna XS.2 discussed here: scaled from 33B/3B active to 118B/8B active, same pre-training data, same architectural family, but with FP8 RL and a behavioral training focus that produced a 70.2% Terminal-Bench score — the architecture established, the scale and RL amplified it.

**Who should read this:** Anyone building or evaluating LLM-powered systems where context length is a cost driver. If you're paying per-token for reasoning models or running agent loops that keep context resident, these architecture changes directly affect your bill. The article is also the best available map of where transformer research is heading in mid-2026.

---

*Sources: [[summary/recent-developments-in-llm-architectures]]*
*Last updated: 2026-05-22*
