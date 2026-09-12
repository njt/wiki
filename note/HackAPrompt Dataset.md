# HackAPrompt Dataset

A dataset of 100K-1M prompt injection attempts from a global hacking competition, tested against GPT-3, FlanT5-XXL, and ChatGPT. Accepted at EMNLP 2023. Contains the actual prompts, completions, expected outputs, and success/failure flags -- making it one of the few large-scale "in the wild" adversarial datasets for LLMs.

---

## Key Quotes

> "Exposing Systemic Vulnerabilities of LLMs Through a Global Prompt Hacking Competition" -- paper title, which is also a prompt injection attempt (the title instructs readers to ignore it)

## Key Themes

#security #prompt-injection #adversarial-ml #datasets #evaluation

This is the empirical counterpart to the theoretical security concerns in [[Security]]. Instead of reasoning about what attacks might work, you get a dataset of what attacks actually worked, with difficulty levels, token counts, and model-specific success rates. The competitive format (lower token counts = better scores) naturally selects for elegant, minimal attacks -- the kind hardest to defend against.

14+ downstream models have been trained on this dataset for prompt injection detection, which is both the obvious application and a nice demonstration of the "generate attacks to train defenders" loop. See also [[Benchmark Exploitation]] for what happens when the evaluation itself becomes the attack surface.

## Critical Analysis

Strong: real adversarial data from real attackers is irreplaceable for security research. The competition format incentivized creative attacks. MIT licensed and publicly available.

Weak: the evaluated models (GPT-3, FlanT5-XXL, ChatGPT circa 2023) are now two generations old. The attack surface of modern models is significantly different -- tool calling, system prompts, and multi-turn context create new injection vectors that this dataset doesn't cover. Still, the fundamental attack patterns (ignore previous instructions, context manipulation, role-playing exploits) remain surprisingly durable.

The dataset is gated behind contact info sharing, which is a minor friction for researchers and a non-issue for serious attackers.

---
*Sources: [[summary/hackaprompt-dataset]]*
*Last updated: 2026-05-14*
