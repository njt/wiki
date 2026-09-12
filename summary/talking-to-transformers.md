---
title: "Talking to Transformers"
url: https://miraos.org/blog/2026/05/02/talking-to-transformers
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

Taylor's four pillars for effective prompting of LLMs:

1. Clear Intent with Domain-Specific Language -- concise, focused prompts rather than verbose context dumps. Mirrors "an eccentric millionaire dictating to an unpaid intern." Distinguishes reasoning models from non-reasoning "nothink" models, recommending the latter for predictable pipeline tasks where "every token is an instruction."

2. Attention Management -- prompting operates as a zero-sum attention budget. Irrelevant tokens distract from critical information. "Once the model commits to that very first token you're along for the ride." Frontloading directives and leveraging the model's existing training patterns proves more effective than fighting base training.

3. Concept Compression -- models function as "universal translators" across domains. Rather than spending tokens explaining hyperparameter tuning, instruct it to "tune it like a carburetor" -- the model maps the metaphor to your technical need.

4. Rigorous Output Review -- treat AI as "MASSIVE autocomplete." Don't accept substandard outputs; rewind and craft better prompts. "Absolute accountability" -- poor outputs reflect the prompter's clarity, not model incompetence.

"The model attaches to and interprets every single word you use. The more words you use the higher the chance of misinterpretation."

"Do not turn your brain off when talking to a transformer."