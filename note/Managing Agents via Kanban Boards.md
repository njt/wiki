# Managing Agents via Kanban Boards

Geoffrey Litt's practical guide to using a Notion kanban board as the coordination surface between humans and AI coding agents. The key innovation: a "blocked" checkbox with red conditional formatting that visually signals when an agent needs human input, plus a vibe-coded CLI tool (`notion-cc wait-for-comment`) that blocks execution until the human responds on the card.

---

## Key Quotes

> "Work on this task: <url>. Periodically update the current status property on the task so the user knows what you're up to. If you get blocked and need user input, set blocked: true and ask the user questions in the comments on the task. Once the user responds, set blocked: false and continue working."

> "I really enjoyed the pattern of 'LLMs writing code to be called by future LLM sessions'. Makes a ton of sense that repeated deterministic logic can be encoded for future use and run more efficiently."

> "To me that's the dream of malleable software: evolving our tools as we use them, making them fit us like a leather shoe, rather than bending to the constraints of the software."

## Key Themes

#task-management #kanban #coordination #cli #agents #malleable-software #mcp

The thread describes a two-level evolution. Level 1 is pure MCP: hook up the Notion MCP to Claude Code, write a prompt that tells the agent how to use the board (update status, set blocked when stuck, move to Done when finished). Level 2 comes when MCP hits its limits: vibe-code a TypeScript CLI (`notion-cc`) that wraps the Notion API for things that are more efficient as code than as LLM-driven MCP calls.

The `notion-cc wait-for-comment` command is the most interesting piece. It polls until a task has a new comment from the user, then returns that comment. This turns the kanban board into a programmable coordination primitive -- you can write shell scripts that dispatch work to agents via card creation and then block until the work is done. This is event-driven coordination without the complexity of message queues or webhooks.

The "LLMs writing code to be called by future LLM sessions" pattern is significant. Rather than having the LLM manually orchestrate API calls every time via MCP, the repeated deterministic logic gets encoded as a CLI tool that future sessions can call directly. This is the compound engineering idea from [[Compound Engineering]] in miniature.

The **malleable software** framing is the deeper point. Litt didn't design this workflow upfront -- he evolved it incrementally while working on his actual project. The "red card" idea came from frustration (flipping between terminal tabs) and took a minute or two to implement. The tools fit the user like a leather shoe because the user shaped them through use.

Part of the task-tracking ecosystem: [[ralph-ban]] (TUI kanban), [[weft]] (Cloudflare-hosted kanban), [[workgraph]] (graph-based coordination). Litt's contribution is the insight that the coordination surface should be *programmable*, not just visual.

## Critical Analysis

The kanban abstraction works well for the specific case Litt describes: a queue of tasks where an agent works independently and occasionally needs human input. The blocked/unblocked state machine is simple, visual, and sufficient.

Where it breaks down is complex dependencies, parallel execution, and conditional branching -- for those, [[workgraph]]'s graph-based approach is more appropriate. But for the common case of independent tasks, this is exactly right.

The most transferable idea is the two-level pattern: start with MCP for rapid iteration, then "graduate" repeated operations into CLI tools that are more efficient and deterministic. This matches the general principle that LLMs should be used for judgment and novelty, while deterministic logic should be codified.

The malleable software angle distinguishes this from other agent-coordination posts. Most people describe their workflow as a finished system; Litt describes how the system emerged from use. That's a more honest and more useful framing.

---
*Sources: [[summary/managing-agents-via-kanban-boards]]*
*Last updated: 2026-05-14*
