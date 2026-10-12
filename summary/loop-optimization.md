---
url: https://bradleygibbs.dev/posts/loop-optimization/
title: "Loop Engineering: Optimizing Code Generation and Verification"
author: Bradley Gibbs
date_fetched: 2026-10-12
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - agent-coding-workflow
---

# Loop Engineering: Optimizing Code Generation and Verification

A long, methodical treatment of the verification side of AI-assisted development, built around three nested loops. Gibbs' premise: faster code generation shifts the bottleneck to verification, so the craft is designing processes that qualify candidates for human review cheaply and route scarce human attention to the claims only humans can judge.

## Terminal evaluation

The outer structure is **terminal evaluation**: an autonomous system attempts a task, then an *independent* evaluator checks the result against an agreed **conformance contract** — the obligations (what the code must do, preserve, never permit) approved by humans before the run. Checks, expected results and stopping rules live outside the builder's authority; "the builder's completion message does not determine qualification, and missing evidence cannot count as a pass." Terminal verdicts establish capability and reliability of a *configured system*, but have low diagnostic resolution — a pass says nothing about how much wasted work it took.

## Engineering the inner loop

Inside the build loop, Gibbs distinguishes two error sources needing different responses: **environmental** errors (missing prerequisites — context, obligations, tools that never reached the stage) and **agent failure modes** (the agent had sufficient, accurate, usable information and still erred). Three activities improve the path: PREPARE each stage deliberately (responsibility, obligations, tools, handoff discipline), CHECK intermediate work (observe before intervening; run detectors in observation mode), and DECIDE/CORRECT when evidence warrants — verifying each correction behaviorally. He cites the *Failure as a Process* study (57.9% of decisive errors were epistemic, i.e. mishandling information the agent already had) while carefully cataloguing why that number can't be transferred to his loop. Meta's Wink A/B test (5.3% fewer tokens, 4.2% fewer engineer interventions) shows targeted intervention can help, without establishing correctness.

## The learning loop

The third loop treats each check and correction as a mechanism with a cost and two rates to estimate: detection rate and correction rate. Evidence flows from five sources (design, build, evaluation, review, post-release defects), and the loop's output is a bounded, testable proposal — which mechanisms to keep, narrow, revise or retire. It gives "the model eating the harness" a practical test: does the vendor agent still need this support under our conditions? Humans approve loop changes.

## Honesty as a feature

Notably, the article refuses to report measured savings from its own twelve months of pipeline use, defines its illustrative examples as hypothetical, and repeatedly states what its evidence does *not* show. That epistemic discipline is unusual in this genre.
