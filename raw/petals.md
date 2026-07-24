---
url: https://petals.dev/
title: Petals — Decentralized LLM Inference at Home
author: BigScience Research Workshop
date_fetched: 2026-07-25
date_published: unknown
---

# Petals: Distributed LLM Inference at Home

Petals is a project from the BigScience research workshop (affiliated with Hugging Face) that enables running large language models locally by distributing the workload across a peer-to-peer network — described as "BitTorrent‑style" model serving.

## Supported Models

- Llama 3.1 (up to 405B parameters)
- Mixtral (8x22B)
- Falcon (40B+)
- BLOOM (176B)

Users can both generate text and fine-tune models using consumer-grade GPUs or Google Colab.

## How It Works

A participant loads only a portion of a model onto their own hardware. They then join a network (status visible at `health.petals.dev`) of other users who serve the remaining parts.

### Performance

Single-batch inference benchmarks cited on the page:
- ~6 tokens/second for Llama 2 (70B)
- ~4 tokens/second for Falcon (180B)

Described as "enough for chatbots and interactive apps."

## Philosophy

The project positions itself as more flexible than traditional LLM APIs. Users can "employ any fine-tuning and sampling methods, execute custom paths through the model, or see its hidden states." Combines "the comforts of an API with the flexibility of PyTorch and 🤗 Transformers."

## Recognition

- Covered by TechCrunch (December 2022)
- Chatbot available at `chat.petals.dev`
- Active Discord community for development

## Participation

GPU owners can contribute compute by connecting their hardware. Colab notebook available for quick experimentation. GitHub for documentation. Mailing list with updates "once a few months."
