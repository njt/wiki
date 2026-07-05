# LLM Guard

A security toolkit that sits between your application and the LLM, scanning both inputs (prompts) and outputs (responses) for harmful content, data leakage, prompt injection, and other risks. 15 input scanners, 20 output scanners, MIT licensed.

---

## Key Quotes

> "By offering sanitization, detection of harmful language, prevention of data leakage, and resistance against prompt injection attacks" -- ensures secure LLM interactions.

## Key Themes

#guardrails #security #agent-architecture

LLM Guard works at the prompt/response boundary rather than the execution boundary (contrast with [[Navaris]] and [[OpenSandbox]]). The scanner taxonomy is comprehensive:

**Input scanning**: anonymize PII, ban code/competitors/topics, detect gibberish and invisible text, catch prompt injection, find secrets, assess sentiment and toxicity

**Output scanning**: everything above plus bias detection, JSON validation, factual consistency, malicious URL detection, reading time estimation, and URL reachability checks

The layered approach is right: no single scanner catches everything, but running 15+ independent checks on inputs and 20+ on outputs creates defense in depth. The anonymization/deanonymization pair is particularly useful -- strip PII before sending to the LLM, then restore it in the response.

Connects to [[dotnet Slopwatch]] (catches LLM shortcuts at the code level while LLM Guard catches them at the prompt/response level), [[OneCLI]] (credential security pairs with prompt security), and the broader agent guardrails discussion.

## Critical Analysis

LLM Guard is a necessary layer for production LLM applications. The scanner approach is extensible and composable, which is the right architecture for a rapidly evolving threat landscape. The main limitation is latency -- running 15 input scanners adds overhead to every request. The `pip install llm-guard` simplicity is good for adoption, but production deployments will want the API server for throughput. The MIT license is important for enterprise adoption. The 2.9k stars suggest healthy traction. What's missing is discussion of false positive rates -- aggressive scanning can make LLMs unusable, and tuning the balance between safety and utility is the real operational challenge.

---
*Sources: [[summary/llm-guard]]*
*Last updated: 2026-05-14*
