# lat.md

A knowledge graph for your codebase, written in markdown. Wiki-style linking between markdown files and source code annotations (`// @lat:` / `# @lat:` comments) create a bidirectional map of architectural concepts and business logic. `lat check` enforces referential consistency so docs can't drift from code.

---

## Key Quotes

> "AGENTS.md doesn't scale. A single flat file can describe a small project, but as a codebase grows, maintaining one monolithic document becomes impractical."

lat.md is the answer to the question nobody was asking loudly enough: what happens when your agent instructions file gets too big? The answer isn't a bigger file — it's a graph of interconnected files that agents can navigate with semantic search.

> "When you review a diff, start with the semantic changes in `lat.md/` to understand *what* changed and *why*. Reviewing code becomes the secondary task."

This is the workflow inversion that matters. Instead of reading through code diffs to reverse-engineer intent, you read the knowledge graph diff first. The code becomes the implementation detail you spot-check, not the primary artifact you review.

> "The context and reasoning behind your prompts is usually lost after a session ends. With lat, agents capture that knowledge into the graph as they work."

This is the [[Agent Memory and Context]] play: the knowledge graph becomes persistent context that survives sessions. It's [[napkin]] but structured and cross-linked.

## Key Themes

#tool #knowledge-graph #codebase-understanding #agent-tooling #spec-driven #test-enforcement

The test spec enforcement pattern is underrated: sections can be marked with `require-code-mention: true`, and `lat check` flags any spec without a `@lat:` backlink in test code. This is the same principle as [[Feedback Loop is All You Need]] applied to test coverage — make missing coverage mechanically detectable rather than a thing you hope someone notices in review.

The CLI commands reveal the mental model: `lat expand` takes a prompt with `[[refs]]` and expands them for agents. This means prompts can reference the knowledge graph directly. You don't say "fix the OAuth flow," you say "fix [[auth#OAuth Flow]]" and lat expands that into the full context the agent needs. This is [[Specifications as the Product]] in microcosm — the reference *is* the understanding.

## Critical Analysis

The bet here is that agents will be disciplined enough to maintain the knowledge graph. The `lat check` validation gives you enforcement, but enforcement only catches drift — it doesn't create the initial documentation. The bootstrapping problem is real: who writes the first `lat.md/` files? If humans do it, it's a chore. If agents do it, you're trusting agents to understand your codebase well enough to document it correctly, which is the very problem lat.md claims to solve.

The comparison with [[graphify]] is instructive. graphify auto-generates a knowledge graph from code analysis (zero maintenance, lower accuracy). lat.md requires human or agent effort (maintenance cost, higher accuracy). The README hints at a hybrid: "run CI tasks that improve the knowledge graph in the background." The endgame for both tools converges — auto-generated graphs with human-verified annotations.

Multi-language support (Rust, Go, C, TypeScript), agent integration (Codex, Cursor, Pi, OpenCode), and the MCP server for editor integration show awareness that this needs to work everywhere. The configuration pattern (env var → file → helper command → config file) is sensible for both local dev and CI.

The semantic search dependency on OpenAI/Vercel AI Gateway is a lock-in vector worth noting. If you're already using those providers it's fine, but if you're running local models via [[maclocal-api]] or [[Self-Hosted LLMs]], semantic search is gated behind a cloud API key.

See also [[Specifications as the Product]], [[Feedback Loop is All You Need]], [[napkin]], [[graphify]], [[docmason]], [[sem]], and [[Agent Memory and Context]].

---
*Source: [lat.md](https://www.lat.md/) | [GitHub](https://github.com/1st1/lat.md)*
*Fetched: 2026-05-14*
