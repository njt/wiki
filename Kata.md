# Kata

Wes McKinney's local-first issue tracker built for AI-assisted software work. An agent-friendly CLI (JSON output, stable commands, predictable failure modes) paired with a human-facing TUI (browse, triage, edit, supervise). Both talk to the same local daemon and SQLite database.

---

## Key Themes

#task-tracking #agentic-coding #devtools

The architectural bet: the issue ledger should be a local service adjacent to workspaces, not a database owned by each repository. A repo using kata gets only a small `.kata.toml` binding file; canonical state lives in `KATA_HOME` behind a daemon API. This keeps task state out of code history while giving agents a structured coordination layer.

Design priorities:
- **Agent ergonomics** -- stable commands, JSON-first workflows, explicit workspace binding, search-before-create, idempotency keys, predictable exit codes
- **Human oversight** -- TUI for browsing and supervising agent activity without reading raw JSON
- **Auditability** -- append-only comments, event history, actor attribution, explicit destructive operations

Compared to Beads (Dolt-powered distributed graph tracker), kata is deliberately smaller: one daemon, one local store, one HTTP API, one TUI, and a narrow issue model. Where Beads offers distributed database semantics and federation, kata offers simplicity and teachability.

This connects to the broader task-tracking-for-agents category in [[Agentic Coding]]. [[When the Target Keeps Moving]] provides the framework for what to track (discovery vs. delivery); kata provides the mechanism. And [[Radical Accountability]] is McKinney's broader thesis -- the same person building kata is arguing that AI removes the excuse for mediocre software.

## Critical Analysis

Kata makes smart trade-offs. Keeping state out of the git repo avoids polluting code history. Using a local daemon instead of a hosted service keeps the tool fast and dependency-free. The narrow issue model (no project management, no workflow engine) is a feature -- it's easy to teach to agents.

The risk: "local-first" means no built-in collaboration. A shared server mode is planned but not yet built. For solo agent-assisted work, kata is ideal. For teams, the lack of multi-user coordination may push people to Beads or traditional issue trackers. The question is whether kata will ship the shared mode before the window closes.

---
*Sources: [[raw/kata]]*
*Last updated: 2026-05-14*