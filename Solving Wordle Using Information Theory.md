---
url: "https://orb.binghamton.edu/nejcs/vol8/iss1/6/"
title: "Solving Wordle Using Information Theory"
author: "Talal Aladaileh, Donald Stephens, Mallak Alqaisi, Congyu Wu"
date_fetched: 2026-06-24
date_published: 2026-04
tags:
  - paper
  - information-theory
  - game-strategy
  - complex-systems
---

# Solving Wordle Using Information Theory

Binghamton researchers apply Shannon entropy to Wordle, showing that maximizing expected information gain per guess beats letter-frequency heuristics: >99% win rate vs. ~90%, with fewer guesses on average. The optimal starting word is "tares."

The paper treats Wordle as an information-acquisition problem — each guess partitions 12,972 possible words into up to 243 outcome buckets (3 states × 5 positions). Entropy measures how evenly those buckets split. A flatter distribution means the feedback is more informative on average. Simple, correct, well-validated with exhaustive simulation across all 2,315 solution words.

## Key Quotes

> "The entropy-based strategy both enhances gameplay performance and shows how systems with information-based adaptation follow the same patterns as complex systems that develop through feedback processes and uncertainty reduction."

This is the paper's framing: Wordle as a toy complex adaptive system. Each guess is a probe; the colour feedback is the system's response; the player adapts. The entropy framework quantifies how fast the system moves from disorder to order. It's a neat pedagogical bridge between information theory and complexity science, even if the paper doesn't fully develop the connection.

> "Choosing words with higher expected information (i.e., entropy), rarer situations are encountered and substantially more information is gained."

The counterintuitive core: you want your guess to produce an *unlikely* pattern, not a likely one. If your guess would usually produce "all gray," you learned almost nothing — that was the expected outcome. If it produces a rare pattern, the information gain is enormous. This is why "tares" (flat probability distribution) beats "audio" (peaked distribution).

> "The entropy-based solver functions as a one-step greedy system because it lacks multi-round prediction features or experience-based learning from past matches, which simplifies calculations but leads to occasional difficult situations."

Honest about the limitation. Optimal play requires ~3.42 guesses on average (Bertsimas & Paskov, 2022); greedy entropy can't match that. The gap — roughly 0.5–1 guess per game — is the cost of not looking ahead. Same trade-off shows up in chess engines, pathfinding, and agent planning: information gain now vs. positioning for later.

## Key Themes

- **#information-theory** — Shannon entropy as a decision-making metric: quantify uncertainty reduction, maximize it greedily
- **#game-strategy** — Wordle as a search problem: 12,972-word space, 243 feedback patterns, 6-guess budget
- **#complex-systems** — Wordle framed as a dynamic feedback system where each probe reconfigures the state space
- **#greedy-vs-optimal** — One-step entropy maximization vs. multi-step tree search; the tension between simplicity and optimality

## Critical Analysis

**This is a solid baseline paper, not a breakthrough.** The math is correct, the empirical validation is thorough (full 2,315-word simulation, not cherry-picked examples), and the authors are honest about limitations. It's exactly what academic publishing should produce: a clean, replicable result that others can build on.

**The best part is the visualization, not the math.** Figure 7 overlays probability distributions for "tares" and "audio" — you can *see* why entropy works before you understand the equations. "Audio" has a tall, peaked distribution (most outcomes are low-information); "tares" has a flatter distribution (outcomes are more uniformly informative). This is the kind of explanatory clarity that makes a paper teach rather than just report.

**The complex systems framing is undercooked.** The paper opens by positioning Wordle as a complex adaptive system and entropy as the measure of disorder-to-order transition. This is an interesting lens but the authors don't return to it after the introduction. The word "emergence" never appears. The "adaptation" in "complex adaptive system" is just the player updating their word list — which is correct but thin. This feels like a conference paper that needed a theory section to justify its venue.

**The greedy-vs-optimal gap is the real story.** The entropy method wins >99% of games. Optimal play wins 100% in ≤5 guesses. The difference is entirely about lookahead: entropy maximizes *this* guess's information; optimal play maximizes *all remaining* guesses' information. This is the central tension in any resource-constrained search problem. The paper acknowledges the gap but doesn't quantify it — how often does the greedy choice diverge from the optimal one? That number would tell us whether entropy is "close enough" or "systematically flawed."

**Bottom line**: Read this if you want a clean introduction to applying information theory to concrete problems. Don't read it for the state of the art in Wordle solving. The value is in the method, not the result.

*Source: Northeast Journal of Complex Systems (NEJCS), Vol. 8, No. 1, April 2026. Fetched 2026-06-24.*
