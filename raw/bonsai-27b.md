---
url: https://prismml.com/news/bonsai-27b
title: Announcing Bonsai 27B — The First 27B-Class Model to Run on a Phone
author: PrismML
date_fetched: 2026-07-18
date_published: 2026-07-14
---

# Announcing Bonsai 27B: The First 27B-Class Model to Run on a Phone

PrismML announced Bonsai 27B, built on Qwen3.6 27B, as the latest multimodal flagship in their Bonsai family and claims it is the first model in its capability class capable of running on a phone.

## Model Variants

**Ternary Bonsai 27B:** Uses ternary weights ({-1, 0, +1}) with FP16 group-wise scaling at 1.71 effective bits per weight. Weighs 5.9 GB and targets laptop-class quality with "full reasoning, tool-calling, and agentic capability."

**1-bit Bonsai 27B:** Uses binary weights ({-1, +1}) with group-wise scaling at 1.125 effective bits per weight. Weighs 3.9 GB, designed to fit within an iPhone 17 Pro's memory budget. Both variants use low-bit representation across the entire language network — embeddings, attention, MLPs, and the LM head — "with no higher-precision escape hatches."

Both are multimodal with a compact 4-bit vision tower for screenshots, documents, and camera input. Supports 262K-token context and speculative decoding. Released under Apache 2.0 License.

## Benchmark Performance (Thinking Mode)

| Category | Qwen 3.6 27B | Ternary 27B | 1-bit 27B |
|---|---|---|---|
| Math (GSM8K, MATH-500, AIME25, AIME26) | 95.3 | 93.4 | 91.7 |
| Coding (HumanEval+, MBPP+, LiveCodeBench) | 88.7 | 86.0 | 81.9 |
| Agentic/Tool-calling (BFCL v3, TauBench) | 80.0 | 74.0 | 66.0 |
| Instruction Following (IFEval, IFBench) | 78.4 | 71.8 | 65.8 |
| Knowledge/STEM (MMLU-Redux, MuSR) | 83.1 | 77.0 | 73.4 |
| Vision (MMMU Pro, OCRBench) | 72.6 | 65.2 | 59.6 |
| **Overall (15 benchmarks)** | **85.0** | **80.5** | **76.1** |

Ternary retains roughly 95% of the full-precision baseline; the 1-bit version retains about 90%.

## Key Technical Details

**Intelligence Density:** The 1-bit version delivers 0.53 per GB, described as more than 10x the full-precision baseline and roughly 2.7x the best available low-bit alternative in the same parameter class.

**Performance:** On an NVIDIA GeForce RTX 5090, Bonsai 27B hits up to 163 tok/s (1-bit) and 134 tok/s (ternary). On an M5 Max, it reaches up to 87 tok/s (1-bit) and 58 tok/s (ternary).

**Phone Constraint:** The article notes phones never expose full memory to apps — a 12 GB iPhone offers roughly 6 GB usable. The 1-bit variant at ~4 GB is "the first to pass through with room to work."

## Platform Coverage

Runs natively on Apple devices (Mac, iPhone, iPad) via MLX and on NVIDIA GPUs via CUDA, using custom low-bit kernels built for a hybrid-attention architecture.

## Company Background

PrismML emerged from a team of Caltech researchers, founded with backing from Khosla Ventures, Cerberus, and Google, with continued support from Samsung.

## Resources Referenced

- **Whitepaper:** Available on GitHub at PrismML-Eng/Bonsai-demo
- **Demo:** WebGPU kernels on Hugging Face Spaces
- **API:** Available via Together.ai
- **iOS App:** Locally AI on the App Store
- **Models:** Hugging Face collection at prism-ml/bonsai-27b
- **Additional Bonsai models:** 8B, 4B, 1.7B, and Bonsai Image 4B

## Noteworthy Claims

The article positions local execution as transformative for agentic workloads — "the marginal cost of a hundred-step loop is zero, and the user's data never leaves the machine." It also proposes hybrid architectures where local models handle privacy-sensitive and non-frontier tasks while cloud models handle the hardest steps, reducing system cost.
