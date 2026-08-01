---
url: https://github.com/spacedock-dev/subspace-beta
title: "Subspace Beta"
author: Spacedock
date_fetched: 2026-07-25
date_published: 2025
---

Subspace is a bridge between AI coding agents and human reviewers. An agent hands off a markdown file; Subspace opens it in a native terminal TUI, lets a human annotate it, and returns structured, validated JSON feedback the agent can consume immediately.

The architecture has three layers: an LLM-hosted skill file (SKILL.md, 131 lines) defining input contracts and permission rules; bash dispatch scripts (614 lines total) that handle six terminal hosts with a strict no-fallback detection priority (Zellij → tmux → Herdr → CMUX → Ghostty → Apple Terminal); and a proprietary closed-source binary, `subspace-tui`, that handles the TUI, briefing construction, and atomic result creation.

The design emphasizes neutral authority — the result schema has no "approved"/"rejected" field, and the skill forbids calling any outcome a decision. Review is input, not a gate. Other notable choices: single-file-only constraint (no directories, no multi-file), SHA-256 artifact pinning before review begins, canonical-path enforcement, and a capsule pattern that wraps async terminals in self-contained scripts with FIFO-based completion signaling.

Licensed Apache-2.0 for the open integration layer; the binary is proprietary. Distributed via spacedock-dev/marketplace for Claude Code and Codex.
