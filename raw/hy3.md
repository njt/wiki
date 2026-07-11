---
url: https://huggingface.co/tencent/Hy3
title: "Hy3 — Tencent's Large MoE Language Model"
author: Tencent Hy Team
date_fetched: 2026-07-11
date_published: 2026
---

# Hy3

Hy3 is a large MoE (Mixture-of-Experts) model developed by Tencent's Hy Team. It has 295B total parameters with 21B active parameters and 3.8B MTP layer parameters. The model is licensed under Apache 2.0.

## Architecture Details

| Property | Value |
|---|---|
| Architecture | Mixture-of-Experts (MoE) |
| Total Parameters | 295B |
| Activated Parameters | 21B |
| MTP Layer Parameters | 3.8B |
| Layers (excl. MTP) | 80 |
| MTP Layers | 1 |
| Attention Heads | 64 (GQA, 8 KV heads, head dim 128) |
| Hidden Size | 4096 |
| Intermediate Size | 13312 |
| Context Length | 256K |
| Vocabulary Size | 120832 |
| Experts | 192 experts, top-8 activated |
| Supported Precisions | BF16 |

Tags: Text Generation, Transformers, Safetensors, hy_v3, hunyuan, hy3, Mixture of Experts, conversational, Eval Results

## Model Description

Following the Hy3 Preview launch in late April, the team gathered feedback from 50+ products and scaled up post-training. Hy3 "outperforms similar-size models and rivals flagship open-source models with 2-5x parameters." It also shows notable gains across various products and productivity tasks.

## Agent Capabilities

The model demonstrates solid gains across reasoning, agentic, and long-context tasks. In productivity scenarios including coding, office work, financial modeling, frontend design, and game development, it has made significant progress.

A blind evaluation with 270 experts gave Hy3 a score of 2.67/4, surpassing GLM-5.1 at 2.51/4. The largest advantage appeared in frontend development, data & storage, and CI/CD tasks.

## Product Experience Improvements

Three key areas were addressed:

1. **Tool call stability**: "Hy3 also generalizes across different agent scaffoldings." On SWE-Bench Verified, accuracy variance across scaffoldings like CodeBuddy, Cline, and KiloCode stays within 4%.

2. **Hallucination reduction**: The hallucination rate dropped from 12.5% to 5.4%, and commonsense error rates fell from 25.4% to 12.7%.

3. **Multi-turn tracking**: Through joint SFT and RL optimization, issue rates on internal multi-turn tests dropped from 17.4% to 7.9%.

## Evaluation Results / Benchmarks

| Benchmark | Score |
|---|---|
| GPQA Diamond | 90.4 |
| SWE-bench Verified | 78 |
| Skillsbench V1 | 55.3 |
| SWE-bench Pro | 57.9 |
| Apex Agents | 25.6 |
| Deep SWE | 28 |
| HLE | 53.2 |
| WildClawBench Overall | 53.6 |

## Usage Instructions

### Transformers (Pipeline)

```python
from transformers import pipeline
pipe = pipeline("text-generation", model="tencent/Hy3")
```

### Transformers (Direct)

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
tokenizer = AutoTokenizer.from_pretrained("tencent/Hy3")
model = AutoModelForCausalLM.from_pretrained("tencent/Hy3")
```

### OpenAI-compatible API

The recommended deployment flow uses vLLM or SGLang, then calling via OpenAI client:

```python
from openai import OpenAI
client = OpenAI(base_url="http://127.0.0.1:8000/v1", api_key="EMPTY")
```

**Recommended parameters**: temperature=0.9, top_p=1.0. For reasoning, set `reasoning_effort` to `"high"` for complex tasks or `"no_think"` for direct responses.

## Deployment

### vLLM

Build from source with Python 3.12 and uv. Serve with MTP enabled:

- Tensor parallel size: 8
- Uses `--speculative-config.method mtp` with `num_speculative_tokens 2`
- Tool call parser: `hy_v3`, reasoning parser: `hy_v3`

### SGLang

Build from source, requires `transformers>=5.6.0`. Launch with:

- TP size 8
- Tool call parser: `hunyuan`, reasoning parser: `hunyuan`
- EAGLE speculative algorithm with `speculative-num-steps 2`, `speculative-eagle-topk 1`, `speculative-num-draft-tokens 3`

## Finetuning

A complete finetuning pipeline is available via a dedicated Finetuning Guide in the repository.

## Quantization

The team provides **AngelSlim**, a toolkit for large model compression supporting "a comprehensive suite of compression tools for large-scale multimodal models." An FP8 quantized version (Hy3-FP8) is also available.

## Model Links

| Model | Description | Platforms |
|---|---|---|
| Hy3 | Instruct model | Hugging Face, ModelScope, GitCode, CNB |
| Hy3-FP8 | FP8 quantized instruct model | Same four platforms |

## License

Apache License 2.0

## Authors / Contact

Developed by the **Tencent Hy Team**. Contact: hunyuan_opensource@tencent.com

## Additional Page Info

- Downloads last month: 6,923
- Model size: 299B params
- Tensor type: BF16, F32
- Finetunes: 1 model
- Quantizations: 40 models
- Spaces using Hy3: 7
- Community discussions: 14
