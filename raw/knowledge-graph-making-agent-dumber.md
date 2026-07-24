---
url: https://praveenvijayan.substack.com/p/your-knowledge-graph-is-making-your
title: Your Knowledge Graph Is Making Your Agent Dumber
author: Praveen Vijayan
date_fetched: 2026-07-25
date_published: 2026-07-24
site: Substack (praveenvijayan.substack.com)
---

# Your Knowledge Graph Is Making Your Agent Dumber

Praveen Vijayan evaluates Graphify, a tool that converts a code folder into a knowledge graph for AI agent use. He tested it on a live TypeScript monorepo (Bun/Turbo, 605 files, ~445k words, 4 apps + 4 packages) with three real questions, comparing it against ripgrep + manual file reads.

## What the graph contained

The built graph had 1,842 nodes and 3,722 links across 103 communities. Node types: 1,520 code nodes (AST-extracted), 164 concept nodes (LLM-extracted), 110 document nodes, 48 rationale nodes. ~76% of all edges came from `contains`, `imports`, and `imports_from` — all AST-derived. Semantically interesting relations: 86 edges total, "under 3% of the graph."

## Where Graphify Won

**Hyperedges over plan directories** — The tool clustered ~110 markdown plan files into named workstreams (e.g., "Notifications subsystem," "Onboarding Persistence Bug Chain"). The author calls this "the standout feature," noting it "correctly ties a plan that is still open to the four completed plans it depends on."

**Community labels as onboarding** — 103 communities with human-readable labels that cross directory boundaries.

**Import cycle detection** — Returned "None detected" cleanly; "genuinely useful."

**Low query-time cost** — ~0.2 seconds wall time per query with no API charges.

**Commit tracking** — `built_at_commit` recorded, making staleness detectable.

## Where It Failed

**Query returns nodes, not answers** — A question about OAuth provider registration returned 170 nodes. "Roughly 8 of 170 nodes are on-topic." Top results were barrel files and unrelated modules. By contrast, `rg` returned 35 lines in 0.01 seconds with the architecture legible in the output.

**Hub poisoning in barrel-export monorepos** — The top three nodes by degree (176, 170, 160) were barrel files. BFS depth-2 from `createApp()` reaches 275 nodes (15% of the graph); from `notifications.ts` it reaches 465 nodes (25%). The author labels this "a graph-topology problem."

**Stale graphs report dead symbols as EXTRACTED** — A function renamed 40 seconds before the next commit still appeared with highest-confidence label. "Wrong data wearing a confidence badge is more dangerous than no data."

**Degree centrality mislabels utility functions** — `cn()` (a four-line Tailwind helper) ranked as a "core abstraction" due to high fan-in. Six of the top ten most-connected nodes were leaf utilities.

**Community cohesion was extremely weak** — All scores ranged 0.03–0.08, meaning "the partition is barely better than arbitrary." Yet LLM-generated labels sounded authoritative. "The label quality is decoupled from the cluster quality."

**Inverted token economics for agents** — Graphify returned ~2000 tokens with ~5% on-target; `rg` returned ~300 tokens at ~100% on-target. The tool "reconstructs, imperfectly, what good repos already write down" — the repo already had hand-written `AGENTS.md` files.

**Build cost** — 1.19M input tokens across two builds. Real expense on first build, though caching helped second run by 6×.

## Structural Prediction Rule

From §2.2: If a repo has barrel files, a shared types package, a DI container, a God app-factory, or a widely-imported `utils.ts`, import-graph traversal will produce this failure pattern. Genuinely modular codebases with narrow interfaces would fare far better.

## Recommended Workflow

- Build once, read `GRAPH_REPORT.md` end-to-end as an onboarding artifact
- Use queries only to find seed entry points, then switch to `rg` + reading
- Never use `explain` on code symbols unless drift is zero
- Scope builds to prose directories rather than code directories
- Add a staleness guard checking `built_at_commit` vs. `HEAD`

## Verdict

**Strong fit:** Repos with large prose corpora, undocumented legacy codebases, one-time onboarding, genuinely modular codebases, mixed-media corpora.

**Poor fit:** Barrel-export TypeScript monorepos, repos with strong docs culture, fast-moving codebases without automated update hooks, precise symbol lookups (use `rg`/LSP instead).

Closing thesis: "Graphify is an excellent document knowledge graph and a mediocre code knowledge graph."

## Method Notes

All measurements from one private TypeScript monorepo over one session. Degree distributions and staleness blast radius computed from `graph.json` directly. `graphify update` and `watch` commands not exercised. Topology argument in §2.2 should generalize; precision numbers should not be assumed to transfer.
