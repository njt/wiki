---
url: https://awaitinginput.substack.com/p/poor-mans-loop-engineering
title: Poor Man's Loop Engineering
author: Unknown
date_fetched: 2026-08-21
date_published: undated
---

# Poor Man's Loop Engineering

by an unnamed author on the Awaiting Input Substack.

The thesis: a working agent loop — an agent that can implement a scoped task end-to-end without regressions — does *not* require elaborate infrastructure. The "poor man's version" needs exactly two things: (1) the agent must be able to test its own work, and (2) someone other than itself must review that work.

## What is Loop Engineering?

The term was coined by Addy Osmani (Google) in his tech blog: "Loop engineering is replacing yourself as the person who prompts the agent. You design the system that does it instead." The author notes this isn't new — devs have built loops for their agents since late 2023. The goal isn't "press go and come back tomorrow" but automating some of the iteration cycles developers normally perform themselves.

## Ingredient 1 — Test, Test, Test

The first step is the most important: give the agent a way to know it's wrong. Not writing unit tests (agents already produce those in bulk) but a *safe* way to run integration/E2E tests — solid Make commands to spin up the Docker stack, a UI sandbox to click around and capture screenshots. The author's example Makefile ends in a single `verify: test check e2e` target — one command for the agent to run everything. This is what gets closer to the "sweet benchmarks" frontier labs gloat about: *here is the sandbox, here are the tools, go do the thing.*

## Ingredient 2 — Get Some More Eyes On This

Adversarial review: set up a Claude hook so that when a task is complete, two *fresh, isolated* context sessions review the branch/worktree changes against the goal set for the session. Reviewers get a clean view of what should have been done vs. what was actually done, and check: do tests pass? Are there gaps in the original vision? Scaling or security issues? They return PASS or FAIL with specific findings, and the loop repeats until clean. Two agents are used — ideally with cross-model validation — "because agents aren't deterministic, and we want to cover as many possible paths, thought processes, and high-level failure points as possible." The method was popularized by the controversial "Rewriting Bun in Rust" post.

## Definition of Done

The loop, compressed: run the project's tests and fix failures → verify end-to-end → ask two fresh isolated agents to review against the original task → fix valid findings → test and review again → only mark complete when the loop is clean.

## The Warning

Osmani himself warns against running too far away: "stepping inside the loop is a critical part of the process." The engineer must still own the system — "and be warned, the agents lack taste." A closing reader comment captures the frame: the goal isn't to engineer ourselves out of the loop, just out of the most tedious parts of it.
