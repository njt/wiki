# Cohere North Mini Code

Cohere's first open-source agentic coding model: a 30B Mixture-of-Experts architecture that activates only 3B parameters per forward pass, runs on a single H100 GPU, and is licensed Apache 2.0. The model is purpose-built for coding agent workflows — code generation, review, architecture design, and sub-agent coordination — achieving 33.4 on the Artificial Analysis Coding Index with 2.8× the throughput of Devstral Small 2. It's the "sovereign developer" play: a model you can run on your own hardware with no API dependency.

---

## Key Quotes

> "Small, efficient, and open-source — our first agentic coding model, built for the sovereign developer ecosystem."
— Cohere team blog post

> "Built to run where you need it" — positioning North Mini Code as a local-first alternative to cloud-only coding models, not just a cheaper API option.

The "sovereign" framing is doing real work here. This isn't just about cost — it's about developers who can't or won't send proprietary code to a third-party API. Cohere is betting that market is larger than the cloud-only one.

---

## Key Themes

#coding-agents #open-source #local-inference #moe #sovereign-ai

---

## Architecture

**MoE with extreme sparsity**: 30B total, 3B active. That's a 10:1 sparsity ratio — more aggressive than Mixtral (8:1) and closer to DeepSeek-V2's design philosophy. The bet is that coding tasks benefit from specialized experts more than general reasoning tasks do.

**256K context, 64K generation**: Large enough for full-repo context in most codebases, though the 64K generation ceiling means it won't produce novel-length diffs. Practical for real-world PRs.

**Single H100 at FP8**: This is the headline. A 30B MoE fitting on one consumer-adjacent GPU means this is genuinely runnable by individual developers, not just enterprises. At 4-bit quantization it reportedly fits on a Mac Studio (~20GB).

---

## Benchmark Reality Check

The 33.4 on the Coding Index puts it above GLM-4.7-Flash (25.9) but below Qwen3.6 35B (35.2). That's the right neighborhood for a 3B-active model — punching above its weight class, but not magically outperforming larger dense models.

The throughput claims (2.8× Devstral Small 2) matter more in practice. For agentic loops where the model makes dozens of sequential calls, per-token latency compounds into wall-clock time that determines whether the agent feels responsive or glacial.

The non-coding weakness (14% GDPval-AA, 37% τ²-Bench Telecom) is honest and expected. This is a specialist, not a generalist. Compare to [[MiMo Code]] which takes the opposite approach — general capability with specialized harness design.

---

## Critical Analysis

**Strong**: The Apache 2.0 license is the real story. Cohere's previous models used CC-BY-NC, which meant "open weights but don't use them commercially." Apache 2.0 removes that friction entirely. For the ecosystem of tools like [[What I learned building an opinionated and minimal coding agent|Pi]], [[clawdBot]], and [[Odysseus]] that let developers run agents locally, this is a genuinely useful new option. The single-H100 requirement puts it within reach of individual developers with a decent GPU budget.

**Strong**: The MoE sparsity ratio (30B→3B) is architecturally interesting. It suggests Cohere believes coding tasks have high expert specialization — different kinds of code (Python vs Rust, backend vs frontend, generation vs review) activate different pathways. If true, this is a more efficient use of parameters than a dense 3B model. If the specialization doesn't hold up in practice, it's just a 3B model with 30B of overhead.

**Weak**: "Chatty" output is a real problem for an agentic coding model. Verbose responses waste context and slow down agent loops — every extra token the model generates is a token the next turn has to process. The 2.8× throughput advantage could be partially or fully eaten by verbosity. Compare [[Honey I Shrunk the Coding Agent]]'s finding that harness design matters more than raw model performance — a chatty model with a bad harness is worse than a terse model with a good one.

**Weak**: The 14% on GDPval-AA and 37% on τ²-Bench Telecom suggest the model falls apart outside its training distribution. For a coding agent that will inevitably encounter non-coding tasks (reading docs, understanding user intent, debugging logic errors that require domain knowledge), this brittleness is a liability. The [[Components of a Coding Agent]] taxonomy makes clear that coding agents need more than code generation — they need reasoning, planning, and tool use that may draw on capabilities North Mini Code simply doesn't have.

**The bigger picture**: This is part of a wave. Between North Mini Code, Qwen3.6, [[MiMo Code]], and the 3B-to-9B models documented in [[Honey I Shrunk the Coding Agent]], we're seeing a convergence: coding agents are becoming small enough to run locally, and the open-weight models are catching up fast. The question isn't whether local coding agents will be viable — it's whether the cloud models' advantage (bigger context, faster iteration, no hardware management) justifies their dependency cost. For the "sovereign developer" Cohere is targeting, the answer is already no.

---

*Sources: [[summary/cohere-north-mini-code]], [Cohere blog: Introducing North Mini Code](https://cohere.com/blog/north-mini-code)*
*Original VentureBeat article: https://venturebeat.com/technology/cohere-open-sources-a-coding-agent-that-runs-on-a-single-h100 (paywalled)*
*Last updated: 2026-06-15*
