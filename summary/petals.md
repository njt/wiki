---
url: https://petals.dev/
title: "Petals — Decentralized LLM Inference at Home"
author: BigScience Research Workshop
date_fetched: 2026-07-25
date_published: unknown
---

Petals lets people run large language models on consumer hardware by splitting the model across a peer-to-peer network. Instead of one machine holding the entire model, each participant serves a slice, and the network stitches responses together — the project calls it "BitTorrent-style" model serving.

It supports models up to Llama 3.1 405B, Mixtral 8x22B, Falcon 40B+, and BLOOM 176B. Users can run inference or fine-tune, including from Google Colab. Benchmarks cite ~6 tokens/second for Llama 2 70B and ~4 t/s for Falcon 180B on single-batch inference, which they describe as usable for chatbots and interactive apps.

The project comes from the BigScience workshop (affiliated with Hugging Face) and pitches itself as more flexible than hosted APIs — you control sampling, fine-tuning, and can inspect hidden states, while still getting an API-like experience via PyTorch and Hugging Face Transformers integration.

There's a chatbot at `chat.petals.dev`, an active Discord, and a network health dashboard at `health.petals.dev`. GPU owners can donate compute; mailing list updates come every few months.
