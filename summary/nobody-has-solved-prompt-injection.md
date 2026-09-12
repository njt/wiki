---
url: https://fabraix.com/blog/nobody-has-solved-prompt-injection
title: "Bounding the Blast Radius: A Survey of Prompt-Injection Defenses for LLM Agents"
author: Ibrahim Abdu
date_fetched: 2026-07-08
date_published: 2026-06-26
topics:
  - security-and-sandboxing
---

A survey of prompt-injection defenses for LLM agents, arguing that no single
defense solves the problem and the pragmatic goal is to make exploitation
costlier than the value an attacker can extract.

The root cause is architectural: unlike SQL, where parameterized queries
enforce a code/data boundary in the engine, a language model concatenates
trusted instructions and untrusted data into one token stream, processed by
one set of weights, with no mechanical separation. A formal separability
result (Zverev et al., 2025, ICLR) confirms that prompt engineering and
fine-tuning do not substantially close this gap.

The survey organizes defenses into four layers by where they sit in the
input-to-action path:

**Prompt-level formatting.** Delimit or mark untrusted spans as data — from
fixed XML-like tags (trivially escapeable) to randomized per-request markers
and spotlighting. Nearly free to deploy but advisory, not enforced; a
persuasive payload can still win.

**Model-level training.** Train the model to prioritize trusted sources, e.g.
OpenAI's instruction hierarchy (system > developer > user > tool output) or
StruQ's structured-input tuning. Measurably raises baseline robustness but
remains probabilistic — adaptive attacks degrade it, and the tool-calling
pathway often lags behind the chat pathway in alignment.

**Input filtering.** Place a classifier or LLM guard in front to block
malicious inputs (Llama Prompt Guard 2, Lakera Guard, Azure Prompt Shields).
Evadable through character-level perturbations; multi-turn attacks like
Crescendo distribute payloads across individually benign messages; even
sub-1% false-positive rates impose friction at scale.

**Output and trajectory monitoring.** Inspect the agent's behavior rather than
its inputs — from passive output guards (NeMo Guardrails) to trajectory-aware
systems (LlamaFirewall) that audit reasoning steps and gate tool calls before
they execute. Catches what input layers miss (e.g., the EchoLeak exfiltration
pattern) but adds latency and cost.

The central empirical claim: adaptive attacks devastate static evaluations. A
multi-org study (Nasr et al., 2025) subjected twelve recent defenses to
gradient-based search, RL, and human red-teaming — most reporting under 5%
attack success on frozen corpora rose above 90%. Human-guided exploration was
the single most effective strategy, the one component a fixed corpus cannot
represent.

The recommended approach is economic, not absolute: compose layers under a
deployment's latency and cost budget, verify the composite against an
adaptive adversary operating in the deployment's own environment, and aim for
the point where breaking the system costs more than the attacker can gain. The
article anchors this to real 2025 incidents — EchoLeak (CVE-2025-32711),
Agentforce "ForcedLeak," and others — and references the authors' Adversarial
Cost to Exploit (ACE) methodology for measuring that threshold.
