---
url: https://alexweil.github.io/sherlock-agent-eval/
title: "How good a detective is an AI?"
author: Alex Weil
date_fetched: 2026-07-05
date_published: 2026
---

# How good a detective is an AI?

**Author:** Alex Weil
**Source:** https://github.com/alexweil/sherlock-agent-eval (code under Apache-2.0; article & diagrams under CC BY 4.0)

## Origin Story

The project began when the author and friends played *Sherlock Holmes Consulting Detective*, an open-ended deduction board game. They fell into a trap: fixating on an obvious victim while missing that the real undercover agent was a living woman the killer was still hunting. The clue that would've cracked it — the killer returning to scan a passenger list post-murder — was noticed but misinterpreted. The author identified this as a failure of "second-order" inference.

## Core Thesis

The author turned the board game into an evaluation benchmark for LLM agents. Claude Fable 5 tied Holmes in hard mode (questions hidden until after investigation). But the headline score is less important than the two failure modes identified and what fixes them.

## Why a Board Game Makes a Good Eval

The game sidesteps common benchmark problems:

- **The solution is physically hidden** — upside-down in the booklet, never in the agent's workspace
- **Information has a price** — visiting locations costs points; thinking and re-reading are free
- "It rewards comprehension, not retrieval" — clues must be assembled into a coherent story
- A **deterministic Game Master** (plain Python, not an LLM) serves clues verbatim; the solution lives outside the agent's reach
- A separate validator cross-checks logs against answers post-hoc

The author calls the setup "cheat-resistant, not cheat-proof," noting potential pre-training data leakage.

## Two Failure Modes

### Failure 1 — Execution: Preferring generated content over retrieved content

Claude Fable 5 found the real undercover agent name in a served clue and wrote it into notes, then at answer time crossed it out and substituted a self-constructed anagram. This happened in two clean single-pass runs. A third run (restarted mid-game due to rate-limiting) resumed as a fresh agent reading only externalized notes — it kept the correct name by "trusting a fact in its notes over a freshly-generated guess."

The author generalizes: "recency plus a bias toward self-generated content beats recalled fact."

### Failure 2 — Comprehension: The decoy trap

The obvious suspect (a murdered former detective) is a stand-in. The real answer is the living woman still being hunted. "Escaping it — reading a clue as a behavior, noticing it contradicts the obvious story" is the second-order inference almost everyone misses. A single-agent "methodical detective" prompt fell for this trap **9 times out of 9**.

## Interventions and Results

### What Didn't Fully Work

| Intervention | Result |
|---|---|
| Generic "good investigator" prompt | Cleaned up process (fabrication disappeared) but comprehension unchanged — trap held 9/9 |
| Revealing questions upfront (soft mode) | Model-dependent: Sonnet 4.6 cracked it; Opus 4.8 didn't benefit; Haiku 4.5 couldn't use it |

### What Worked: Split Comprehension from Exploration

The author introduced a **duo architecture**:

- **Theorist** (comprehension engine): No access to the world (no `grep`, no visiting, no GM). Maintains a model of the world, labels facts with sources, hunts loose ends, tries to falsify its own hypothesis. Re-spawned fresh each turn from an externalized ledger. The case-model "is a document the Theorist rewrites, not a state it holds."
- **Explorer** (perception/action engine): Has the workspace and GM. Takes loose-ends, resolves names to addresses, visits, relays clues verbatim. Forbidden from drawing conclusions.
- **Conductor**: A pure verbatim pipe between them.

The duo's Theorist produced this second-order inference:

> "They tortured the sister for hours to extract an identity. If the dead brother were the infiltrated agent, they'd already have him"

...and concluded the agent is alive. The author clarifies this isn't about connecting facts the monolith couldn't — both had the same clues. "What differs is the *prior* that stitching runs under" — the Theorist's job is to falsify, not defend.

## Evidence Matrix

| Configuration | Description | Escaped decoy trap? |
|---|---|---|
| **Baseline** | One agent per model, no scaffolding | Mixed: Fable 5 escaped; Haiku/Sonnet/Opus fell |
| **Methodical-prompt monolith** | One agent (Opus 4.8) with investigator instructions | Fell 9/9 |
| **Clean-context monolith** | One agent (Opus 4.8), re-spawned fresh each turn | Fell 3/3 |
| **Reasoner + explorer duo** | Opus 4.8 reasons, Sonnet 4.6 explores | Broke 2/2 |
| **Same duo, weaker reasoner** | Sonnet 4.6 in both roles | Broke 1/1 |

Each is N=3 per model unless otherwise noted.

### The Author's Interrogation of Their Own Conclusion

Two alternative explanations were tested and ruled out:

1. **Clean context alone?** Built a "clean monolith" — single agent re-spawned fresh each turn — fell 3/3. "One run even visited the shipping office, *saw* the killer still hunting, and *still* concluded the dead man was the agent."
2. **Just using the smarter model (Opus 4.8)?** Ran the duo with Claude Sonnet 4.6 (weaker) in both roles — it broke the trap.

Final claim: "Model capability alone was neither necessary nor sufficient."

## Architecture & Isolation Details

- The agent's directory holds only permitted material; GM internals and solution live outside; prompt forbids leaving
- GM is plain Python (not LLM), serves clues verbatim, logs everything, never holds the solution in memory
- Separate validator cross-checks served log against answers; knowledge with no served origin gets the run discarded
- Tell of leakage would be naming the hidden solution without being given it — "that never appeared"

## Honest Limitations

- **One case only**: "The decoy trap is a single instance of second-order reasoning in a single case."
- **The bottleneck moves**: Solving comprehension revealed an exploration-coverage problem. The duo understood the plot but missed the agent's literal name behind an unpulled thread.
- **Mundane matters**: A Unicode-normalization bug in `grep` (accent-sensitive search silently returning nothing) cost more points than the scaffolding earned.
- **Score is noisy; binary result isn't**: LLM-as-grader introduces ±25 point variance on same answers; author stands behind the binary trap-or-not finding.

## Lessons for Agent Builders

1. **Retrieved beats generated — but agents don't believe it.** The deepest failure is overriding a retrieved fact with a self-generated guess.
2. **For comprehension, topology is orthogonal to model size.** A role-split fixed models that fell solo, using a *weaker* model in the reasoning seat. "Doing the investigation instills a pull toward the obvious reading."
3. **Bottlenecks are layered** — fixing comprehension surfaces exploration-coverage problems.
4. **Watch your judge and your `grep`** — noisy graders and accent-sensitive search moved more points than flashy failures.

## What's Next

- Replicate on a different case (kill the single-case caveat)
- "Naming-completeness" pass so the duo stops leaving named actors unidentified
- Longer-horizon cases, red-team of isolation, non-Anthropic models in the same harness

## Model Notes

The author used Anthropic's models for their "clean capability ladder." Claude Fable 5 was temporarily disabled by Anthropic on June 12, limiting Fable 5 results to at most three playthroughs. Findings "are about agent topology, not any one vendor or model."

## Game Source

The case comes from [Sherlock Holmes Consulting Detective: Baker Street Irregulars](https://www.spacecowboys-games.com/game/the-baker-street-irregulars/), published by Space Cowboys. Case material is paraphrased, not reproduced.
