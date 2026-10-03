---
url: https://cactuscompute.com/blog/whistle
title: "Whistle: Speech Recognition for Mobiles, Wearables, Robots, Smart Home, Automotive and Microcontrollers"
author: Cactus Compute
date_fetched: 2026-10-03
date_published: 2026-10-03
topics:
  - local-and-open-source-inference
  - ai-research-and-models
---

Cactus Compute releases Whistle, a 16.9 MB on-device speech recognition model that runs on CPU with no dependencies and loads into the same C++ engine ("Needle") as their text models. It does three jobs: transcription (7 European languages, 30 s clips), word timestamps from decoder attention, and raw speech embeddings without decoding.

The architecture is deliberately shared with Needle's language models: an 8-block encoder of "Simple Attention" blocks (mHC residual lanes, Monarch Hadamard MLPs), and an 8-block laddered decoder with grouped-query attention (8 query to 2 KV heads), engram lookups, and one speech-specific addition per layer — a gated cross attention reading the encoder, whose K/V projections run once per clip and are cached, so 5-beam search costs five transcript caches rather than five audio passes.

Benchmarks: Whistle leads Whisper base on LibriSpeech (clean/other), SPGISpeech, Earnings-22 and FLEURS average; Whisper wins TED-LIUM, AMI and MLS — at 145.3 MB vs 16.9. Time to first token tracks clip length (5.9 ms at 5 s, 36.3 ms at 30 s) because no padding to 30 s is needed. Deployment is the real story: prebuilt binaries for 17 targets from iOS to RISC-V to WASI, a three-function C API, no environment variables, and a `.cact` container format that lets one binary do speech, text, or transcription-then-tool-calling in a single pass.
