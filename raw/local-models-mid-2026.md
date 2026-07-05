---
url: https://coles.codes/posts/local-models-mid-2026
date_fetched: 2026-07-05
backfilled: true
---

# Local models in mid-2026: the engineering that closed the gap

The 2026 local-model story is quieter than the headlines suggest. Open weights did not catch up to the frontier, but they got close enough on the work most of us do day to day. Running LLM’s locally yourself isn’t just a hobby project anymore and turned into a reasonable choice if you’re after a basic model for writing and research, or running as a specialised agent.

What I find interesting is the engineering that got us here, and the progress didn’t just mean we had to get more RAM to run bigger models. If anything it was the reverse: people figured out how to spend less compute and less memory per token without losing quality.

## My current favourite models #

Qwen 3.6 shipped an open dense 27B alongside a 35B mixture-of-experts that only fires about 3B parameters per token. Gemma 4 from Google spans a spread of sizes, and the larger ones punch well above their weight. GLM-5 is a 744B Mixture of Experts (MoE) and Kimi K2.6 is a trillion parameters with 32B active (although both GLM-5 and Kimi K2 require a bit too much RAM to run with my local setup 😅!). DeepSeek previewed V4 in April in two flavours, Flash and Pro, both MoE with a million-token context.

Interestingly, almost no model in the above list is a dense model you load whole. The overall parameter counts are large and the active counts are small, and that gap is what the rest of this comes down to.

## Sparse attention #

Standard attention is quadratic: the work grows with the square of the context length. Every new token has to look back at every token before it, so a context twice as long means twice as many tokens each doing twice as much looking, which is four times the work. Ten times the context is a hundred times the work. At a million tokens that really adds up, and it’s probably why context windows stayed small for so long (that, and models losing track of what you originally asked once the context got long).

```
              attends to ->
              t1  t2  t3  t4  t5  t6  t7  t8
full    t4    ■   ■   ■   ■                     every token reads every
        t8    ■   ■   ■   ■   ■   ■   ■   ■     token before it: n tokens
                                                doing n look-backs = n^2
sparse  t8    ■   ·   ·   ■   ·   ·   ■   ■     a small recent window plus
                                                the indexer's top-k picks
```
DeepSeek’s work is the great example of sparse attention. DeepSeek V3.2 introduced what they call DeepSeek Sparse Attention, and V4 builds on it. The mechanism is a “lightning indexer”, a cheap scoring function running in FP8 that decides, for each query token, which earlier tokens are actually worth attending to. You keep a small sliding window of recent tokens at full resolution for local coherence, and for everything older you attend only to the top-k the indexer flagged. Complexity drops from quadratic to roughly linear in the selected set. The indexer runs on a separate CUDA stream so its latency hides behind work that’s already happening instead of landing on the critical path.

DeepSeek reported V4-Pro needs something like a quarter of the per-token inference FLOPs and a tenth of the KV cache that V3.2 needed at million-token context, which is the gap between long context working as a demo and working as something you can actually build on.

## Mixture-of-experts #

MoE is the reason a trillion-parameter model runs at all. Instead of one big dense feed-forward network, you have many smaller “expert” networks and a router that sends each token to a handful of them. Kimi K2.6 has 384 experts and activates eight plus a shared one per token. GLM-5 activates roughly 40B of its 744B. The model has the knowledge capacity of its full parameter count but the per-token cost of something much smaller.

It’s worth noting you still have to hold every expert in memory even though you only touch a few per token. So MoE is cheap on compute and bandwidth but very heavy on capacity. That tension is exactly why the hardware section below matters, and why unified-memory machines turned out to suit these models almost by accident.

## The KV cache problem #

People underestimate this one. For long context, and for reasoning models that emit twenty thousand tokens of working-out, the dominant memory cost at inference isn’t the weights: it’s the KV cache, the stored keys and values for every token you’ve seen so far. It grows linearly with context and it has to stay in fast memory.

Two lines of attack showed up everywhere this year. The first is Multi-head Latent Attention, DeepSeek’s trick of compressing the KV cache into a low-rank latent rather than storing it in full, which cuts the footprint by something like ninety percent. Kimi and others adopted variants. The second is simpler: store the cache at lower precision, FP8 and increasingly FP4, which halves or quarters the memory for a small accuracy cost you can mostly train back. Combine compressed attention with a compressed, quantised cache and the long-context memory wall moves a long way out.

## Multi-token prediction #

This one is simple, and it’s the reason local generation feels faster this year. Normally the model produces one token, feeds it back, produces the next, one at a time. Each step is bottlenecked on memory bandwidth rather than maths, so the hardware mostly sits idle waiting for weights to arrive.

Multi-token prediction, which DeepSeek-V3 proved out at scale and which Gemma 4, Qwen and the rest now ship, puts that idle compute to use. It guesses several tokens ahead with a small cheap drafter, then has the full model verify all of them in a single parallel pass. It accepts the run that matches and throws the rest away. DeepSeek reported the second predicted token getting accepted eighty-five to ninety percent of the time, for roughly a 1.8x throughput gain. Gemma 4 ships dedicated little drafter models for exactly this, sharing the main model’s embeddings and KV cache so they cost almost nothing to run.

The property that makes this work, and the reason it isn’t a quality compromise, is that it’s lossless: the big model still checks every token, so the output is identical, just faster. The catch is that the gains depend on the workload. Predictable text drafts well and runs fast, while genuinely novel or high-entropy output gets more drafts rejected, and every rejected draft is wasted work. On an already-fast small model the bookkeeping can occasionally make things slower, so the win is real but how big it is depends on what you’re generating.

## Four-bit quantisation #

The other quiet shift was precision. FP4, in the NVFP4 and MXFP4 formats, went from research to shipping. OpenAI released gpt-oss natively in MXFP4. Nvidia’s Blackwell does FP4 in hardware. A Qwen 3.6 27B drops from around 17GB at four-bit-ish quant to about 14GB in NVFP4, with quantisation-aware training recovering most of what naive rounding would throw away. FP4 does cost you accuracy on small or sensitive models, where the tiny block sizes interact badly with outlier handling, but for the larger models it’s become a sensible default rather than a compromise.

## The memory supply crunch #

All of that engineering made the models cheaper to run but we’ve hit other problems running local models this year, hardware got more way more expensive due to everyone buying up hardware in the AI race.

The memory makers reallocated capacity toward datacentre HBM because it earns several times more per wafer than ordinary DRAM, and an HBM wafer displaces roughly three wafers of the normal stuff. Conventional DRAM contract prices went up somewhere around 90 to 98 percent quarter on quarter in early 2026, with PC DRAM passing a hundred percent, NAND followed, and a 1TB SSD roughly doubled. SK Hynix told an earnings call it had already sold out next year’s capacity, and any real relief for anyone wanting to buy GPU’s, RAM and hard drives isn’t expected before late 2027.

The timing is a bit of a joke: the models are finally good enough to run at home right as the box to run them on got expensive. One upside is that the local-inference machine doesn’t have to be done by a stack of GPUs, you can use a system with unified-memory, an Apple Silicon Mac Studio or an AMD Strix Halo mini-PC with 128GB shared between CPU and GPU. That suits MoE really well, because those models need a lot of capacity to hold all the experts but only modest bandwidth to run the few experts needed to answer a query. A used 3090 is still the budget pick and a 5090 the fast one if you have the $$$. The mid-ground and interesting hardware now is the unified memory systems, although they are getting pricey too.

Later in the year we’ll see some other hardware like Nvidia’s RTX Spark, announced with Microsoft at the end of May and due to ship this fall. It’s a Grace CPU and a Blackwell GPU joined into one superchip with up to 128GB of unified memory, and the pitch is running 120B-parameter models at million-token context locally, in a slim laptop, from the likes of Dell, Lenovo, HP and Surface.

I’ve got an older DGX Spark on the desk already, so I have a fairly good read of what the experience would be like on the RTX Spark and i suspect memory bandwidth may still be an issue as well as cooling for inference. Will keep my eye out for reviews as hopefully those issues are improved and it’d be great to have a full CUDA stack on a portable with that much unified memory, and that would make late-2026 local inference more interesting re hardware choice. I want to see the bandwidth figure and a real tokens-per-second number before I buy, but it’s still the hardware I’m watching most closely right now.

## Where the gap to closed models sits #

Epoch’s measurement is a good benchmark, and it puts the best open weights at roughly four months behind the closed frontier. That’s slightly wider than the three-month average they measured over the previous couple of years. On coding and agentic work the open models are within a few points of the closed ones, and you genuinely might not notice the difference. On hard reasoning and novel maths, the problems that are actually difficult, the closed frontier is still ahead and you can feel it.

In terms of actual benchmark numbers, Artificial Analysis’ June index has Kimi K2.6 at 54 against GPT-5.5’s 60 and Claude Opus 4.7’s 57, DeepSeek V4 Flash level with Sonnet 4.6, and the small dense models you’d actually fit on one GPU, the Qwen 27B and Gemma 31B class, are a tier further down. Their index is built mostly from coding, tool-use and agentic evals, so it measures the day-to-day work I was just talking about. No single chart of theirs has all the models from this post on it, so I’ve tried to collate their numbers on my own chart:

Two things to be aware about though re: benchmarks. Every model benchmaxes these days, so public scores run a bit high for everyone. One model not reflected just yet is Claude Fable 5 as right as I was writing this it came out, and if the early numbers hold, that four-month figure is already stale on the optimistic side. The flip side is that for a lot of people none of this matters past a point: a good-enough model in a good harness already covers their day-to-day, and that’s a perfectly happy place to land for most of what they do.

## Why I run my own #

I run models at home because it’s where the learning happens. An afternoon spent working out why the same model does fourteen tokens a second on one machine and forty on another, watching the KV cache eat your VRAM as you push the context, teaches you more than a month of reading about it.

The other half is flexibility. The open stack lets me pull a model the day it drops, quantise it down to fit whatever box is free, fine-tune it on my own data, and keep the sensitive stuff on hardware I control.

The reason any of this is possible at all is that the engineering above is mostly open: sparse attention, MoE routing, latent KV compression, multi-token prediction and four-bit quantisation are published papers and merged commits rather than trade secrets, and that’s the part of the field worth protecting. The models being good is nice, but it’s the methods staying out in the open that gives the rest of us options.

## $ comments --load
