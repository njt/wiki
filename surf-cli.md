# surf-cli

A CLI that lets AI agents control your browser through Unix sockets instead of MCP. No API keys, no relay processes, no subscriptions -- load the Chrome extension, install the native host, and any agent that can run shell commands has browser automation. This is what Nat uses for browser automation in his own workflows.

---

## Key Quotes

> "Most tools require complex setup, tie you to specific AI providers, or break on real-world pages."

> "Browser automation for AI agents is harder than it looks."

## Key Themes

#tool #browser-automation #agent-tooling #cli

The fundamental insight is architectural: by using Unix sockets and pure CLI commands, surf-cli decouples browser automation from any specific AI provider. Every agent harness -- Claude Code, GPT, Gemini, custom -- gets the same interface. This is the opposite of the MCP approach, where each tool speaks a protocol that the harness must understand.

The battle-testing matters. They specifically tested against "agent-hostile pages like Discord settings" and built graceful CDP fallbacks. Most browser automation tools work great on toy examples and shatter on production sites.

Token efficiency is a quiet win: auto-resizing screenshots to 1200px and auto-capturing after actions eliminates round trips that eat both time and tokens. Small details that compound.

## Critical Analysis

The Unix socket architecture is elegant but platform-constraining -- this is a Mac/Linux tool. The element reference system (e1, e2, e3) that survives DOM changes is genuinely clever; most automation breaks when the page re-renders.

What's missing is a story for headless/remote scenarios. Nat uses it locally, but for server-side agents (like the Hermes containers), you'd still need something like a headless Chrome CDP setup. The extension-based approach trades remote flexibility for zero-config local experience.

The network capture feature (automatic HTTP request logging with filtering) is underrated -- it turns the browser into a debuggable API client, which is exactly what agents need when they're interacting with web apps.

See also [[claude-replay]] for another tool in the agent observability space, and [[Feedback Loop is All You Need]] for why deterministic tooling beats prompting.

---
*Sources: [[raw/surf-cli]]*
*Last updated: 2026-05-14*
