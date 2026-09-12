# Claude Sidecar

A parallel AI window that shares your Claude Code session context with other models (Gemini, GPT, DeepSeek, Qwen, Grok, etc.), lets you interact simultaneously, then folds structured results back into your main session. Built by John Renaldi as an Electron harness on top of OpenCode. Installed via npm, configured with a graphical setup wizard, and integrated into Claude Code via MCP and a Skill file.

---

## Key Quotes

> "Your main Claude session keeps working while you interact with the sidecar. It's not sequential, it's simultaneous."

This is the core insight. Sidecar isn't a model-switcher — it's a parallel track. The sidecar operates *alongside*, not after. You don't stop Claude to ask Gemini something; both run concurrently.

> "We don't reinvent the wheel. OpenCode handles the hard parts of LLM interaction so Sidecar can focus on the parallel-window workflow."

Renaldi is refreshingly honest about what Sidecar is: a harness, not a new engine. OpenCode provides the conversation runtime, tool execution, and agent system; Sidecar adds context sharing from Claude Code, the Electron shell, fold/summary workflow, and session persistence. This is the right call — the value is in the integration pattern, not in building yet another LLM runtime.

> "Sidecar reads your Claude Code session from ~/.claude/projects/[project]/[session].jsonl and passes it to the sidecar model automatically."

The context-sharing mechanism is file-based and practical. It reads the JSONL session transcript directly from disk. No API integration, no middleware, no cloud dependency. Sidecar lives entirely client-side, on your machine. This is desktop-tool thinking applied to AI workflows.

> "Click FOLD (or press Cmd+Shift+F) to generate a structured summary. The summary flows back into your Claude Code context for Claude to act on."

The Fold is the critical UX innovation. It's not just copy-paste — it's a structured handoff with sections for findings, attempted approaches, recommendations, code changes, and open questions. The output is designed to be useful to an agent, not just readable by a human.

## Key Themes

#tool for the parallel-window pattern in AI-assisted development. #concept context-sharing as a file-based primitive rather than an API integration. #pattern fold mechanism: structured agent-to-agent handoff that returns cleanly to main context. #person John Renaldi, developer of Sidecar.

## Critical Analysis

Sidecar solves a real problem that anyone who's used Claude Code heavily will recognize: you want a second opinion but don't want to lose context or interrupt flow. The "just open another window" approach — copy-paste your prompt into ChatGPT or Gemini — is friction-heavy, strips conversation history, and produces results you have to manually re-integrate. Sidecar automates all three steps.

The architecture is simple in the best way. Read JSONL from disk, pass to OpenCode, render in Electron. No server, no cloud dependency, no API middleware. This is a desktop tool for desktop developers, and that constraint is a feature: nothing leaves your machine except the API calls to models you've configured.

The MCP integration is the sleeper feature. Sidecar tools (`sidecar_start`, `sidecar_status`, `sidecar_read`) appear natively in Claude Desktop and Cowork, so sidecars can be launched programmatically by other agents. Combined with session persistence (`sidecar list`, `sidecar resume`, `sidecar continue`), this creates a workspace where sidecar sessions become persistent, queryable resources rather than ephemeral windows. You can launch a sidecar, let it run, come back to its results hours later.

The three agent modes (Chat/Plan/Build) map cleanly to escalating trust levels: human-in-the-loop, read-only review, full autonomy. This is a mature design decision — one mode doesn't fit all use cases, and the constraints are enforced by the harness, not by prompting.

Headless mode (`--no-ui --agent Build`) is where things get interesting. This creates autonomous background workers that return structured `[SIDECAR_FOLD]` summaries. Combined with MCP, you could have Claude Code programmatically spawning sidecars to parallelize work. This is effectively multi-agent orchestration via a sidecar pattern — lighter weight than full orchestration frameworks, but potentially sufficient for many workflows.

Weak spots: 13 stars and 10 open issues suggest very early adoption. The "experimental" Claude Code web/Desktop support introduces version risk if Anthropic changes session formats or MCP APIs. The dependency on OpenCode means Sidecar inherits OpenCode's limitations and release cadence. But the blast radius of failure is small — it reads files from disk and renders a web UI. If it breaks, you lose a convenience, not your work.

The biggest open question: does the Fold format actually serve as effective agent-to-agent communication, or does it lose too much nuance? Structured summaries are lossy by design. The 80/20 is probably strong — most sidecar tasks are bounded investigations — but for complex reasoning chains, you might still want to read the full conversation.

## Cross-References

- [[On a Year of Multi-Model Development]] — Hoffman's field report on multi-model workflows, which Sidecar mechanizes
- [[Fresh Eyes]] — The same principle (different model, different blind spots), automated into a tool
- [[Components of a Coding Agent]] — Sidecar as a case study in harness-over-model design
- [[Inside the AI Workflows of Every's Six Engineers]] — Multi-model, planning-first workflows converging on patterns Sidecar enables
- [[Designing Agentic Loops]] — Simon Willison on the meta-skill of choosing tools and guardrails; Sidecar is one answer
- [[Agent Orchestration]] — Headless sidecars as lightweight multi-agent coordination
- [[Building 200+ Integrations with OpenCode]] — What OpenCode provides that Sidecar builds on
- [[Control Plane MCP Server]] — MCP as the integration layer Sidecar plugs into
- [[Agent Memory and Context]] — Context sharing is the engineering problem Sidecar addresses
- [[Coding Agents and Complexity Budgets]] — Sidecar as a context-preservation strategy: deep-dive without pollution
- [[Addy Osmani's Workflow]] — Work in focused chunks; Sidecar keeps tangents from fragmenting the main thread
- [[Claude Code Cheat Sheet]] — The Claude Code surface Sidecar extends
- [[pi-gui]] — a native desktop shell for the Pi coding agent; the single-agent GUI answer to the same "agents need more than a terminal" problem Sidecar solves with a parallel-window harness

---
*Sources: [[summary/claude-sidecar]]*
*Last updated: 2026-05-15*
