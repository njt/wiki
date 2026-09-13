---
url: https://helloitsliam.com/2026/09/08/advanced-prompt-injection-techniques/
title: "Advanced Prompt Injection Techniques"
author: Liam (helloitsliam.com)
date_fetched: 2026-09-13
date_published: 2026-09-08
topics:
  - security-and-sandboxing
  - guardrails-and-feedback-loops
---

The closing installment of a tutorial series on probing LLM security boundaries, covering three technique families that avoid obviously malicious prompts. **Multi-turn injection ("salami slicing")** spreads an objective across conversations: the worked example collects apple varieties, spices, and sweeteners through individually innocent baking questions, then asks ChatGPT to summarize "what makes Grandma Evelyn's famous apple pie so special" — reassembling the protected recipe without ever requesting it. **Translation testing** asks the model to "translate" a document containing a placeholder the model must first fill in, testing whether changing languages or task framing changes how safety controls apply. **Structured-format testing** submits a JSON `system_override` payload claiming a security audit, probing the misconception that JSON/XML/YAML carries more authority than ordinary language.

The article's empirical finding is that modern frontier models generally resist all three: ChatGPT protects the recipe across the multi-turn arc, refuses to invent the missing content before translating, and treats the JSON as ordinary user input rather than trusted configuration. The interesting observation is that safety appears to be evaluated at the level of conversation-wide intent rather than the individual prompt — which the author notes is considerably harder to build and to filter for.

The conclusion insists the series is a methodology rather than a checklist — the goal is learning to systematically evaluate security boundaries, not memorizing prompts that work against one model — and concedes that enterprise systems (RAG, agents, external APIs, business workflows) introduce attack surfaces beyond simple prompting that these consumer-ChatGPT exercises never touch.
