---
url: https://genai.owasp.org/llmrisk/llm01-prompt-injection/
title: "LLM01: Prompt Injection"
author: OWASP GenAI Security Project
date_fetched: 2026-09-13
date_published: undated
topics:
  - security-and-sandboxing
  - agent-architecture
---

The canonical entry for prompt injection in OWASP's Top 10 for LLM Applications. A prompt injection vulnerability occurs when user-supplied prompts alter LLM behavior or output in unintended ways — and, critically, the input need not be human-readable at all, "as long as the content is parsed by the model." The entry splits the vulnerability into direct injections (the user's own prompt derails the model, intentionally or not) and indirect injections (instruction-bearing content arriving from external sources such as websites, files, and retrieved documents). It draws a definitional line between prompt injection generally and jailbreaking specifically: jailbreaking is the subset where the attacker makes the model disregard its safety protocols entirely, and it requires ongoing updates to training and safety mechanisms rather than just prompt-level safeguards.

The listed impacts span disclosure of sensitive information, leaking system prompts and infrastructure details, content manipulation, unauthorized access to the LLM's available functions, arbitrary command execution in connected systems, and manipulation of critical decisions. A section on multimodal AI warns that instructions hidden in images accompanying benign text expand the attack surface in ways current defenses barely address.

Seven mitigation strategies follow: constrain model behavior; define and validate output formats with deterministic code; input/output filtering including the RAG Triad (context relevance, groundedness, answer relevance); privilege control and least privilege — giving the application its own API tokens and handling extensible functions in code rather than handing them to the model; human approval for high-risk actions; segregating and clearly denoting untrusted external content; and adversarial testing that treats the model as an untrusted user. The entry is candid that these are mitigations only: "it is unclear if there are fool-proof methods of prevention for prompt injection." Nine example attack scenarios (direct, indirect, unintentional, RAG poisoning, a real CVE in an email assistant, payload splitting, multimodal, adversarial suffixes, multilingual/obfuscated payloads) close it out, alongside references and MITRE ATLAS mappings (AML.T0051.000/.001, AML.T0054).
