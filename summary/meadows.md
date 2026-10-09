---
url: https://github.com/lorezzed/meadows
title: "Meadows — a text language for stock-and-flow diagrams"
author: lorezzed
date_fetched: 2026-10-10
date_published: 2026
topics:
  - developer-tools
  - software-engineering-craft
---

Meadows is a tiny text language for **stock-and-flow diagrams** — the system-dynamics notation from Donella Meadows' *Thinking in Systems* — with an interactive playground that renders the model as a live force-directed diagram and simulates its behavior over time. You write one-line chains like `| =>inflow [water in tub: 50] =>outflow: 5 |`; the compiler turns them into graph JSON (stocks as boxes, flows as faucets on pipes, clouds for the outside-the-model boundary, thin arcs for information links), and the frontend lays it out with d3 forces and plots each stock's trajectory.

The repo is deliberately two halves meeting at one JSON seam: a **PureScript compiler** (`src/`: Lexer → Parser → Evaluator, ~900 lines) that turns source text into `{nodes, links}` — with positioned lex/parse errors and semantic "model errors" (formula cycles, reading a faucet outside a time shift) — and a **TypeScript + d3 frontend** (`ui/`, ~6,900 lines) that renders the graph and simulates it. The frontend imports the *compiled* backend from `output/`, so editing the compiler requires `spago build` before a browser refresh means anything. The simulator (`ui/simulate.ts`, dependency-free) integrates forward Euler at DT=0.05, rations stock outflows so levels never go negative, and implements goal-seeking and reinforcing faucet semantics inferred from the *shape* of drawn information arrows — a bare faucet reading a discrepancy becomes an exponential approach to a goal.

The language's core trick is **identity by name**: the first mention of a name creates its node, every later mention anywhere refers to the same node, so a web is built by cross-referencing across lines. Formulas (`name: (expr)`) automatically draw the information arrows they imply, deduplicated against hand-drawn ones; `x(t - T)` time shifts are ring buffers that also break dependency cycles, which is what makes the dealership/hog-cycle oscillation models work. Built-in examples reproduce the book's figures 1–18 and the dealership (31/33). Toolchain (purs, spago, node 22, esbuild) is pinned by a Nix flake; tests include golden JSON, headless simulator/layout tests that run the *same* code the browser runs, and a CLI (`make run-with "a->b"`) that prints graph JSON.
