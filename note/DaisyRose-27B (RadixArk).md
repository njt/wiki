# DaisyRose-27B (RadixArk)

RadixArk's first release: a 27.8B multimodal reasoning model that takes Qwen3.8-27B and improves it with **reinforcement learning only** — no SFT, no distillation — using ~1,200 curated long-horizon agent tasks and 18 days on 144 H200s. Terminal Bench 4.1 roughly doubles (5.6 → 12.0), the base model's 256K context and image understanding are preserved, and the Apache-2.0 weights ship in BF16/FP8/NVFP4 as a drop-in replacement.

---

## What the card actually documents

Most model cards are marketing with a benchmark table stapled on. This one is better than most: it names the harness for every benchmark (mini-swe-agent, Terminus 2), the number of repetitions averaged, where each competitor's number came from, and the sampling settings (temperature 1.0, top_p 0.95, top_k 20, `xhigh` reasoning effort, no token cap). It even discloses that reported scores are "the higher of our BF16 and FP8 runs" — an honest wrinkle most cards would hide.

> "Each task targets a mistake the base model makes and is tuned until its solve rate is neither too easy nor too hard to learn from."

This is the curriculum engineering that makes RL-only training plausible at all. RL is only as good as its task distribution; here the distribution is adversarially curated against the base model's specific failure modes, and tasks that could be "passed without really solving" were fixed or removed after review of the model's actual attempts. That last clause — human review of *attempts*, not just tasks — is the anti-reward-hacking discipline, and it echoes the failure-first methodology in [[What Broke and Why — RL Post-Training]].

> "128 H200s ran the agent on its tasks, served by SGLang, while 16 more trained on the finished trajectories; after every step the updated weights went back to the 128."

The infrastructure reveal: the training is *asynchronous agent RL*. Most of the compute (128/144 GPUs) generates long sandboxed agent sessions — 1M-token trajectories, 100+ tool calls — and only ~11% does gradient work. This is agentic RL as a data-generation pipeline, with the cost profile of a software factory (sandbox time dominates) rather than classic RLHF. ~63,000 GPU-hours for a 27B model is also a datapoint on the price of post-training against agent benchmarks.

## The benchmark read

The honest headline is narrower than the table suggests. Terminal Bench 4.1 at 12.0 still trails MiMo V2.6 Flash (16.1) and GPT-6 Luna (15.2); on DeepSWE v1.1 (46.2) it's behind several much larger models; on SWE Bench Pro v2 (94.7) and GPQA Diamond (90.7) it's genuinely competitive with 500B+ open models. The pattern: a dense 27B trained on hard agent tasks punches far above its size on coding/agent work while staying even on general reasoning. Also notable: Muse Glimmer-30B scores 0.5 on TB 4.1 — benchmark-version churn is brutal, and yesterday's agents are tomorrow's zero scores.

## Key Themes

- **#concept RL-only post-training**: no SFT stage, no distillation from a teacher. The bet is that curated long-horizon agent environments with verifiable outcomes can carry the entire adaptation — a datapoint for the agentic-RL thesis that [[Lessons from Building Cursor]] also endorses (RL as the only path to tool-use).
- **#pattern Drop-in post-training**: keeping the base architecture, tokenizer, chat template, vision encoder and MTP head unchanged means existing inference and post-training stacks work without modification. The RL run changed the weights, not the shape.
- **#tool Serving ergonomics as part of the release**: SGLang + DSpark speculative decoding launch commands per GPU/precision, thinking-mode sampling defaults, and the instruction to send `reasoning_content` back on multi-turn turns. The card treats local serving as a first-class product surface.

## Opinionated take

The most interesting thing here isn't the numbers — it's that a small team can now buy a doubling on the current agent benchmark for ~63K GPU-hours, provided they invest as heavily in *task curation* as in compute. The two paragraphs on training data ("reviewed every task together with the model's attempts") are doing more work than the table. The comparison to GPT-6 Luna is friendly framing: at 27B dense with a 23GB FP4 footprint, this model competes with frontier *agent* models on cost-per-token-of-attention, not on absolute score — which is exactly the trade the local-inference crowd cares about.

The weak spot the card doesn't discuss: with only 1,200 training tasks, generalization beyond the task distribution is an open question, and the benchmarks are all agent-shaped, which is both the point and the risk of overfitting. Still, as a statement about where open post-training is heading — curated sandboxes, async agent RL, single-GPU deployability — this is a strong datapoint.

## Related pages

- [[Wall-Clock Time and the Qwen3.8-27B Daily Driver]] — a practitioner already running the base model daily as a coding agent; DaisyRose claims to be a drop-in upgrade for exactly that workload, so his wall-clock thesis gets a direct test case.
- [[Qwen3.8 27B Hardware Tests]] — the same base model's local hardware envelope; DaisyRose's FP8/NVFP4 variants (29 GB / 23 GB) shift those results meaningfully downward.
- [[What Broke and Why — RL Post-Training]] — Verma's RL debugging field guide gives the "why it breaks" companion to this card's "how we trained"; the card's anti-gaming task review is his correctness-gated rewards in practice.
- [[Open models lag state-of-the-art closed models by 4 months]] — this release is a concrete datapoint for the lag question: a 27B open-weight agent model trailing GPT-6 Luna by ~3 points on TB 4.1 while beating several 500B+ open models.

---
*Sources: [[raw/daisyrose-27b]], [[summary/daisyrose-27b]]*
*Last updated: 2026-11-20*
