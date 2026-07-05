---
url: https://web.archive.org/web/20260522230330/https://rosmine.ai/2026/05/18/fixing-llm-writing-with-distribution-fine-tuning/
title: "Fixing LLM writing with Distribution Fine Tuning"
author: Rosmine (Ben Rosmine)
date_fetched: 2026-07-05
date_published: 2026-05-18
site: Rosmine ML Blog
---

# Fixing LLM writing with Distribution Fine Tuning

**Abstract/TLDR:** LLMs are notoriously formulaic at writing, overusing certain tokens or phrases. Rosmine shows that models trained with SFT fail to match the distribution of the training data by using Maximum Mean Discrepancy (MMD), Judge Model Quality (JMQ), and L2 Token Distribution. To fix this, they created a new training algorithm, Distribution Fine Tuning (DFT), an LLM post-training step that makes the distribution of model outputs better match the training distribution (improving MMD by 49% and JMQ by 63%). The model trained with DFT is much better at writing than an SFT baseline, improving creativity scores by +164%, as well as coherence (+28%), clarity (+16%), meaningful detail (+146%) and it does not have any overused "slop signs" like too many emdashes, or "it's not X, it's Y".

A demo (14B param model) is available at https://dft.rosmine.ai/

Models trained with DFT have much more human writing style — a sample of 100 model outputs scored as 100% human written by Pangram AI detector.

## Key Metrics: Quantifying output quality

Instead of measuring "quality" itself, which is not well defined, Rosmine measures similarity to human writing samples using three metrics:

1. **N-gram Token distribution L2 distance**: Captures word choice similarity, useful for detecting overuse of certain words/phrases. Uses L2 (euclidean) distance — KL or JS Divergence don't work well because tokens appearing in one distribution but not the other have outsized contribution.

2. **Maximum Mean Discrepancy (MMD, Gretton)**: Gets embedding for each text sample via Llama-embed-nemotron-8B and computes a distance between the embedding distributions using a Gaussian RBF kernel. Measures content similarity — captures if outputs are overly generic or overuse certain concepts.

3. **Judge Model Quality (JMQ)**: Uses GPT-5.4-mini as judge with randomized prompt order to prevent positional bias. JMQ = 2 × win rate for model outputs (optimal is 1.0, meaning 50% win rate — indistinguishable from human).

## The Problem: SFT is not all you need

Starting from Qwen3 instruct-tuned models (base models had poor instruction following), Rosmine trained on a subset of 185K Fineweb samples. Even without RLHF or other post-training steps that might cause reward hacking, SFT models fail to match the training data distribution.

Graphs show MMD, JMQ, and L2 Token Distance across temperatures — SFT never reaches 0 MMD or 1.0 JMQ at any sampler setting. The proposed explanation: SFT focuses on training individual samples and misses distribution-level information. DFT trains at this higher level.

Potential explanations referenced: exposure bias (Ranzato, Bengio), unreliable tail probabilities (Holtzman), likelihood objective (Welleck), miscalibration (Braverman), local typicality (Meister).

## Sample Model Outputs

Comparing SFT T=0.7, T=1.0, and DFT on the same prompt:

- **SFT T=0.7**: Repetitive structure — most sentences start with the same word, text is generic without deeper details
- **SFT T=1.0**: Big incoherent transitions, random non-English characters (Chinese and Korean) — at T=0.9, over 9% of outputs have non-English characters
- **DFT**: More natural, varied structure with deeper detail

## Results: "Super Baseline" vs DFT

The "super baseline" takes the best metric value over ALL hyperparameter configurations (learning rates, LoRA vs full fine-tuning, sampler settings) — making it better than any single model could achieve. DFT uses a single model with fixed hyperparameters.

| Model | MMD↓ | JMQ↑ | Token L2↓ |
|-------|------|------|-----------|
| 4B SuperBaseline | 0.047 | 0.27 | 0.0040 |
| 4B DFT | 0.025 | 0.40 | 0.0042 |
| 8B SuperBaseline | 0.041 | 0.37 | 0.0040 |
| 8B DFT | 0.023 | 0.56 | 0.0031 |
| 14B SuperBaseline | 0.037 | 0.49 | 0.0039 |
| 14B DFT | 0.018 | 0.80 | 0.0036 |

A 4B DFT model beats a 14B superbaseline at MMD and an 8B superbaseline at JMQ. All training done on a local 6× 6000 Ada server.

## Fine-Grained Judge Model Analysis

| Dimension | 14B Baseline | 14B DFT |
|-----------|-------------|---------|
| Clarity | 70.5 | 82.0 |
| Coherence | 54.5 | 70.0 |
| Creativity | 32.5 | 86.0 |
| Depth | 35.5 | 87.5 |
| Prompt Relevance | 44.0 | 75.0 |

Biggest gains in Creativity (+164%) and Depth (+146%).

## Next Steps

DFT is proprietary. Rosmine is offering a beta model training service (1-2 collaborations initially). Plans for open weights model and larger model. Interested in expanding beyond web content to creative writing, emails, movie scripts, etc.

## Unverified Hype/Speculation

DFT is not specific to writing — could replace SFT for more accurate outputs, or apply to audio for AI-generated music. No experimental results yet for other use cases.

## Limitations

- All DFT training used LoRA (not full fine-tuning)
- Larger models show better scores — scaling helps but existing large models still have clear LLM-generated style
- Trained on Fineweb subset — good for blogs/news articles, unlikely to match creative writing performance

## Anti-Slop Considerations

Key argument: "LLMs are not the cause of slop. Lack of effort/care is." If you spend days researching and planning a blog post with a detailed outline, ChatGPT output will be interesting even with em-dashes.

Mitigations implemented:
- Input requires prompt, outline, writing style, and use case — forcing thought before generation
- Injected random fruit/cute animals into output to prevent blind copy-paste
- No public API to prevent automated use

Future vision: an "IDE for writing" with fine-grained control, editing, and automated quality checks ("unit tests, but for writing"). Currently: "LLMs for writing are like GPT-4 for coding. People think that LLMs help them write, but it's actually just adding bugs faster."

## Prior Work

- **MAUVE** (Pillutla) — saturates at .997+ even for 4B baseline
- **FID with BERT** (Alihosseini) — tested but Frechet metrics assume Gaussian distributions
- **TextGail** (Qingyang, Ho) — outperformed SFT at 64 tokens but failed at 1024 due to training instabilities
- **IQLearn** (Wulfmeier) — could beat SFT for specific metrics at specific sampler settings, but no single setting beat the super baseline

## Key Findings from Appendices

- **Token overuse**: Top 10 tokens contribute 87.2% of the L2² value — mostly common tokens becoming more common ("the", ".", "is", "The", "a", "to", "that", "are", "was", "of")
- **Data size**: SFT plateaus at 185K samples — quarter→half data shows improvement, half→full data shows plateau/regression
- **Output diversity**: DFT self-BLEU (0.063 for 14B) nearly matches human reference (0.061), much better than SFT T=0.9 (0.079)
- **Slop signs**: DFT token overuse patterns match human-human baseline noise. GPT-5.4 overuses "corridors" (45.2×), "norms" (43.1×), "align" (36.0×) — DFT has no such outliers
- **Other models compared**: Claude 4.6, Gemini 3.1 Pro, Kimi 2.5, GPT 5.4 all score far worse on MMD and Token L2 (expected — different training data). JMQ favors them (judge models prefer LLM outputs)
- **Repetitiveness**: 53.3% of SFT T=0.7 outputs have 3+ consecutive sentences starting with the same word vs. 17.4% human and 18.6% DFT
- **Non-English characters**: 36.45% of SFT T=1.0 outputs contain non-English characters vs. 0.1% human and 8.1% DFT

## References

Key citations: Gretton (MMD), Babakhin (Llama-embed-nemotron-8B), Yang (Qwen3), Penedo (Fineweb), Hu/Lialin (LoRA/ReLoRA), Pillutla (MAUVE), Qingyang/Ho (TextGail/GAIL), Wulfmeier (IQLearn), Ranzato/Bengio (exposure bias), Holtzman (neural text degeneration), Welleck (unlikelihood training), Braverman (calibration), Meister (locally typical sampling), Laurito (AI-AI bias), Panickssery (LLM evaluators favor own generations), Read ("Elara Voss"), OpenAI ("Where the goblins came from"), Pangram (AI detector), Zhu (self-BLEU), Heusel (FID), Popović (ChrF), Papineni (BLEU)
