---
url: https://github.com/Aevonix/FlashAlign
title: "FlashAlign — ReAlign models instantly, on demand"
author: Aevonix Research
date_fetched: 2026-11-02
date_published: 2026-10-30
topics:
  - local-and-open-source-inference
  - security-and-sandboxing
---

FlashAlign (Aevonix, v1.1.1) applies alignment profiles — in practice, abliteration-style refusal-reduction transforms — to open-weight models per request, without permanently modifying the weights. Profiles are fitted to specific pinned checkpoints (DeepSeek V4.1 Flash 1017, GLM 5.3 Full/Flash, Qwen3.8-27B, Gemma 4 26B A4B) and load into supported engines (TensorFold, vLLM, SGLang, llama.cpp); switching among already-loaded profiles happens per request.

The distinctive design is governance rather than algebra. Two activation modes: **Verified contacts** (default) applies the profile only to messages from an approved channel/account/contact, proven by a harness-native identity adapter that binds rendered text and media to authenticated ingress — text claiming to be an approved sender grants nothing. **Always On** applies the profile to every routed request including tool and subagent calls. Tool inheritance is controllable per tool name, so `delegate_task` can be excluded from profile propagation.

The engine contract is a signed per-request API: `owner_mode` profile name plus `owner_mode_scale` in (0,1], authenticated by `X-Owner-Mode-Auth: v1:<ts>:<nonce>:<HMAC-SHA256>` over the exact body, with a 60-second timestamp window and a persistent nonce replay store whose failure must fail closed. The loopback proxy signs every final wire body, recognises exactly four engine 400 messages by full-string match (only "no profiles" demotes to a stock retry), and keeps stock and profile-mode model caches on separate identities. Benchmarks show small decode slowdowns (2–10%) from active transforms.
