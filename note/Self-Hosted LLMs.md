# Self-Hosted LLMs

A GPU memory calculator and inference performance estimator by Eran Sandler (2026). Maps specific GPU hardware to LLM inference capability: which models fit, how many concurrent requests you can serve, and how fast tokens will generate.

---

## What It Calculates

Two formulas drive the tool:

**Max Concurrent Requests = Available Memory / KV Cache per Request**

This walks through total VRAM, model memory adjusted for quantization, KV cache per request (a function of context length, model size, and KV overhead), and available memory after system overhead. Output is a concurrency number with a rough readiness label: <1 means the model won't fit at that context length, 1-2 is personal use, 10+ is production-ready.

**Tokens/sec = (Memory Bandwidth / Model Size) x Efficiency x Quantization Boost x Context Impact**

Memory bandwidth is the primary bottleneck. The formula layers in diminishing efficiency as models grow larger (85% utilization at <=7B, dropping to 30% at 70B+), quantization speedups (INT4 = 2.2x faster than FP16), context length penalties, and imperfect multi-GPU scaling.

## What Makes It Useful

Most model cards list VRAM requirements but not real-world token generation rates on specific hardware. Knowing a model *fits* is different from knowing it'll generate tokens fast enough to be usable. This tool answers both questions at once.

The model database is impressively comprehensive — covers Llama, Qwen, DeepSeek, Mistral, GLM, Gemma, Phi, Granite, OLMo, Kimi, MiniMax, Nemotron, and many more, with explicit handling for Mixture-of-Experts architectures (Mixtral, DeepSeek V3, Qwen3 MoE) where only active parameters matter for memory calculation.

## Critical Analysis

The formulas are rough estimates, and the tool is transparent about this. Real-world performance varies by framework (vLLM vs. TGI vs. Ollama), batch size, attention mechanism (MHA/MQA/GQA), memory fragmentation, and tensor parallelism efficiency. The ±20% real-world variance caveat is honest.

The multi-GPU scaling numbers (85% for 2 GPUs, 75% for 4, 65% for 8) reflect the harsh reality of inter-GPU communication overhead. This is the kind of detail that matters for actual deployment planning but is often glossed over.

The tool is a calculator, not a benchmark. It won't tell you which model is better for your use case — it tells you whether your hardware can run a model at acceptable speed. For the "I have $X budget, what GPU should I buy?" question, this is probably the most practical single resource available.

The site's real durability risk is maintenance: new models ship constantly, and the database must keep pace. If Sandler stops updating it, the calculator's utility decays with each new model generation. But the formulas themselves don't age — they're physics and architecture, not benchmarks.

---

*Sources: [[summary/self-hosted-llms]]*
*Last updated: 2026-05-18*
