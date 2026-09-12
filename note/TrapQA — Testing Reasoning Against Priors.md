# TrapQA — Testing Reasoning Against Priors

A UW-Madison benchmark that diagnoses hallucination as "inference misalignment" — the gap between what a prompt's constraints demand and what statistically salient associations push toward. TrapQA questions embed a shortcut and a decisive constraint; the test is whether the model follows the constraint or falls for the shortcut.

---

## The Core Mechanism

The paper frames hallucination as a routing problem between two competing pathways:

> "Salient association → frequent relation → wrong answer"

versus

> "Decisive prompt constraint → correct relation → correct answer"

The shortcut path exploits what the model has seen most often in training. The constraint path requires attending to what the prompt actually says. Hallucination isn't fabrication from nothing — it's the model defaulting to the statistically dominant answer when the prompt requires a less common one.

## The Fermi-Einstein Trap

The paper's central demonstration is deceptively simple. A prompt states that a scientist **did not** formulate special relativity, then asks which scientist it is. The phrase "special relativity" is so strongly associated with Einstein that models choose him anyway — with confidence scores of 98 — even though the negated constraint points to Fermi.

The paper reports failures in Claude Sonnet 4.6 and GPT-5.5 Instant.

What makes this a genuine finding rather than a gotcha: models can correctly answer isolated probes ("Did Einstein formulate special relativity?" → Yes; "Did Fermi?" → No) yet still choose Einstein in the comparative question. This is the "inference misalignment" that the paper names: the knowledge is there, but it doesn't survive the comparative framing.

## The Benchmark

TrapQA has two complementary closed-book settings:

**ScientistQA** — 2,925 scientist disambiguation questions with three variants: names-only (retrieval-sensitive), profiles-in-context (control), and diagnostic probes. Results: names-only errors are dramatically higher than profiles-in-context errors. When the model has the relevant facts in context, it handles the constraint. When it has to retrieve from weights alone, the shortcut wins.

**Real-Life Constrained QA** — 500 everyday two-option scenarios across 13 aspects of life. These test whether the same shortcut-vs-constraint dynamic applies beyond trivia: spatial constraints ("the shuttle is in the inspection pad — what do you do?"), procedural constraints, physical constraints.

## Critical Analysis

**The strong claim.** Naming hallucination as "inference misalignment" rather than "generation from nothing" is genuinely useful. It reframes the problem from a defect in the model's knowledge to a defect in how that knowledge is routed during inference. This suggests different fixes — not more training data, but better mechanisms for privileging prompt constraints over statistical defaults.

**The weak claim.** The evaluation methodology caveat — that "web-chat environments can be unstable because hidden system prompts, memory, prior conversations, and tool availability may affect the result" — is doing a lot of work. If the benchmark only works in carefully controlled API calls, it's measuring something real but fragile. The thing it measures may not survive contact with the environments where most people actually use these models.

**What's missing.** The paper demonstrates the problem elegantly but doesn't propose a solution. It's a diagnostic tool, not a therapeutic one. The two-pathway model is compelling as description but doesn't immediately suggest an intervention — you can't just tell the model "pay more attention to constraints" because that's exactly what the prompt already does and exactly what the model fails at.

**The community contribution model is interesting.** Authorship for benchmark contributions is unusual and may be a clever incentive design or a sign that the benchmark needs more diverse questions to be broadly useful.

**The confidence score finding deserves more attention.** Models confidently assert wrong answers. The 98% confidence on Einstein-when-it-should-be-Fermi isn't just a mistake — it's a mistake the model is *sure about*. This is the alignment-shaped hole in the paper.

## Relevance to Agent Design

The TrapQA finding that models answer probes correctly but fail comparative questions has direct implications for agent architecture. [[The Advisor Strategy]] and [[Thrifty (Tiered Delegation for Claude Code)]] both use tiered model routing — but if the failure mode is "model knows the answer in isolation but can't deploy it in context," then routing to a weaker model for verification is routing to a model that will make exactly the same mistake. The [[Sherlock Agent Eval]]'s Theorist-Explorer split may be more relevant here — different roles, not just different capability tiers.

The paper also strengthens the case for [[Guardrails and Feedback Loops]] that are deterministic rather than model-based. If a model can't reliably follow a negated constraint in the prompt, it can't reliably follow a guardrail instruction either. This is the [[Smart Models Dumb Pipes]] argument from a different angle: don't ask the model to enforce constraints it's architecturally prone to ignoring.

---

*Sources: [[summary/trapqa-testing-reasoning-against-priors]], [arXiv:2607.00447](https://arxiv.org/abs/2607.00447)*
*Last updated: 2026-07-05*
