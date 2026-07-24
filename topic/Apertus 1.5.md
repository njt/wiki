# Apertus 1.5

Swiss AI's Apertus 1.5 is a major open-weight model release from the ETH AI Center and EPFL AI Center, continuing pretraining of the earlier Apertus 1.0 at 8B (4T additional tokens) and 70B (2T tokens) scales. It adds native image understanding, switchable reasoning mode, a 262K-token context window, and improved tool use — all while publishing weights, training data, and full training details.

---

## Key Quotes

> "Fully open: open weights, open data, open values, and full training details."

This is the strongest openness commitment in the current landscape. Most "open" releases stop at weights; Apertus publishes the training data and methodology too. In a field where [[Open models lag state-of-the-art closed models by 4 months]] and [[The Open-Weight Deceleration Thesis]] argues openness kills frontier investment, Apertus is a counterexample: a well-resourced, institutionally-backed open release that isn't just weights.

> The context window has been quadrupled to 262,144 tokens compared to Apertus 1.0.

That's ~200K words — enough to ingest entire codebases or book-length documents. Context expansion is the brute-force answer to the retrieval problem, and Apertus is betting on it alongside [[Recent Developments in LLM Architectures]] that tackle long-context inference cost. The gap between "can read 262K tokens" and "can reason across 262K tokens" is where the real evaluation lives, and the promised technical report will need to address it.

> "Thinking Mode" — a switchable reasoning mode that allows the model to reason about the input before answering.

Switchable reasoning is becoming table stakes. [[Controlling Reasoning Effort in LLMs]] maps the design space: RLVR with length penalties, discrete vs. continuous effort levels. Apertus 1.5 enters a field where DeepSeek R1, GLM-5.2, and others have already shipped reasoning toggles. The open question is whether Apertus's implementation is competitive or checkbox-level.

## Key Themes

- **#tool** — Apertus 1.5 as a fully open model with image understanding, reasoning, and tool use
- **#concept** — Openness as a spectrum: weights < data < methodology < charter. Apertus claims the full stack
- **#pattern** — Continued pretraining as a release strategy: extend existing models with multimodal data rather than train from scratch
- **#comparison** — Sits alongside [[Bonsai 27B]] (phone-scale openness), [[Granite 4.1]] (enterprise openness), [[Cohere North Mini Code]] (coding-specific openness), [[Hy3]] (Apache 2.0 MoE openness), and [[GLM-5.2 Is the Step Change for Open Agents]] (regulatory-catalyst openness)

## Critical Analysis

**The good:** Full-spectrum openness is genuinely rare and valuable. Most "open" models are weights-only releases that are impossible to reproduce, audit, or extend. Publishing training data and methodology alongside weights makes Apertus a scientific artifact, not just a product. The Swiss institutional backing (ETH Zurich, EPFL) gives it credibility that startup "open" releases lack — these aren't venture-funded models that will vanish when the funding dries up.

**The concerning:** The technical report is "coming in the coming weeks." Without benchmarks, training pipeline details, and intermediate checkpoints, we can't assess whether the 4T/2T token budgets were well-spent or whether the multimodal training degraded text performance. The 262K context claim needs throughput and needle-in-haystack numbers to be meaningful. "Improved instruction-following" and "improved tool use" are assertions without evidence until the report lands.

**The strategic bet:** Apertus is positioning as the European counterweight to US and Chinese open-weight models. The Swiss AI SME Circle events and ETH/EPFL job postings suggest this isn't just a model release — it's institution-building. The question is whether European academic AI can keep pace with the industrial training budgets of Meta, xAI, DeepSeek, and Alibaba. The 4T-token training run for the 8B model is substantial but not unprecedented; the 70B at 2T tokens is more conservative.

**What's missing:** No mention of agentic capabilities, which is where [[State of Open Source AI 2026]] identifies the harness as the new frontier. No discussion of inference cost or hardware requirements — can this run on consumer GPUs or does it need datacenter hardware? And the experimental "spoken language processing" addition is the kind of feature that either becomes the headline six months from now or gets quietly dropped.

**Bottom line:** Apertus 1.5 is a credible, well-positioned open model release with full-spectrum openness that few competitors match. Its real value depends on the technical report and whether the Swiss ecosystem can sustain the training cadence to stay competitive. Worth tracking, but not yet worth switching workflows for.

---
*Sources: [[raw/apertus-1-5]]*
*Last updated: 2026-07-25*
