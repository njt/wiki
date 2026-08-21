# Qwen3.8 27B Hardware Tests

Hardware Corner's llama.cpp benchmark of Qwen3.8 27B (Q4_K Small, 16.68 GiB, 27.32B params) across six hardware targets — RTX 3090, RTX 4090, RTX 5090, dual RTX 5060 Ti, Apple M5 Max, and NVIDIA GB10 — measuring what it actually takes to run the model locally as context scales. The throughline is that VRAM, not compute, is the first practical limit for a 27B-class model, and that Qwen3.8 is a closer cousin to Qwen3.6 than a speed improvement, trailing Meta's [[Muse Glimmer]] 30B at long context.

---

## Key Quotes

> "That makes VRAM the first practical limit."

The article's organizing claim, and it holds for every card tested. A 16.68 GiB Q4 model fits in 24 GB, but the KV cache and runtime overhead are what you're actually buying VRAM for. The numbers scale predictably: 18 GB at 4k, 22 GB at 64k, 26 GB at 128k, 34 GB at 256k.

> "The important number for a single 24 GB card is 64k."

The practical ceiling for the most common consumer GPU. At 64k the model uses ~22 GB, leaving "limited headroom." 128k needs ~26 GB — past the 24 GB cards — which is why the 32 GB RTX 5090 is the only single-GPU route to 128k.

> "This is an important point when looking only at GPU specifications."

The RTX 5090's spec sheet says 32 GB and higher compute, but its generation rate falls from 74.83 tok/s at 4k to 22.79 at 128k. More VRAM buys the *ability* to run long context, not the *speed* — a distinction that spec-sheet shopping erases and this article restores.

> "the Qwen3.8 result should not be interpreted as a straightforward speed improvement over Qwen3.6."

The single most surprising finding. On the same RTX 5090, Qwen3.8 generates 26.22 tok/s at 64k versus Qwen3.6's 63.66 — a 2.4× regression — and prompt processing is similarly behind. A point-release model that is dramatically *slower* at long context is a caution against assuming newer = better for local inference.

> "Muse Glimmer 30B is faster in our RTX 3090 tests at every directly comparable context length."

And more memory-efficient: Glimmer used ~16 GB at short context and ~20 GB at 256k, where Qwen3.8 hit ~22 GB at 64k and ~26 GB at 128k. For long-context agentic work on a used 3090, the article says Glimmer is "the easier model to run."

---

## Key Themes

#concept **VRAM is the binding constraint, context is the variable.** For 27B models the weights fit on a 24 GB card; what the card *can't* do is hold the KV cache for very long contexts. This is the exact relationship [[Self-Hosted LLMs]] formalizes as "max concurrent requests = available memory / KV cache per request."

#pattern **The capacity-vs-throughput tradeoff.** The GB10 and M5 Max are "high-memory... rather than high-throughput" platforms (12.2 and 31.4 tok/s at 4k respectively); the RTX 4090/5090 are throughput cards that hit a VRAM wall. There is no single card that is both — you pick.

#tool **llama.cpp as the common substrate.** Everything runs on one llama.cpp build (153d324bc) with Flash Attention enabled and MTP disabled, which makes the cross-card numbers comparable but also pins them to one quant and one engine — the article is explicit that these are measurements, not universal figures. Note the quirk: llama.cpp reports the architecture internally as `qwen35`.

#comparison **Qwen3.8 is a hardware regression on Qwen3.6.** The benchmark's real contribution is falsifying the "next version is faster" assumption — at long context Qwen3.8 is markedly slower than both Qwen3.6 and Muse Glimmer.

---

## Critical Analysis

The value here is that someone ran the same model, the same quant, and the same engine across six targets and published the tables — rarer than it sounds in local-inference land, where most "benchmarks" are a single rig and a single anecdote. The internal consistency (VRAM scales predictably; cross-model comparisons share hardware) is what makes the numbers trustworthy despite the absence of a byline or publication date.

The Qwen3.8-vs-Qwen3.6 result is the finding that matters, and it deserves more attention than a performance table usually gets. A 2.4× generation regression at 64k is not noise; it suggests Qwen3.8 traded long-context efficiency for something else (quality, reasoning, whatever the release notes claimed), and that trade lands squarely on the local-inference user. It also sharpens the [[Muse Glimmer]] thesis: Meta distilled Glimmer *specifically* for the always-on local niche and trained failure-recovery into it, while Qwen3.8 appears to have drifted *away* from that niche's most important constraint. For the [[Local Qwen Is Not a Worse Opus]] framing — local models as a different tool, not a worse frontier — it's a reminder that the "tool" keeps changing shape between point releases.

The RTX 3090 thesis is now close to settled doctrine in this wiki's local-hardware thread, and this article reinforces it from a new angle: it's not just cheap, it's the card where the 24 GB VRAM wall and the value ceiling happen to coincide. [[Datacenter GPU in a Gaming PC]] got 32 tok/s on a £200 V100; [[Dual GPU RTX 5080 + RTX 3090 Qwen 3.6 Setup]] got 80+ with two current-gen cards. Qwen3.8's numbers slot cleanly between them and confirm the same VRAM-first logic.

The honest caveat, which the article itself volunteers, is scope: one quantization (Q4_K Small), MTP disabled, one llama.cpp build. The Qwen3.6 comparison used Q4_K Medium and Muse Glimmer used a 14.78 GiB build, so the quants aren't perfectly matched — the 2.4× gap could narrow on equal-footing builds, though it's unlikely to reverse. Treat the within-card orderings as solid (throughput: 3090 < 4090 < 5090; on the 3090, Glimmer > Qwen3.8; on the 5090, Qwen3.6 > Qwen3.8) and the absolute numbers as provisional.

---

*Sources: [[raw/qwen3-8-27b-hardware-tests]], [[summary/qwen3-8-27b-hardware-tests]]*
*Last updated: 2026-08-21*
