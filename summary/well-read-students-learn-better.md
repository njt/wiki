---
url: "https://arxiv.org/abs/1908.08962"
title: "Well-Read Students Learn Better: On the Importance of Pre-training Compact Models"
author: "Iulia Turc, Ming-Wei Chang, Kenton Lee, Kristina Toutanova"
date_fetched: 2026-07-03
date_published: 2019-08-23
source_type: academic-paper
venue: "arXiv:1908.08962 (cs.CL)"
doi: "10.48550/arXiv.1908.08962"
topics:
  - ai-infrastructure-and-hardware
---

# Well-Read Students Learn Better: On the Importance of Pre-training Compact Models

**Authors:** Iulia Turc, Ming-Wei Chang, Kenton Lee, Kristina Toutanova (Google Research)

**Submitted:** August 23, 2019; revised September 25, 2019

## Abstract

Recent developments in natural language representations have been accompanied by large and expensive models that leverage vast amounts of general-domain text through self-supervised pre-training. Due to the cost of applying such models to down-stream tasks, several model compression techniques have been developed (e.g., distillation, pruning, quantization). However, the simple baseline of just pre-training and fine-tuning compact models has been overlooked. In this paper, the authors show that pre-training remains important in the context of smaller architectures, and fine-tuning pre-trained compact models can be competitive to more elaborate methods proposed in concurrent work. They then propose Pre-trained Distillation, a simple yet effective and general algorithm that transfers knowledge from a large fine-tuned model to a smaller one via standard distillation, bringing further improvements. The paper studies the interaction between pre-training and distillation under two variables that have been under-studied: model size and properties of unlabeled task data. A key finding: these two techniques exhibit a compound effect even when sequentially applied on the same data. The authors released 24 pre-trained miniature BERT models to accelerate future research.

## Key Contributions

1. **The overlooked baseline**: Pre-training compact models from scratch + fine-tuning is competitive with elaborate compression methods (distillation from large pre-trained models, pruning, quantization).

2. **Pre-trained Distillation**: A two-stage algorithm — pre-train a compact model on general-domain text, then distill task knowledge from a large fine-tuned teacher. Standard KD loss, no architectural tricks.

3. **Compound effect**: Pre-training and distillation compound even when applied sequentially on the same data — surprising because each step sees the same information, yet they stack.

4. **24 released models**: Pre-trained miniature BERT architectures at various sizes, enabling research that doesn't require starting from BERT-base.

## Method

The approach is straightforward:
1. Pre-train a compact BERT variant on general-domain text (standard MLM + NSP objectives)
2. Fine-tune on downstream tasks
3. Optionally apply Pre-trained Distillation: use a fine-tuned BERT-large teacher, distill into the pre-trained compact student using standard KD on logits

The key experimental variables:
- **Model size**: Varying hidden dimensions and layer counts
- **Unlabeled task data**: Properties of the data used for distillation (domain match, quantity)

## Results

- Pre-trained compact models match or exceed the performance of models compressed via distillation from large pre-trained teachers
- Pre-trained Distillation adds further gains on top
- The compound effect holds across model sizes and data regimes
- Simple pre-training + fine-tuning is a strong baseline that had been overlooked in the compression literature
