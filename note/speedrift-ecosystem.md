# speedrift-ecosystem

An autonomous dark-factory workflow: an operations control plane that supervises agent-driven software work across multiple repositories. Each repo keeps its own [[workgraph]] as the source of truth. Speedrift watches those repo graphs, detects stalls and drift, plans bounded corrective actions, dispatches agents when policy allows, and writes every decision back to tasks, logs, and ledgers.

---

## Key Quotes

> "Speedrift can restart safe services, emit corrective tasks, and run deterministic handlers under policy. It cannot rewrite unrelated local work without a trace, bypass verification gates, or perform destructive git history operations."

## Key Themes

#orchestration #dark-factory #multi-agent #drift-detection #automation

The architecture is a two-layer design that cleanly separates execution (who runs tasks) from judgment (what to do). Local repositories maintain authority over their task graphs; the central plane handles cross-repo visibility and planning. The "continuous cycle: observe -> prioritize -> plan -> execute -> record" is the OODA loop applied to software operations.

The module names reveal the philosophy: coredrift, specdrift, datadrift, archdrift, depsdrift, uxdrift, therapydrift, yagnidrift, redrift. Each one monitors a specific dimension of project health. The bounded autonomy is well-designed -- Speedrift can emit corrective tasks but cannot bypass verification gates or perform destructive operations.

Built on [[workgraph]] for the task graph spine, plus Agency (agent composition) and Freshell (browser terminal access). This is the most ambitious project in the "dark factory" category, making [[maestro]]'s role-based orchestration look tame by comparison.

## Critical Analysis

This is either the future of software development or an over-engineered solution to a problem that's better solved by smaller, simpler tools. The "therapydrift" and "yagnidrift" modules suggest a self-awareness about scope creep, which is either charming or worrying depending on your perspective. The key question: does a centralized control plane for agent work actually improve outcomes compared to simpler per-repo tooling like [[ralph-ban]] or [[weft]]? The answer probably depends on scale -- for 3 repos and 5 agents, this is overkill. For 50 repos and 200 agents, something like this becomes necessary. Watch this space.

---
*Sources: [[summary/speedrift-ecosystem]]*
*Last updated: 2026-05-14*
