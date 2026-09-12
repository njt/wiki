---
url: https://cacm.acm.org/blogcacm/goodharts-law-comes-for-every-benchmark-you-trust/
title: "Goodhart's Law Comes for Every Benchmark You Trust"
author: Alex Williams
site: Communications of the ACM (Blog@CACM)
date_published: 2026-07-28
date_fetched: 2026-08-06
topics:
  - guardrails-and-feedback-loops
---

# Summary: Goodhart's Law Comes for Every Benchmark You Trust

Alex Williams surveys the accelerating crisis in AI benchmark integrity. The central diagnosis: when a measure becomes a target, it ceases to be a good measure — Charles Goodhart's 1975 insight about monetary policy now applies with full force to the benchmarks that decide which AI companies raise money, which models enterprises buy, and which press releases get written.

The evidence is now quantitative, not anecdotal. Three mechanisms dominate:

**Contamination.** GSM1k (Zhang et al., 2024) commissioned 1,205 fresh math problems never seen on the public web. Leading model accuracy fell by up to 13%, with the Phi and Mistral families showing systematic overfitting. The correlation between verbatim memorization of GSM8K and score gaps (Spearman's r² = 0.36) confirms this is memorization, not mathematics. Separately, Gema et al. found 6.49% of MMLU questions contain errors — wrong answer keys, ambiguous phrasing, unanswerable questions — and correcting the errors changes model rankings.

**Adversarial gaming.** "The Leaderboard Illusion" (Singh et al., 2025) analyzed two million Chatbot Arena battles and found that private best-of-N testing breaks the statistical assumptions of the rankings. Meta tested 27 private Llama-4 variants and published only what it chose; selecting the best score from N attempts makes the leaderboard assume each model is one honest sample when it isn't. Two identical checkpoints submitted under different names scored 17 points apart.

**Benchmarketing.** Providers select temperature, prompting strategy, and few-shot configuration to maximize headline numbers, and those settings rarely match anyone's production defaults. Saturation does the rest: MMLU, HumanEval, and HellaSwag are effectively ceilinged at ~90%. Each new benchmark becomes a target, which contaminates and saturates in turn, with each cycle shorter than the last.

Williams offers a constructive turn: private, refreshed test sets are the only intervention that attacks the mechanism itself. Contamination detection is table stakes but lags gaming. The most useful advice for deployment decisions: evaluate on your own data. Every public score should be treated as a marketing claim that happens to carry decimal places.
