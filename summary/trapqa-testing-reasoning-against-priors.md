---
url: https://neohughus.github.io/Understanding_Why_Language_Models_Hallucinate/
title: "TrapQA: Understanding Why Language Models Hallucinate — Testing Reasoning Against Priors"
author: Yangfan Hu, Xuhan Tong, Haoyue Bai, Xi Ding, Shashank Muralidhar Bharadwaj, Siyang Cao, Robert Nowak, Jiawei Zhang
date_fetched: 2026-07-05
date_published: 2026
topics:
  - ai-research-and-models
---

# TrapQA: Understanding Why Language Models Hallucinate — Testing Reasoning Against Priors

## Authors

Yangfan Hu, Xuhan Tong, Haoyue Bai, Xi Ding, Shashank Muralidhar Bharadwaj, Siyang Cao, Robert Nowak, Jiawei Zhang — all affiliated with the **University of Wisconsin–Madison**.

arXiv: 2607.00447, primary classification cs.CL.

## Core Idea

The paper studies hallucination as "inference misalignment" — a mismatch between the answer the prompt supports and the one favored by statistically salient associations. A TrapQA question includes a tempting shortcut and a decisive constraint. The benchmark tests whether models can follow the constraint rather than the shortcut.

Two competing pathways:

- **Shortcut path**: "Salient association → frequent relation → wrong answer"
- **Constraint path**: "Decisive prompt constraint → correct relation → correct answer"

## Motivating Example

The phrase "special relativity" strongly primes models toward Einstein, even when the prompt explicitly states the scientist **did not** formulate special relativity. Authors observed failures in "Claude Sonnet 4.6 and GPT-5.5 Instant." In one run, the model returned Einstein with a confidence of 98, despite the negated constraint pointing to Fermi.

Key diagnostic finding: models can correctly answer isolated probes (e.g., "Did Einstein formulate special relativity?" → Yes; "Did Fermi?" → No) "yet still choose Einstein in the comparative question."

## Dataset

TrapQA has two complementary closed-book settings:

1. **ScientistQA** — 2,925 primary scientist disambiguation questions with three variants:
   - `prepend_names`: names-only, retrieval-sensitive
   - `prepend_profiles`: profiles-in-context control
   - `probes`: two closed-book diagnostic probes per primary question

2. **Real-Life Constrained QA** — 500 everyday two-option scenarios across 13 aspects of life (physical, spatial, procedural, medium-specific constraints)

## Results

- **ScientistQA**: names-only errors are much higher than profiles-in-context errors
- **Real-Life Constrained QA**: error rates reported across model families over 500 scenarios

## Evaluation Methodology

Closed-book evaluation. Authors recommend disabling external tools, web search, retrieval systems, persistent memory, and auxiliary profiles. They caution that "web-chat environments can be unstable because hidden system prompts, memory, prior conversations, and tool availability may affect the result."

Recommended reporting fields: model ID and provider, evaluation date, prompt/config variant, tool setting, reasoning/thinking setting, and answer normalization rule.

## Community Contribution

Authors welcome community-submitted TrapQA examples via Hugging Face Discussions, Google Forms, or Pull Requests. Credit policy: substantial accepted contributions before conference acceptance may lead to authorship; after acceptance, reflected in dataset and arXiv records.

## Citation

```bibtex
@misc{hu2026understandinglanguagemodelshallucinate,
      title={Understanding Why Language Models Hallucinate: Testing Reasoning Against Priors},
      author={Yangfan Hu and Xuhan Tong and Haoyue Bai and Xi Ding and Shashank Muralidhar Bharadwaj and Siyang Cao and Robert Nowak and Jiawei Zhang},
      year={2026},
      eprint={2607.00447},
      archivePrefix={arXiv},
      primaryClass={cs.CL},
      url={https://arxiv.org/abs/2607.00447},
}
```

## Example Questions

- **Hawking vs. Dyson** (correct: Hawking) — constrained by never receiving the Templeton Prize
- **Einstein vs. Zewail** (correct: Einstein) — constrained by never winning the Nobel Prize in Chemistry
- **Hawking vs. Dirac** (correct: Hawking) — constrained by never winning the Nobel Prize in Physics
- **Shuttle inspection pad** (correct: drive the bus) — real-life constrained scenario
- **Rideshare inspection lock** (correct: drive the car) — everyday constraint scenario
- **Original receipt constraint** (correct: hand over a copy) — office shredding practice changes right answer
