# North Mini Code GGUF

heimann's community GGUF quantization of Cohere's North Mini Code 30B MoE model — the practical "how to run it" companion to the base model release. Two quants available (Q4_K_M at 18.6 GB, Q5_K_M at 21.7 GB), both fitting on a single 24 GB consumer GPU with ~210–230 tok/s throughput. Requires a patched llama.cpp fork until upstream cohere2_moe support lands.

---

## Key Quotes

> "Uses the standard GGUF keys and are expected to load on upstream once that PR lands."

heimann is doing the right thing here — using the canonical GGUF key schema even though it means running a fork for now. When llama.cpp PR #24260 merges, these quants will load without changes. This is the opposite of the pattern where quant publishers bake in fork-specific conventions that create a migration tax later. The speedy-llama requirement is temporary scaffolding, not a permanent fork.

> Q4_K_M delivers 230 tok/s fully offloaded on a 24 GB card with PPL 8.34 (+3.2% vs bf16).

For a coding agent loop, 230 tok/s is the difference between a model that feels conversational and one that makes you check your phone between responses. At that speed, the 30B→3B MoE sparsity suddenly matters in practice — you're getting 30B-parameter quality at speeds that would be impossible from a dense 30B on consumer hardware.

---

## Architecture Notes

**The cohere2_moe bottleneck**: The model uses Cohere's `cohere2_moe` architecture, which isn't yet in upstream llama.cpp. This is the critical detail. Until PR #24260 lands, anyone wanting to run this locally needs to compile [speedy-llama](https://github.com/heimann/speedy-llama) — heimann's own llama.cpp fork. The fork exists purely to bridge the gap, and the use of standard GGUF keys means the transition to upstream will be seamless.

**Quant choice trade-off**: Q5_K_M at 21.7 GB is the sweet spot if your card can spare the VRAM — only +1.3% PPL degradation vs bf16 and 93.4% top-token agreement. Q4_K_M at 18.6 GB is the default: +3.2% degradation but fits comfortably on a 24 GB card with room for context. These numbers follow the pattern documented in [[Choosing a GGUF Model]] — K-quants hit the knee of the quality-per-byte curve, and the `_M` suffix spends extra precision where it matters (attention value projections, final layers).

**Sampling params worth noting**: `--temp 1.0 --top-p 0.95` with the jinja chat template. Temperature 1.0 is high for a coding model — most coding workflows use 0.1–0.3. This likely reflects the base model's training, which tuned for agentic flexibility rather than deterministic code generation. If you're using this for structured output (function calling, diff generation), start lower.

---

## Critical Analysis

**Strong: This is the model the local-first ecosystem needed.** The [[Cohere North Mini Code]] base release was an announcement — Apache 2.0, 30B MoE, runs on a single H100. But actual availability matters, and this GGUF repo is what makes it real. 766 downloads in the last month for a model that requires a custom fork suggests genuine demand. The local coding agent ecosystem ([[clawdBot]], [[Odysseus]], [[What I learned building an opinionated and minimal coding agent|Pi]], [[Hermes]]) now has a credible open-weight coding specialist that fits on consumer hardware.

**Strong: The benchmark transparency is unusual and welcome.** Most GGUF repos on Hugging Face just say "Q4_K_M" and leave you to guess about quality. heimann published PPL, top-token agreement, and mean KL divergence for both quants, with methodology notes. This is what responsible quant publishing looks like — it lets users make informed trade-offs instead of downloading the most popular file and hoping.

**Weak: The speedy-llama dependency is a real friction point.** Compiling a custom llama.cpp fork is not a one-click experience for the developers Cohere is targeting with the "sovereign developer" pitch. The PR is pending, but "pending" in open source can mean weeks or months. Until it lands, the barrier to entry is a C++ build system, and that will filter out a significant fraction of potential users.

**Weak: Only two quants is thin.** Most popular models on Hugging Face ship 6–8 GGUF variants covering the full range from IQ2_XXS (for VRAM-constrained) to Q8_0 (for maximum quality). Two quants — Q4_K_M and Q5_K_M — is a minimal offering. There's no IQ variant for users who want to squeeze this onto a 16 GB card, and no Q8_0 for those with 48 GB who want near-lossless. This may reflect heimann's pragmatic assessment that the model barely fits at 4-bit and lower quants would degrade unacceptably, but it limits the audience.

**The bigger picture**: The GGUF release ecosystem is the supply chain for local AI. [[Cohere North Mini Code]] is the factory; this repo is the distribution center. The quality of the quantization directly determines whether the model is usable in practice — a bad quant can ruin a good model. heimann's repo is a good quant of a good model with honest benchmarks and a temporary fork requirement. That's about as good as the GGUF ecosystem gets in mid-2026. The fact that it still requires "compile a fork" is a reminder of how immature the local inference tooling layer remains, even as the models themselves cross the viability threshold.

---

## Cross-References

- [[Cohere North Mini Code]] — The base model this GGUF is quantized from. Architecture, benchmarks, and the sovereign developer thesis.
- [[Choosing a GGUF Model]] — Benjamin Marie's taxonomy of GGUF quantization formats applied: where Q4_K_M and Q5_K_M fit in the quality/size/speed trade-off space.
- [[Local and Open Source Inference]] — Hub page for the ecosystem this model serves.
- [[Local Models in Mid-2026]] — Matt Coles on why local models crossed the viability threshold right as DRAM prices doubled.
- [[Datacenter GPU in a Gaming PC]] — Practical local hardware setups where GGUF format selection is a daily concern.
- [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] — A concrete two-GPU config example showing quant choice as the difference between fitting and not.
- [[Inference Cost Napkin Math]] — The arithmetic that makes 4-bit quantization necessary in the first place: memory bandwidth is the bottleneck, compute sits idle.
- [[MiMo Code]] — Xiaomi's coding agent that takes the opposite approach: general capability with specialized harness rather than specialized model with general harness.

---
*Source: [[raw/north-mini-code-gguf]]*
*Last updated: 2026-07-05*
