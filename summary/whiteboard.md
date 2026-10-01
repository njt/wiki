---
url: https://github.com/devdotfast/whiteboard
title: "Whiteboard — an open-source canvas for thoughtful software design"
author: devdotfast (dev.fast)
date_fetched: 2026-10-02
date_published: 2026
topics:
  - ai-code-review
  - developer-tools
---

Whiteboard is an open-source (MIT) desktop app where humans and coding agents architect software together on a shared canvas. It plugs into Claude Code, Codex, Cursor and others and gives each agent an SDK/MCP server to "draw" on an in-app canvas: sequence diagrams, ER diagrams, flow diagrams, call-stack diffs, and quotes from the agent's own trace.

Three headline features: (1) **diagrams that lead to code** — clicking any visualization or trace quote jumps to the underlying source, with VS Code keybindings and LSP support from the vendored Code-OSS fork; (2) a **semantic, AST-aware diff viewer written in Rust** (`diffr`, distributed as a WASM-plugin-extensible binary) that summarizes large added functions as pseudocode and collapses tests/docs by default; (3) a **decision log** — agents query and link their own traces on the board so you can see which requirements were set, how they were implemented, and what the agent decided autonomously.

Architecturally it is a pnpm monorepo: `packages/review` (~84k lines of TypeScript) is the CLI + runtime + authoring engine, `packages/trace-core`/`trace-protocol` capture and store agent session transcripts per harness, `packages/review-protocol` defines the Zod-validated block/document schema, and `apps/review-desktop` vendors Code-OSS wholesale (patchless fork, "45% of the codebase is Copilot") because agents struggle with patches and the team only needs the review/editor shell. The agent-facing surface is an MCP stdio server plus per-harness plugins (`.claude-plugin`, `.codex-plugin`, `.cursor-plugin`, a pi skill) and served instructions (`session_get_instructions({topic: "authoring" | "scratchpad" | "trace-archaeology"})`), so prompting guidance ships with the product rather than living in a README. A headless CLI can author reviews from CI, and a git-hook + commit-trailer system (`Agent-Session: <id>`) ties commits back to the agent sessions that produced them — the basis of the "trace archaeology" workflow for asking why code exists.

Positioning: understanding is the bottleneck, not generation — the README cites Geoffrey Litt, Maggie Appleton and Karpathy's "junior engineer savants". It runs against local checkouts, is self-hostable, and a hosted team product is planned.
