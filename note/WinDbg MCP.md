# WinDbg MCP — Debugging with Natural Language, Grounded in Evidence

Microsoft ships an MCP server inside WinDbg so AI clients (VS Code, GitHub Copilot CLI) can root-cause crashes and kernel hangs in plain language, with a five-phase "Diagnostician" skill that treats model conclusions as hypotheses and Secure Mode plus prompt-injection classification as defaults.

---

## What it is

WinDbg MCP is an MCP server inside WinDbg (from version 1.2610.1001.0), connecting an AI client to a live debugger session through a local proxy. The debugger connection stays local; only the AI client talks to its configured model service. Supported clients today are VS Code and GitHub Copilot CLI; one client per session at a time.

The pitch is a workflow shift, not a command translator: instead of knowing which WinDbg commands to run and how to read their output, you ask *"Root cause this crash, show the supporting evidence, and state any unverified assumptions."*

## Diagnostician: root-cause as a procedure

The Diagnostician skill (part of the `windbg` plugin in the public win-dev-skills catalog) runs a five-phase investigation for apps, services, UMDF drivers, and kernel drivers. Its stated discipline:

> It treats pattern matches as hypotheses, tests alternatives, and delivers candidate root causes with targeted next steps and clear evidence gaps.

That's the right shape for agentic debugging — pattern recognition produces *hypotheses*, not verdicts, and the deliverable includes what's still unknown. It even writes a timestamped Markdown report to `.diagnoses/<short-id>/`, making the investigation an artifact you can diff and review.

## Security is the design center

Two safeguards ship on by default:

> **Secure Mode** restricts high-risk operations, including loading or executing untrusted code, launching processes, and performing unsafe file operations.

> **Cross-prompt injection protection** checks debugger command output for content that could mislead the AI into unsafe actions. For protected operations, WinDbg uses the AI client's configured model service to classify the output and blocks content identified as potentially unsafe.

The notable catch: Claude Code "may work if you disable cross-prompt injection protection, which we don't recommend" — because it doesn't support MCP sampling, the mechanism WinDbg uses to run the output classifier. In other words, an agent pointed at raw debugger output *is* a prompt-injection surface, and Microsoft is honest enough to name the trade-off rather than pretend all clients are equal.

## Take

This is one of the more mature MCP deployments anywhere, because the threat model is unusually sharp: debugger output is attacker-controlled text (crash dumps can embed adversary data), and the agent holds a loaded kernel-debugging session. Secure Mode + visible AI-initiated actions + an auditable log + classified output is exactly the layered posture this wiki keeps describing for dangerous tool surfaces. It also shows MCP sampling doing real security work — a protocol capability most tool discussions ignore.

The other quiet point: the Diagnostician skill is evidence-gap-driven analysis, the same posture as good human debugging. Agentic debugging done well doesn't remove the craft; it enforces it.

## Related pages

- [[Chrome DevTools MCP — Debug Your Browser Session]] — the same pattern one layer up: an MCP server exposing a live debuggable session (browser vs. process/kernel) to an AI client; WinDbg shows the security bar that session-debugging MCPs should meet.
- [[MCP Is Dead; Long Live MCP]] — WinDbg MCP is a counterexample to any "MCP is dying" take: a flagship Microsoft tool line adopting it as a first-class interface, with sampling doing real work.
- [[Win Dev Skills]] — Diagnostician ships in Microsoft's win-dev-skills plugin catalog; this source gives that catalog its first flagship workload.
- [[DDB — Source-Level Interactive Debugging for Distributed Applications]] — academic debugging harness vs. Microsoft's productized one; both move debugging from command recall to intent.

---
*Sources: [[raw/introducing-windbg-mcp-debug-with-natural-language-grounded-in-evidence]], [[summary/introducing-windbg-mcp-debug-with-natural-language-grounded-in-evidence]]*
*Last updated: 2026-10-08*
