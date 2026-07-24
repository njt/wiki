---
url: https://apertus-ai.org/articles/2026-07-apertus-1-5/
title: Apertus 1.5
author: ETH AI Center & EPFL AI Center
date_fetched: 2026-07-25
date_published: 2026-07-24
---

# Apertus 1.5

Apertus 1.5 is a continued pretraining of the earlier Apertus 1.0 models at the 8B and 70B parameter scales. The 8B variant received 4 trillion additional tokens of text and multimodal training data, while the 70B model received 2 trillion. The models are "fully open: open weights, open data, open values, and full training details."

## Key Enhancements

- **Native Image Understanding:** The models can now accept images alongside text, including documents, diagrams, and photos. Spoken language processing is also included but noted as experimental.
- **Thinking Mode:** A switchable reasoning mode allows the model to "reason about the input before answering," boosting performance on reasoning-heavy tasks.
- **Long Context:** The context window has been quadrupled to 262,144 tokens compared to Apertus 1.0.
- **Improved Instruction-Following:** The post says this gives "more predictable and accurate responses" compared to the previous generation.
- **Improved Tool Use:** Training was optimized for better integration with external tools and APIs.
- **Open Values:** The models show "stronger adherence to the Apertus Charter," which publicly documents the model's values and principles.

## Technical Report & Access

A full technical report with benchmarks, training pipelines, and intermediate checkpoints is promised "in the coming weeks." Model cards and running instructions are already available on Hugging Face for both the [8B](https://huggingface.co/swiss-ai/Apertus-v1.5-8B) and [70B](https://huggingface.co/swiss-ai/Apertus-v1.5-70B) releases.

## Getting Involved

The article encourages builders and users to reach out via the Contact page and to participate in community forums on Hugging Face and GitHub. It highlights recurring Swiss AI SME Circle events where small and medium enterprises can meet engineers in person. Multiple engineering positions are open at ETH Zurich and EPFL, with six job links provided.

The page also promotes an "Inside Apertus newsletter" for updates and includes sharing links for Mastodon, Hugging Face, and LinkedIn.

© 2026 ETH AI Center & EPFL AI Center
