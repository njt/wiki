---
url: https://www.openappa.com/
title: "What is OpenAPPA"
author: OpenAPPA
date_fetched: 2026-10-11
date_published: unknown
topics:
  - security-and-sandboxing
  - guardrails-and-feedback-loops
---

OpenAPPA is a deterministic, MIT-licensed, vendor-agnostic guardrail engine for AI agents, backed (per its own claims) by a NeurIPS-accepted paper, that aims to be 100% resistant to data exfiltration from prompt injection or hallucination without breaking agent task completion. The landing page is a manifesto for why the alternatives fail.

Its first argument is that **non-deterministic guardrails miss the problem by design**. The industry's answer to approval fatigue — a second model judging each tool call (Claude Code's auto mode, Codex's auto-review) — cannot track data flow across tool calls, and since classifiers are themselves prompt-injectable, harnesses deliberately hide tool outputs from them. The judge never sees the data. And even the best classifier tops out around 99.3%, which at millions of calls is a lot of breaches.

Its second argument is that **pattern-based deterministic guardrails either break agents or don't work**: blacklisting `rm -rf /` just makes the model write a Python one-liner; regex lists give zero auditable coverage; rule sets end up either so tight the agent stalls or so intricate nobody can verify what they permit.

The alternative: OpenAPPA **tracks flows instead of matching patterns**. It runs outside the agent's prompt and execution loop (unseeable, unnegotiable), driven by a declarative configuration describing data sources, audiences, trust levels, and authorities. Every trajectory carries a security label — `audience × trust` — that only ever gets more restrictive (reading a private repo narrows audience; reading an unvetted web page lowers trust); each tool-call decision is derived algebraically from the label. "You cannot prompt-inject an algebra." Coverage over the whole tool graph can be validated in CI/CD.

To keep strict enforcement from killing utility, the engine returns a machine-readable **remedy plan** instead of a bare "forbidden": sanitizers that transform payloads (mask secrets, redact PII) so they can flow to a wider audience, authorities that approve one specific action without lifting session restrictions, and subagents that isolate an untrusted read in a disposable branch. Claimed benchmark result: task completion lifted from 37% to 90% versus strict enforcement.
