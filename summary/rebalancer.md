---
url: https://github.com/facebook/rebalancer
title: "Rebalancer"
author: "Meta Platforms (Neeraj Kumar, Pol Mauri Ruiz, Vijay Menon, Igor Kabiljo, et al.)"
date_fetched: 2026-10-10
date_published: 2026-10-06
topics:
  - developer-tools
  - distributed-systems
---

Rebalancer is Meta's open-source (Apache 2.0) **assignment solver library**: a C++ core, single-process and multi-threaded, that takes any "assign objects to containers" problem — shards to hosts, tasks to machines, ML jobs to clusters — subject to constraints (capacity, colocation, disaster-recovery spread) and goals (balance, minimize movement), and computes a good assignment. It handles ~1M objects and containers.

The API is deliberately declarative: the user names the objects and containers, attaches numeric dimensions to each, and adds *specs* — reusable, parameterized constraint/goal templates like `CapacitySpec` or balance — rather than writing solver code. The same problem definition can then be handed to two very different engines: a **local search** solver (dozens of move types — single moves, swaps, chain moves, group moves — applied greedily or randomly until no improvement, scaling to huge problems with no optimality guarantee) or an **optimal MIP solver** backed by HiGHS, Gurobi, or FICO Xpress (provable optimality, poor scaling). Choice of algorithm is independent of problem definition.

The codebase (~240K lines under `algopt/`) is layered: `entities/` (Universe, Objects, Containers, Scopes, dimensions), `materializer/` (~60 SpecBuilders that lower declarative specs into an expression tree), `solver/expressions/` (the expression evaluator the local search differentiates over), `solver/moves/` (the move-type library), and `algopt/lp/` (a generic MIP abstraction over three backend solvers). Bindings ship via nanobind (PyPI) plus Thrift-based C++/Python interfaces; a bundled **Rebalancer Explorer** (C++ Thrift service → JSON proxy → Next.js UI) lets you inspect runs. Production pedigree: dozens of Meta resource-allocation systems, documented in an OSDI 2024 paper. Notably, the repo also ships `.llms/skills/` — Claude Code skills for debugging solver runs with `eval_move` and `query_mip` CLIs.
