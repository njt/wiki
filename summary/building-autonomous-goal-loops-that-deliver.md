---
url: https://jx0.ca/building-autonomous-goal-loops-that-deliver/
title: Building Autonomous Goal Loops That Deliver
author: Unknown
date_fetched: 2026-09-04
date_published: undated
topics:
  - agent-architecture
  - guardrails-and-feedback-loops
---

# Building Autonomous Goal Loops That Deliver

by an unnamed author on jx0.ca, a follow-up to their earlier "The Convergence Problem" post.

The problem: first-generation agent loops — give the agent a plan, run until tests pass, return failures — only work when tests describe the job. They fail when the job is to grow a product capability nobody understands yet. The agent keeps *moving*, but movement isn't delivery: "a screen can look complete while it stores nothing."

## The harness, not the retry prompt

What's missing between the agent's turns and the human's rounds is a **harness** that does three things: expose a real failure, locate the missing capability, and preserve the lesson after the session ends. That's "a different machine from an agent with a retry prompt."

## Two systems, engineered separately

The obvious system is the feature; the hidden one is the process that decides what to change next. Four roles keep them apart: the **development agent** changes the feature; the **driver** approaches it through the same surface a user would; the **scorer** reads what happened; the **controller** picks the next gap. The product agent gets a fresh conversation and only the tools the product exposes — if it shares the development agent's context, the test is worthless.

## Everything important sits outside the prompt

The loop needs a reproducible starting point (data, services, model, environment, served code all matching the recorded run), a preflight to prove the reset worked, and **real demand** — a small corpus of human-approved requests sent unchanged through a fresh interaction. The persistent *effects* matter most, not the visible result.

## Floor vs. direction

Every loop needs a **deterministic floor** — tests, validators, schemas, effect checks that hold already-earned behavior. The floor starts green; a red floor becomes the next job. But the floor can't say what to build next. Direction comes from driving a real request and inspecting the shortfall. Signals are noisy: one perfect run proves success is *possible*, not *reliable*.

## One round closes one gap

Restore fixture → run floor → send one request → collect behavior/result/effects → classify the largest gap → close it through every required layer → add a deterministic check → re-send. "One gap does not mean one file. It means one causal claim." The path runs `request → representation → operation → persistent effect → visible proof`.

## The scorer is part of the threat model

A loop optimizes whatever it can reach. The authority model has three tiers: **Free** (normal levers), **Propose** (cross a boundary, human decides), **Frozen** (the exam itself — corpus, fixture, rungs, score rules, rubric). The loop may *add* a regression check but never *weaken* one, and never change a product lever and its measure in the same round.

## Three loops at different speeds

The **product loop** closes one capability a round. The **harness loop** changes when repeated evidence exposes a development failure — one question: *"What cost time that a rule or check could prevent?"* The **direction loop** sits outside both: a person reads a batch of evidence and decides to continue, redirect, or stop.

## The repository is the control plane

State lives in a small `goals/` package — `goal.md`, `facts.md`, `plan.md`, `LOOP.md`, `STATE.md`, `corpus/`, `rounds/` — so sessions are disposable without making the work forgetful. A new agent reads the goal, procedure, state, and evidence, then continues from the same decision.

The closer: most tickets don't need any of this. If complete tests describe the work, give the agent the tests. The harness earns its cost only when *use* must reveal the capability sequence — and the measure of quality is what remains after a round, not how long the loop ran.
