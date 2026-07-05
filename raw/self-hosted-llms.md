---
url: https://selfhostllm.org/
date_fetched: 2026-07-05
backfilled: true
---

This memory is needed for each active request's attention cache.

4. Available Memory for InferenceAvailable = Total VRAM - System Overhead - Model Memory

Example: 48GB - 2GB - 7GB = 39GB

This is what's left for KV caches after loading the model.

5. Maximum Concurrent RequestsMax Requests = Available Memory / KV Cache per Request

Example: 39GB / 11.47GB = 3.4 requests

What the Results Mean:

< 1 request: Can't handle full context length, need smaller context or better GPU

1-2 requests: Basic serving capability, suitable for personal use

3-5 requests: Good for small-scale deployment

10+ requests: Production-ready for moderate traffic

Mixture-of-Experts (MoE) Models:

Special Handling for MoE Models

MoE models (like Mixtral, DeepSeek V3/R1, Qwen3 MoE, Kimi K2/K2.5, GLM-4.7) work differently:

Total Parameters: The full model size (e.g., Mixtral 8x7B = 56B total parameters)

Active Parameters: Only a subset of experts are used per token (e.g., ~14B active)

Memory Calculation: We automatically use active memory for these models

Why this matters: You only need RAM for active experts, not the entire model

Example: Mixtral 8x7B shows "~94GB total, ~16GB active" - we calculate using 16GB

Important Notes:

This is a rough estimate - actual usage varies by model architecture

Assumes worst-case scenario: All requests use the full context window. In reality, most requests use much less, so you may handle more concurrent requests

KV cache grows linearly with actual tokens used, not maximum context

Different attention mechanisms (MHA, MQA, GQA) affect memory usage

Framework overhead and memory fragmentation can impact real-world performance

Dynamic batching and memory management can improve real-world throughput

MoE models: Memory requirements can vary based on routing algorithms and expert utilization patterns

• Model memory includes weights and activation overhead
• KV cache grows linearly with context length and batch size
• System overhead includes OS, drivers, and framework memory
• Quantization reduces precision but may impact model quality
• Professional GPUs (A100, H100) have better memory bandwidth
• Multiple GPUs require tensor parallelism for large models
