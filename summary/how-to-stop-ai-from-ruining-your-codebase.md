---
url: https://coding-is-like-cooking.info/2026/08/how-to-stop-ai-from-ruining-your-codebase/
title: "How to Stop AI from Ruining Your Codebase"
author: Ivett Ördög (interview), Modern Software Engineering channel
site: coding-is-like-cooking.info
date_published: 2026-08
date_fetched: 2026-08-25
topics:
  - guardrails-and-feedback-loops
---

# Summary: How to Stop AI from Ruining Your Codebase

The Modern Software Engineering channel interviews Ivett Ördög — an independent consultant, TDD practitioner, and early adopter of agentic AI — about **Habit Hooks**, her tool for giving coding agents better habits. The framing opens with Kent Beck's "I'm a good developer with great habits": habits are what you fall back on when concentrating on something else, and now the inner loop of detailed development is done by agents. The question becomes what habits we want the *agent* to have — to avoid the "slopocalypse" where a codebase collapses under accumulated technical debt.

The trigger was repetition: Ivett kept saying the same thing to her agent — "this code doesn't look nice, please refactor it." Poorly designed code costs more tokens, is slower for the agent to work with, and makes coding errors more likely.

Habit Hooks runs the linter with JSON output and renders a prompt from it — but instead of passing the bare metric ("function too long, max 12 lines"), it explains what the smell *means* (doing too many things, not cohesive) and gives examples of how to fix it. The point is to shift the agent's attention from "number of lines" to "what problem am I trying to fix."

The worked example shows why this matters. A bare linter signal makes the agent split a function just to meet the line limit — even naming the second half "2" — so the code gets worse *and* the linter no longer detects it. With guidance, the agent decomposes the function into sensible one-line pieces. Ivett frames Habit Hooks as Birgitta Böckeler's "sensor" placed right next to the "guide": the sensor result (what to focus on) plus the guide (what to do with it).

An independent evaluation by Liina Suoniemi (Finland) tested 18 Python functions with two code smells (over-complexity, exception swallowing) across three prompts and two models. A bare "improve this code" prompt fixed the smell ~30% of the time; a bare linter metric ("fix it") did better for Haiku but worse for Sonnet — Suoniemi's hypothesis is that the stronger model is better at *gaming* the metric; a prompt including refactoring guidance fixed the smell **over 80%** of the time. The conclusion: adding refactoring guidance to linter output is worth doing.
