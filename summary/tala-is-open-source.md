---
url: https://d2lang.com/blog/tala-is-open-source/
title: TALA is open-source
author: Terrastruct (d2lang)
date_fetched: 2026-09-08
topics:
  - developer-tools
---

# TALA is open-source

Terrastruct open-sources TALA (Terrastruct's AutoLayout Algorithm), the autolayout engine behind D2, under the MPL-2.0 license — the same license as D2 itself. Publication date is not stated in the post.

TALA is a novel autolayout algorithm built specifically for software architecture diagrams. It is primarily an **orthogonal layout engine** — producing the right-angle, whiteboard-style layouts engineers actually draw — rather than the DAG-based engines (Dagre, ELK) that grow in one direction. It blends techniques from several graph-drawing research papers (cited in source) with original work, and optimizes for multiple competing notions of "aesthetic": symmetry, median distance, flow, and clustering of like nodes.

Three properties distinguish it, each demonstrated with examples:

1. **Comparison quality.** Side-by-side renders of public GitHub D2 files show TALA vs. Dagre vs. ELK — not cherry-picked; some diagrams read better under the other engines.
2. **Custom positioning.** Node positions and sizes can be locked in. This targets agentic use: models are good at placing nodes in 2D space but struggle with edge routing, so TALA lets the model do the placement and handles the routing.
3. **Partial positioning.** A hybrid where *some* nodes specify coordinates and the rest are left to the engine — pin a cluster's shape, let TALA finish.

The author is candid about tradeoffs: TALA is randomized (3 seeds, best-scoring wins, so a one-node addition can reshuffle the whole diagram); it handles long flowing DAG graphs worse than Dagre/ELK; and it scales nonlinearly for larger diagrams (benchmarks at github.com/d2lang/d2-benchmarks). It ships in D2 v0.9.0 (`--layout=tala`) and runs client-side at play.d2lang.com.

Special thanks go to Gavin Nishizawa (broad contributions) and Júlio César Batista (hierarchy algorithms).
