---
url: https://ianbarber.blog/2026/06/19/llms-are-complicated-now/
title: "LLMs are complicated now"
author: Ian Barber
date_fetched: 2026-07-05
date_published: 2026-06-19
---

# LLMs are complicated now

**Author:** Ian Barber
**Blog:** Ian's Blog
**Publication Date:** June 19, 2026
**Category:** Modelling
**Tags:** gpu, llm, recsys, triton

## Full Content

The author reflects on how LLM architectures have evolved from the clean, uniform transformer stacks seen in early Llama models to far more complex systems resembling the "terrifying" recommendation system graphs that once seemed chaotic by comparison.

### Architectural Complexity Growth

Ian notes that while "attention might be all you need," modern models deploy many attention variants — including query grouping, compressed, sparse, linear, and sliding-window approaches. Mixture-of-Experts has expanded beyond feed-forward layers to route attention blocks and even residual streams. Vision and audio encoders are now tightly integrated rather than bolted on, and multi-GPU inference introduces communication ops that create additional boundaries within models.

### Parallels with Recommendation Systems

The author draws a direct comparison to the evolution of recommendation systems, which maintained a simple two-tower sparse neural net architecture for years. Complexity there emerged from "the tension between the need to continually increase capabilities and the need to stay efficient."

### The Optimization Trap

A central argument is that agents alone won't solve this complexity problem. The author writes that "you can't generate your way forward without a baseline to check." The gap between performance being an optimization versus a necessity shrinks rapidly in practice. If variant B of an attention mechanism is 10% slower, that's tolerable; being an order of magnitude worse isn't. But without fused implementations, you can't even evaluate whether a novel variant is worth pursuing. The author concludes that "the only way out is to design for composability up front."

### FlexAttention as a Positive Example

The article highlights PyTorch's FlexAttention as a standout kernel development. It enabled generating kernels for a whole class of attention operations via Triton templates, allowing exploration "with only a very mild impact to performance" by building on composable, verifiable design.

### On Karpathy's Role

The post notes Andrej Karpathy joined Anthropic to advance auto-research loops, but emphasizes that "being able to cut architectures to their essence and make them composable" matters as much as clever agentic setups when climbing the research frontier.

*The article also includes two footnotes — one acknowledging colleagues in Content Understanding and integrity work at Meta, and another referencing automated approaches inspired by Hazy Research's Megakernels repository.*
