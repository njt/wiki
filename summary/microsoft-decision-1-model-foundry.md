---
url: https://commandline.microsoft.com/microsoft-decision-1-model-foundry/
title: "Microsoft-Decision-1: a new decision model for fast, low-cost decision scoring"
author: Microsoft (Command Line blog)
date_fetched: 2026-10-10
date_published: 2026-10-10
topics:
  - ai-research-and-models
  - agent-architecture
---

Microsoft launches Microsoft-Decision-1, a "decision model" — a model class built not to generate text but to return structured, calibrated decisions that software acts on immediately. It is post-trained from Qwen3.5-9B for fast single-pass decision scoring, available in Microsoft Foundry and via OpenRouter, with plans to rebase on MAI and OpenAI models. It targets routing, classification, prioritization, verification, and workflow control through a structured API that scores a fixed set of answer options (yes/no, multiple-choice, ratings, rubric grading) with probabilities.

The launch makes four headline claims against rivals (Quyet-1.0-Large, Surogate Rune 26B, GPT-6 Luna Decisions, deck-31B, H2O-Lightning-4B): first on average accuracy across 36 benchmarks kept blind from training (~147K questions); fastest measured, 4.5× quicker than Quyet-1.0-Large and 35× quicker than GPT-6 Sol at p50 85 ms; 1.3% decision-flip rate under eight perturbation types; and second on calibration (0.9 behind Quyet). A safety eval across 11 benchmarks (5,250 requests incl. jailbreaks and prompt injection) claims refusal of harmful requests with high utility.

Internal use cases do the selling: Xbox Research labeled 10,000+ pieces of feedback 14× faster and 200× cheaper than GPT-6 Sol; the Copilot team found it competitive with GPT5.6 Luna at 100× the speed; Microsoft Discovery's adaptive replanning loop scored 46× more consistent than an LLM-based score at 3× speed. A long use-case list covers agent controls, model routing, AI judging, incident routing, and computer use. Pricing: $0.042/M input tokens, output free — with the benchmarked scale math landing at ~$2,434 (GPT-6 Sol) vs ~$11 (Microsoft-Decision-1) per million texts.

The framing essay is the interesting part: "decision models are quickly emerging as an important new category in AI," and with agentic AI real, "decision models have the potential to help people guide and control those agents through complex environments" — cost is now a first-class driver of model choice, and the right-model-for-the-right-job argument gets a new tier.
