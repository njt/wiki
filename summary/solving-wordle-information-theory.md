---
url: "https://orb.binghamton.edu/nejcs/vol8/iss1/6/"
title: "Solving Wordle Using Information Theory"
author: "Talal Aladaileh, Donald Stephens, Mallak Alqaisi, Congyu Wu"
date_fetched: 2026-06-24
date_published: 2026-04
journal: "Northeast Journal of Complex Systems (NEJCS), Vol. 8, No. 1, Article 6"
doi: "10.63562/2577-8439.1146"
pdf: "https://orb.binghamton.edu/cgi/viewcontent.cgi?article=1146&context=nejcs"
tags: ["information-theory", "shannon-entropy", "wordle", "game-strategy", "complex-systems", "search-space-reduction"]
topics:
  - misc
---

# Solving Wordle Using Information Theory

## Source

Binghamton University, School of Systems Science and Industrial Engineering. Published in the Northeast Journal of Complex Systems (NEJCS), Vol. 8, No. 1 (2026), as part of the CSMA 2026 Special Issue.

PDF: https://orb.binghamton.edu/cgi/viewcontent.cgi?article=1146&context=nejcs

## Abstract

The study applies Shannon entropy to guide word selection in Wordle, aiming to maximize information gain per guess. The authors report that entropy-based word selection improves performance compared to a heuristic approach based on letter distribution. The work is positioned as a baseline for implementing Shannon entropy in different games.

## Key Details

### Methodology

- **Word pool**: 12,972 valid five-letter words (Wordle's dictionary), with 2,315 candidate solution words
- **Entropy calculation**: For each word, compute probabilities for all 3^5 = 243 possible letter-state outcomes (gray/yellow/green per position), then calculate Shannon entropy H(S) = -Σ P[s] × log₂(P[s])
- **Greedy strategy**: Always pick the word with highest entropy given current constraints; re-compute after each guess based on the reduced solution space
- **Baseline comparison**: Letter-frequency heuristic (pick words containing the most common letters: 'a', 'e', 'r')
- **Validation**: Exhaustive simulation across all 2,315 solution words

### Key Finding: "tares" is the optimal starting word

Entropy analysis of all 12,972 valid words shows "tares" has the highest expected information gain. This matches intuition — it uses common letters in favorable positions without duplicate letters.

### Results

- **Entropy strategy**: >99% win rate within 6 guesses, lower average guess count
- **Baseline (letter-frequency) strategy**: ~90% win rate, higher average guess count
- **Failure mode of baseline**: Gets trapped in anagram cycles (e.g., "least", "stale", "slate", "steal" — same five letters, different arrangements)

### Limitations (authors' own)

- The entropy method is **greedy** — it only looks one step ahead, not multi-step optimal
- Optimal dynamic programming solvers (Bertsimas & Paskov, 2022) guarantee solution in ≤5 guesses with ~3.42 average — the entropy method doesn't match this
- Assumes uniform word probability; doesn't weight by English usage frequency
- Doesn't exclude previously-used Wordle answers from consideration
- Requires the full word list upfront (can't be applied without it)

## Full Text Notes

The paper opens by framing Wordle as a "dynamic feedback system in which each guess will provide information that will affect the subsequent guesses." This is the core insight — Wordle isn't just a word game, it's an information-acquisition problem where each guess is a query designed to maximize uncertainty reduction.

The 243-state outcome space (3^5) is the key mathematical structure. Each guess partitions the remaining word list into at most 243 buckets. Entropy measures how evenly those buckets are sized — higher entropy means more uniform partition, which means the feedback is more informative on average.

The letter distribution plot (Figure 2) shows 'e' dominates (~912 words contain it), while 'j', 'q', 'x', 'z' are rare. ~70% of words have five unique letters (Figure 3), which explains why "tares" (five distinct common letters) is optimal.

The authors explicitly compare their probability distributions for two words — "audio" (tall/peaked distribution = lower entropy) vs. "tares" (flatter distribution = higher entropy). This is a nice visualization of why entropy works: you want rare (high-information) outcomes, not common (low-information) ones.

## Critical Analysis

This is a clean, well-executed undergraduate/graduate-level application of Shannon entropy to a concrete problem. The math is correct and the empirical validation is thorough (full 2,315-word simulation). It's exactly what you'd want from a "baseline" paper — the authors are honest about limitations and don't overclaim.

**What it gets right**: The explanation of entropy is accessible without being dumbed down. The probability distribution overlay (Figure 7) is genuinely illuminating — you can see why "tares" beats "audio" without understanding the equations. The full simulation approach (rather than cherry-picked examples) gives confidence in the results.

**What's missing**: The authors acknowledge but don't really grapple with the gap between their greedy entropy approach and optimal play (Bertsimas & Paskov's 3.42-average solver). The difference — ~0.5-1 guess per game — is the cost of only looking one step ahead. This is the same tension that shows up everywhere from chess engines to agent planning: greedy information gain vs. multi-step strategy. The paper would be stronger if it quantified exactly how often the greedy choice diverges from the optimal choice.

**The interesting meta-point**: This paper positions Wordle as a complex adaptive system — each guess provides feedback that changes the system state. The entropy framework provides a quantitative measure of how "organized" the game state becomes. This framing connects Wordle to broader complex systems theory, which is presumably why it's in NEJCS rather than a games journal. The connection is gestured at but not fully developed — the paper doesn't return to complex systems theory after the introduction.

**What a practitioner should take away**: If you want to write a good Wordle bot, use entropy. If you want to write the *best* Wordle bot, you need dynamic programming or tree search — entropy alone leaves performance on the table. The same trade-off applies to any information-acquisition problem under a budget constraint.
