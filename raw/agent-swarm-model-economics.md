---
url: https://cursor.com/blog/agent-swarm-model-economics
title: "Agent swarms and the new model economics"
author: Wilson Lin
date_fetched: 2026-07-21
date_published: 2026-07-20
---

# Agent swarms and the new model economics

Cursor Blog, Research category. 17 min read.

## Trees and leaves

The swarm has two roles organized around tree-like task decomposition. **Planner agents** (powered by top models) split goals into pieces and delegate. **Worker agents** (faster, cheaper models) execute those pieces. The swarm's shape grows to match the problem's contours rather than imposing a fixed topology.

> "The design is a superset of more rigid orchestration systems."

The approach generalizes to building a browser, solving math problems, optimizing GPU kernels, finding vulnerabilities, raising test coverage, and generating synthetic training data.

## What the tree does for memory

A single agent handling a full task must walk the entire tree, holding ancestors, position, and goals in context simultaneously.

> "We think this explains why long-running single agents drift."

In the swarm, planners never implement (context stays free of low-level detail) and workers never plan (context is fully devoted to one narrow piece). The authors cite economist Ronald Coase's theory of the firm — coordination costs grow faster than work itself, leading to bounded tiers.

## A version control system for agents

The earlier browser swarm peaked at ~1,000 commits/hour on Git. The new system hits ~1,000 commits/second. Cursor built a new VCS from scratch to enable this throughput and make collisions visible.

## Failure modes at 1,000 commits per second

Five failure modes identified:

1. **Split-brain design** — Two planners unknowingly implement the same concept differently. Fix: planners make decisions directly and ensure no two subtrees decide the same question.

2. **Contention between planners** — Planners aware of each other fight over the same files. Fix: shared design docs with compile-checked references; a reconciler merges contradictory docs.

3. **Merge conflicts** — Workers collide on files but can't absorb other agents' context to merge. Fix: a neutral third-party agent intervenes to resolve conflicts impartially.

4. **Megafiles** — Popular files grow bloated, becoming expensive to transport/diff/merge and sites of constant collisions. Fix: workers flag bloated files, new commits block, and an outside agent decomposes them.

5. **Ossification** — Agents avoid touching core code even when needed. Fix: "license intentional breakage" — agents can make focused patches outside scope with explanatory comments; the compiler propagates changes downstream.

> "The compiler carries the change through the rest of the system."

## Review lenses

Multiple review approaches were tested — full transcripts, output-only, codebase-only, different models/personalities.

> "No single lens catches everything, but decorrelated lenses stack."

Review compute is described as high-return since it's much cheaper than the work it audits. This stacked review system is cited as a major contributor to sustained quality.

## Letting agents shape the environment

The concept of **stigmergy** (from ant/termite colonies) is introduced — organisms coordinate by shaping the environment, which in turn shapes the next organism. The **Field Guide** is a self-authored folder whose `index.md` is injected into every agent at start, curated by agents with only a line budget constraint.

> "model weights are frozen, so it's precisely surprise encounters that are worth capturing"

## The SQLite experiment

The swarm was instructed to implement the entire 835-page SQLite manual in Rust. Source code, test suites, SQLite binary, and internet access were withheld. Progress was graded against **sqllogictest** (millions of queries with known answers). The swarm was unaware of the test suite's existence. Human reviewers checked for cheating.

### Results across model mixes

Four configurations tested:
1. **GPT-5.5** as both planner and worker
2. **Grok 4.5** as both planner and worker
3. **Opus 4.8** planner + **Composer 2.5** worker
4. **Fable 5** planner + **Composer 2.5** worker

The new harness outperformed the old in every mix. The Fable 5 hybrid passed ~2/3 of the suite within the first hour. New runs scored 73–85% at the 4-hour cutoff; old runs scored 11–77%. Every new configuration eventually passed 100%.

GPT-5.6 Sol was originally intended but produced "runaway spirals unlike anything the other models produced" and was replaced with GPT-5.5.

### A deep dive into the runs

The old Grok 4.5 run produced 68,000 commits in 2 hours (~70× the new run's pace), but most were thrash. The old run accumulated >70,000 merge conflicts (accelerating); the new run had <1,000 over 4 hours. The hottest file in the old run had 7,771 conflicts touched by 1,173 agents; in the new run, the most contested file saw 47.

> "the old swarm's biggest coordination failure — split-brain"

The old run sprawled to 54 Rust crates (including 3 separate SQL packages); the new run settled on 9 crates. In the Fable 5 mix, the old swarm needed 64,305 lines of engine code to pass the suite; the new one used 9,908. Opus mix: 19,013 lines at 97% (old) vs 4,645 lines at 100% (new).

## Model economics

Costs ranged from **$1,339** (Opus 4.8 hybrid) to **$10,565** (GPT-5.5 alone). Workers carried at least 69% of tokens (over 90% in most runs). However, planner tokens cost more — in the Opus/Composer mix, Opus produced few tokens but ~2/3 of the cost.

> "Few moments in a large task genuinely require frontier intelligence."

GPT-5.5 worker costs alone: $9,373. Opus planner + Composer 2.5 workers: entire worker fleet cost $411. Fable 5 used fewer planning tokens than Opus (despite 2× per-token price) but its workers consumed many more tokens, making the total run more expensive.

## Specs as prompts

Each AI capability jump raised the abstraction level: autocomplete → line, early models → block, agents → file/feature, swarms → spec.

> "What was scarce in this experiment... is the right description of intent."

The swarm is compared to a compiler — translating intent (analogous to source code) into executable work through intermediate steps. The difference: compilers preserve meaning deterministically while the swarm is probabilistic at every step. The entire post describes efforts to close that gap.

The public codebase is available at github.com/cursor/minisqlite (solo Opus 4.8 run).

## Footnotes

1. Solo runs of Opus 4.8 and Fable 5 were conducted for cost comparison but only informally graded; their costs appear as hatched bars in the chart.
2. GPT-5.6 Sol was deselected due to sensitivity to literal wording causing "runaway spirals"; GPT-5.5 was used instead to avoid prompt-tuning confounds.
