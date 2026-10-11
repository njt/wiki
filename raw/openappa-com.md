---
url: https://www.openappa.com/
date_fetched: 2026-10-11
---

# What is OpenAPPA

OpenAPPA is a frontier deterministic AI guardrail that is 100% resistant to data exfiltration caused by prompt injection or model hallucination, and the first of its kind that doesn't break agents.

Backed by a NeurIPS-accepted paper, it is open, vendor-agnostic, and MIT-licensed.

And yes, it outperforms competitors on benchmarks:

## Non-deterministic guardrails miss the problem#

The industry's answer to approval fatigue is a second model that judges each tool call: Claude Code's auto mode, Codex's auto-review, and other auto-modes.

By design, they cannot track data flow across tool calls. Because classifiers are prompt-injectable themselves, harnesses hide tool outputs from them, so the judge never sees the data at all.

Because of their probabilistic design, even the best top out at 99.3%: at millions of calls, 0.7% is a lot of breaches.

## Other deterministic guardrails either break agents or don't work#

Blacklisting commands against an LLM is a dead end. Block `rm -rf /` and the model writes a Python one-liner; block `curl` and it pushes to an external git remote. A list of regexes also gives zero visibility into coverage: nobody can verify that it closes every path, or that three mundane tools chained together don't leak.

Rule sets end up either so tight they break the agent or so intricate nobody can audit what they permit.

## OpenAPPA tracks flows instead of matching patterns#

OpenAPPA is a cross-platform, pluggable engine driven by a single configuration. It runs outside the agent's prompt and execution loop, so the model cannot see, negotiate with, or manipulate it, and it plugs into an existing agent loop in one place.

Instead of allowed and blocked tools, the configuration describes data sources, audiences, trust levels, and authorities. Every trajectory carries a security label, `audience × trust`: reading a private repo narrows the audience, reading an unvetted web page lowers trust. The label only ever gets more restrictive, and the engine derives each decision from it algebraically.

An injected prompt telling the agent to leak secrets is irrelevant: you cannot prompt-inject an algebra. Because the configuration is declarative, you can validate in CI/CD that your whole tool graph is covered. The configuration is data-specific, so you can scale to millions of agents without changing it.

## It's full of tricks to help agents accomplish their tasks#

Strict enforcement is where utility usually dies: a bare "forbidden" makes an agent stall, retry, and fail. OpenAPPA instead returns a machine-readable remedy plan with the ways the agent may legally proceed:

- **Sanitizers**transform the payload, masking secrets or redacting PII, so it can flow to a wider audience. Stock sanitizers ship in the box; custom ones, including model-based ones, plug in with a clear blast radius.
- **Authorities**approve one specific action, through a human or an internal API, without lifting the session's restrictions for later calls.
- **Subagents**isolate an untrusted read in a disposable branch, so the parent trajectory continues unpoisoned.

This is what lifts task completion from 37% to 90% on our benchmarks and makes deterministic security practical.
