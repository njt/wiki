---
url: https://eve.security/blog/what-security-teams-need-to-know-about-agents-using-abliterated-models
title: "What Security Teams Need to Know About Agents Using Abliterated Models"
author: Eve Security
date_fetched: 2026-10-26
date_published: 2026 (approx., post-July 2026)
topics:
  - security-and-sandboxing
  - local-and-open-source-inference
---

Eve Security on abliteration — surgically removing a model's refusal training from open-weight models like GLM-5.3, Llama, or Qwen — and what it means for security teams. Because abliteration requires the weights themselves, it only works on models you can run yourself; API-only models are immune. The post opens with the July 2026 OpenAI/Hugging Face sandbox-escape incident as motivation and cites Anthropic testing showing abliterated GLM-5.3 achieved 100% compliance with harmful cyber-attack instructions.

The core governance argument: the model's safeguards are only one piece. Security outcome depends on the whole agent — task, tools, context — so control must live in **a neutral party observing and restricting the agent**. Model-level safeguards judge whether content should exist at all; policy instead enforces scope and authorization at the action layer. But policy has a known limit: static rules are written in advance, and an abliterated model is good at finding paths a rule didn't anticipate. Eve proposes "agent interrogation" — pausing unusual, high-risk actions and requiring the agent to justify itself, with the justification checked against real context (e.g. a verified change ticket).

On Shadow AI: an abliterated model connection can hide in a local Python binary making an HTTPS call to a model API hostname — invisible in agent inventories or Cursor settings. Detection means tying API calls to process, user, and destination; blocking happens through the EDR plus a globally-propagated policy, written at agent speed and shared across customers.
