# Leaky Language Models

Majidi, Mireshghallah & Taram (Purdue/CMU) demonstrate that fine-grained per-token generation timing — the very thing that makes streaming LLM APIs feel responsive — leaks both deployment optimizations and architectural details to any observer who can time API responses. The paper recovers layer counts, hidden dimensions, and attention heads from black-box API access, and shows definitively that Google Gemini Flash uses speculative decoding with a ~128K-token draft context window. This is the first systematic study proving that latency optimization and model confidentiality are in direct tension.

---

> "Model architecture and deployment strategies are valuable assets that provide competitive advantage."

This framing matters. The paper treats architecture secrecy not as an abstract privacy concern but as a business asset in the global AI race. The authors are explicit: providers optimize for low-latency streaming, and that optimization IS the attack surface. You can't fix this without making the product worse.

## Two Attacks, One Signal

### Attack 1: Speculative Decoding Detection

The insight is beautifully simple. Speculative decoding uses a smaller draft model to propose tokens cheaply; the large model verifies them in batches. But draft models have shorter context windows — when you exceed the draft's limit, it goes blind, every proposal gets rejected, and generation slows to the main model's speed.

The attack crafts prompts requiring progressively longer context (a number-memory task with variable padding), then watches for the sudden latency spike that says "the draft model just ran out of window." The spike's position reveals the draft model's context length.

**What they found in the wild:**

- **Gemini Flash 2.5** and **Flash 2.5 Lite**: speculative decoding with ~128K draft context
- **Gemini Flash 1.5**: speculative decoding with ~32K draft context
- OpenAI, Cohere, and Mistral APIs showed no detectable pattern — though the authors are careful to note this could be false negatives from different architectures or more sophisticated draft strategies

> "If a prompt requires a context longer than the draft model's maximum context length, the draft model produces incorrect predictions, tokens are rejected, and latency falls back to the large model's timing."

The attack was responsibly disclosed to Google (Nov 2025, acknowledged Jan 2026), which means Google sat on this finding for months without obvious mitigation. That's not Google dropping the ball — it's that the mitigations are genuinely painful.

### Attack 2: Architecture Recovery

This is the paper's technical centerpiece. The authors build a timing predictor that models transformer inference down to individual matrix multiplications, then use it as an oracle to search ~1,500 candidate architectures and rank which one best matches observed API timings.

The predictor is a hybrid: analytical asymptotic terms (O(TH²) for a Q-projection, etc.) combined with linear regression coefficients learned from instrumented open-source models. Critically, they also model cuBLAS's dynamic kernel selection — a non-linearity that naive approaches miss — using a two-phase ensemble of LightGBM classifier + per-kernel random forest regressors.

**How well it works:**

| Target | Layers (top-5) | Hidden dim (top-5) | Both (top-5) |
|--------|-----------|--------------|---------|
| Llama 3.2 3B (Eager) | 86% | 72% | 65% |
| Llama 3.2 3B (Flash2) | 98% | 100% | 55% |
| Remote: Llama 3.1 8B | rank #1 | rank #4 | rank #28 |

The remote API result is the one that matters: against a real deployed model (Weights & Biases' Llama 3.1 8B), they recovered hidden dimension as the #1 candidate and got within striking distance of the full architecture. They only needed the API key and a stopwatch.

Cross-family generalization is surprisingly strong: a predictor trained on Llama 3.2 1B recovered Phi-3.5-mini's architecture at 80% joint accuracy in the top 5. The modeling generalizes because it's built on primitives (matrix multiply, attention), not model-specific profiling.

## Why This Matters

The paper lands at a moment when frontier model architectures are increasingly treated as trade secrets. OpenAI, Google, and Anthropic disclose less about each new model. This isn't paranoia — architecture IS competitive advantage, determining the cost-quality Pareto frontier.

The attack vector is hard to close:

- **Constant per-token timing** would require making every generation as slow as the worst case, "potentially increasing latency by several times." That's the product, not just a setting.
- **Buffering** tokens into constant-interval batches kills the streaming UX that users expect. For latency-sensitive applications, this is a non-starter.

The authors conclude, correctly, that "defending against these timing side channels without degrading latency or user experience remains an open and difficult challenge." This isn't the usual academic hedge — it's a structural tension between making models fast (which providers must do to compete) and keeping their internals private (which they increasingly want to do).

## Critical Analysis

The paper's strongest contribution is methodological, not empirical. The hybrid analytical-empirical predictor — decomposing transformer runtime into named primitive operations, then learning coefficients from real hardware — is more interesting than any single recovered architecture. It's a general technique that survives GPU generations and model families. The cuBLAS kernel-selection corrector in particular is the kind of detail that separates a working attack from a toy model: naive polynomial regression gives NRMSE 0.411; the corrected version hits 0.119.

The weakest link is the threat model's GPU-knowledge assumption. For the full architecture attack, the adversary needs to know which GPU class the target runs on, and that's not always inferable from API behavior alone. The authors acknowledge this but don't quantify how much accuracy degrades with wrong GPU assumptions.

The speculative decoding attack is the one that most worries me. It requires no special access, no GPU knowledge, no offline training phase — just an API key and a loop that gradually lengthens prompts. Any competitor can run this against any streaming API today. The architecture attack takes more setup but is still well within reach of a motivated team.

The paper's framing as "the first systematic study" is fair. Prior work either needed extra API surface (logit bias from Carlini et al.) or focused on fingerprinting rather than architecture recovery. The through-line — that streaming token delivery IS the side channel — reframes latency optimization itself as a confidentiality decision.

What's conspicuously absent: any discussion of whether the recovered architectures have actual competitive value, or whether they're just interesting. Knowing that Gemini Flash 2.5 uses speculative decoding tells you something about Google's inference stack, but does it tell you enough to replicate or undercut it? The paper proves the leak exists; it doesn't prove the leak matters economically.

---

*Sources: [[raw/leaky-language-models]]*
*Last updated: 2026-07-25*
