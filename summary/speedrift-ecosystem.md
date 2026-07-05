---
title: "speedrift-ecosystem"
url: https://github.com/dbmcco/speedrift-ecosystem
date_fetched: 2026-05-14
section: "AI-enhanced Coding"
---

# Speedrift Ecosystem

Operations control plane that supervises agent-driven software work across multiple repositories. Observes distributed task graphs, detects stalls and drift, plans bounded corrective actions, and executes safe automation while maintaining full audit trails.

## Key Features
- Continuous observation of multi-repo task states through a central control plane
- Drift detection and automated corrective task emission
- Bounded autonomy execution (safe, traceable automation only)
- Live dashboard with status tracking and decision ledgers
- Integration with Claude/Codex agents for task execution

## Architecture
Two-layer design separates execution (Workgraph + Agency handles *who* runs tasks) from judgment (Planforge + Speedrift handles *what* to do). Local repositories maintain authority over their task graphs while the central plane coordinates cross-repo visibility and planning.

## Notable Quotes
"Speedrift can restart safe services, emit corrective tasks, and run deterministic handlers under policy. It cannot rewrite unrelated local work without a trace, bypass verification gates, or perform destructive git history operations."

Mental model shift: from "human polls many repos manually" to "daemonized control plane supervises all tracked repos" with "continuous cycle: observe -> prioritize -> plan -> execute -> record."

## Primary Modules
- Core lanes: coredrift, specdrift, datadrift, archdrift, depsdrift, uxdrift, therapydrift, yagnidrift, redrift
- Control modules: factorydrift, sessiondriver, secdrift, qadrift, plandrift, northstardrift

## Dependencies
Built on: Workgraph (task graph spine), Agency (agent composition), and Freshell (browser terminal access).
