# How AI Coding Agents Actually Use Your Technology

Waldek Mastykarz traces the exact path from "developer types a prompt" to "agent generates code" — seven steps through the AX (Agent Experience) stack where your SDK, CLI, or MCP server either creates lift or silently degrades into drag. Most failures are invisible: your tool wasn't dropped, it was just never selected, or it was called but returned the wrong kind of content, and you'll never know unless you instrument each step.

> "You have no idea what's actually happening between 'developer types a prompt' and 'agent generates code with your technology.' Is the agent reading your docs? Is it calling your MCP server? Is it ignoring both and guessing from memory?"

The core insight: **agents don't use your technology the way humans do**. They read everything at once. They make tool selection decisions via semantic matching, not keyword search. They take your error messages literally because they have no intuition. And context window pressure means your extension description is competing for space before the model even sees it — if it exceeds the harness's length limit, it gets dropped entirely, no matter how relevant.

> "If you return 3,000 tokens of documentation when 200 would do, you just pushed other relevant context out of the window. That's drag."

> "Your error messages aren't just for human developers anymore. They're for agents, and agents have no intuition to fall back on. They take your error message literally."

> "Most AX failures are invisible. They happen upstream, silently, and you never see them."

## Key Themes

#AX #agent-experience #tool-discovery #information-cascade #MCP #context-window

## Critical Analysis

This is the best practical guide I've read for anyone shipping tools that agents consume. Mastykarz is doing something rare: explaining the *mechanics* of agent-tool interaction rather than the metaphysics. The seven-step cascade is a diagnostic framework, not just a taxonomy — each step has a different fix, and you can't fix what you can't locate.

The "subrouting failure" example in Step 4 is devastating and specific: two MCP servers, one returns data directly while the other returns a routing menu, and the model sees the first response and stops — never calling the subtool that held the actual answer. This is the kind of integration bug that passes all unit tests and only shows up in production with real model behavior.

The LSP/Problems panel observation in Step 6 is underrated. If your VS Code extension contributes diagnostics, you're influencing agent output *during* generation, not just after. That's a surface most tool vendors haven't thought about.

My one critique: the article assumes a single-harness view (Copilot/VS Code) and doesn't address how wildly different harnesses (Claude Code, Cursor, Codex, Aider) handle context assembly. The principles generalize, but the specifics are harness-dependent in ways that matter — what Claude Code puts in context vs. what Copilot does are different enough that a tool optimized for one might fail on another.

## Related

- [[10 Principles for Agent-Native CLIs]] — Chow's framework for designing CLIs agents actually use well
- [[Components of a Coding Agent]] — The harness matters more than the model; six-component taxonomy
- [[Building Agents for Production Systems with MCP]] — Anthropic's guide to MCP as the standard integration layer
- [[Agent-Native Architectures (Every)]] — Five design principles for agent-first systems
- [[Guardrails and Feedback Loops]] — Linters beat prompts; deterministic enforcement over instructions
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management is the real engineering challenge
- [[Printing Press]] — Agent-native CLI generation from API specs
- [[Control Plane MCP Server]] — Real-world example: 80+ tools with virtual resources as safety curation

---
*Source: [Microsoft for Developers](https://developer.microsoft.com/blog/how-ai-coding-agents-actually-use-your-technology), Waldek Mastykarz, 2026-05-27. Part 2 of the AX stack series.*
