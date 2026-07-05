# weft

Cloudflare-hosted Trello/kanban for your agents. A personal task board where AI agents autonomously work on assigned tasks -- reading emails, drafting responses, updating spreadsheets, creating PRs. All state-mutating actions require human approval. Self-hosted on Cloudflare Workers with Durable Objects for persistent state and real-time WebSocket updates.

---

## Key Quotes

> "Task management, but AI agents do your tasks. Self-host on Cloudflare."

## Key Themes

#task-management #kanban #cloudflare #agents #approval-workflow

The approval-based safety model is the key design decision: agents can execute read-only operations freely, but anything that mutates state (sending emails, creating PRs, modifying documents) requires human sign-off. This is the right default for a general-purpose agent task system.

Built-in integrations cover the common agent tasks: Gmail, Google Docs, Google Sheets, GitHub, Cloudflare Sandbox for code execution, plus remote MCP servers for custom tools. Scheduled tasks (cron expressions) enable recurring agent work.

The Cloudflare-native architecture is interesting: Workers for HTTP/auth/frontend, Durable Objects for state management, Workflows for durable agent loops with automatic checkpointing and retry. This means the infrastructure cost is near-zero at small scale (Cloudflare Workers pricing) and the deployment is a single `wrangler deploy`.

Part of the agent task-tracking ecosystem alongside [[ralph-ban]] (simpler, TUI-based), [[workgraph]] (more sophisticated, graph-based), and [[Managing Agents via Kanban Boards]] (Geoffrey Litt's approach). weft is the web-native option with the broadest integration surface.

## Critical Analysis

The Cloudflare dependency is both strength and weakness. Strength: the Durable Objects abstraction handles the hard parts of persistent, real-time state management. Weakness: you're locked into Cloudflare's platform. The approval workflow is well-designed for safety but could become a bottleneck -- if you have 10 agents generating 50 approval requests per hour, the human becomes the rate limiter. For personal use or small teams, this is a clean solution. For larger operations, you'd need [[speedrift-ecosystem]]'s bounded autonomy instead.

---
*Sources: [[summary/weft]]*
*Last updated: 2026-05-14*
