---
url: https://thenewstack.io/jetbrains-mellum2-open-source-coding-model/
date_fetched: 2026-07-05
backfilled: true
---

# JetBrains open-sources Mellum2 to go where Claude Code can’t

JetBrains announced on Monday it has open-sourced Mellum2, a 12B-parameter coding model aimed at the infrastructure layer of agentic AI systems — routing, retrieval pipelines, and sub-agent tasks — as well as private on-premises deployment, somewhere Claude Code and its ilk can’t go.

It’s the follow-on to Mellum, a 4B-parameter model that JetBrains debuted in late 2024 as a proprietary code completion tool for its own IDEs before open-sourcing it in April 2025. But unlike its predecessor, Mellum2 is open from day one.

It’s worth noting that Mellum2’s scope has also changed considerably. Where Mellum did one thing — code completion — Mellum2 is built for the broader set of tasks that now define how engineering teams are deploying AI: coordinating between models, handling sub-agent workloads, compressing context in retrieval pipelines, and running inference on infrastructure teams control themselves.

Mellum2 is built for the broader set of tasks that now define how engineering teams are deploying AI.


In a blog post co-authored by staff research engineer Nikita Pavlichenko and product manager Anton Semenkin, JetBrains describes Mellum2 as a “focal model” — fast and specialized, rather than competing with frontier models on breadth.

“Frontier models will continue to push the limits, but practical AI products also require focal models: fast, specialized components that handle high-frequency tasks efficiently,” they write. “This specialization ensures the model excels in software engineering environments while remaining lean and fast.”

Additionally, two post-trained variants ship alongside the base model: an “instruct” version that answers directly, and a “thinking” version that produces an explicit reasoning trace before responding, aimed at harder multi-step and agentic tasks.

## Built for speed at scale

Mellum2 uses a Mixture-of-Experts (MoE) architecture, with 12B total parameters but only 2.5B active per token. The design routes each token through a subset of the model’s 64 experts rather than the full network, which keeps inference fast without sacrificing the model’s overall capacity.

In its technical report, JetBrains benchmarked Mellum2 against Alibaba’s Qwen2.5-7B and Qwen3-8B on a single H100 GPU, using input and output sizes representative of real production code completion workloads.

In single-request mode, it matches Qwen2.5-7B almost exactly — 192 tokens per second versus 193. Under concurrent load, which is where production deployments actually operate, it pulls 21% ahead of Qwen2.5-7B and 79% ahead of Qwen3-8B.

The cost profile follows the same logic. With only 2.5B parameters active per token, the architecture is designed to behave more like a 2.5B model than a conventional 12B dense model from an inference perspective — relevant for teams routing high volumes of requests through it daily as part of a larger agentic system.

On function-level code generation, measured by the EvalPlus benchmark combining HumanEval+ and MBPP+, Mellum2 scores 78.4% in its thinking variant — ahead of the other models included in the comparison table, including Qwen3.5-9B at 71.8% and the code-specialized Seed-Coder-8B at 73.8%.

The picture becomes more mixed once the evaluation moves beyond software-engineering tasks. JetBrains’ own results show that Qwen3.5-9B retains an advantage in broader reasoning and knowledge evaluations, including GPQA Diamond and MMLU-Redux.

JetBrains acknowledges this directly in its technical report, noting that the model’s narrower training focus comes at a cost.

“The gap reflects a deliberate tradeoff in our training mix toward code and developer documentation rather than broad encyclopedic coverage,” the authors write.

## The dependency argument

The more pointed case for Mellum2, perhaps, is about what it *doesn’t* require. Anthropic’s Claude Code and OpenAI’s Codex run locally on the client but route inference through Anthropic and OpenAI’s APIs, respectively.

Cursor, for what it’s worth, is also dabbling with its own proprietary coding model strategy, recently introducing Composer 2.5. Those capabilities remain tied to Cursor’s platform, while the company’s recently announced partnership with SpaceX’s xAI places another critical layer of the stack — infrastructure and future model development — outside customers’ control.

Mellum2 arrives with open weights under Apache 2.0, giving enterprises the option to own and operate that layer themselves. Whether that argument gains traction at enterprise scale will depend on enterprise appetite for self-hosted AI infrastructure.

JetBrains is betting that deployment flexibility, operational control, and ownership will remain important considerations as AI becomes more deeply embedded in software engineering workflows. A reasonable bet — but one that remains to be proven at scale.

Mellum2 is now available on Hugging Face, with base, instruct, and thinking checkpoints released under Apache 2.0, along with the full technical report detailing the architectural decisions and training pipeline behind it.
