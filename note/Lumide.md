# Lumide

A cross-platform code IDE built in Flutter and Dart that makes two bets: that native desktop performance (200ms cold start, 80MB idle, ~40MB download) is a moat against Electron-based editors, and that agentic AI belongs in a first-class sidebar, not a chat panel bolted onto a pre-AI architecture.

---

## What It Is

Lumide is a from-scratch IDE targeting developers who want an editor that feels like a desktop tool rather than a web page in a Chromium wrapper. Built in Flutter/Dart — the same stack that powers hot-reload mobile development — it promises sub-second cold starts and sub-100MB idle memory, positioning itself against VS Code's multi-second startup and multi-hundred-MB footprint.

The headline feature is the "agentic sidebar": AI assistance integrated as a persistent workspace panel rather than a pop-up chat window. The pitch is "agent-native development" — the IDE was built after LLMs existed, so AI integration is architecture, not afterthought.

Available on macOS, Windows, and Linux. The macOS download clocks in at ~40MB.

## Key Quotes

> "200ms cold start / 80MB idle"

This is the entire value proposition in two numbers. Every Electron user knows the pain this is selling relief from. Whether these numbers hold up with plugins, LSPs, and real workloads is the open question.

> "Crafted with Pure Flutter & Dart for people who want an IDE to feel like a desktop tool: fast startup, low memory use, and no Electron layer."

The "no Electron layer" is doing a lot of work here. It's a tribal signifier as much as a technical claim — the same move Zed makes with Rust. The difference is that Flutter carries mobile baggage that Rust doesn't.

> "Native speed for agent-native development."

Concise positioning: performance and AI as co-equal pillars, not AI bolted onto a slow substrate.

## Key Themes

- #tool — A code IDE with agentic features as first-class architecture
- #concept — "Agent-native development": AI as a persistent sidebar companion, not a transient chat window
- #pattern — Native desktop performance as moat: Flutter/Dart over Electron, same thesis as Zed with Rust
- #concept — Cross-platform from day one (macOS, Windows, Linux) with a single codebase and small binary

## Critical Analysis

**The Flutter bet is bold and weird.** Flutter is a mobile-first framework. Its desktop story has improved dramatically but still lags behind its mobile support. Using it for an IDE — the most demanding desktop application category — is either visionary or hubristic. The upside is genuine: hot reload during IDE development, a single codebase across three platforms, and far smaller binaries than Electron. The downside is fighting uphill against every desktop-native expectation (text rendering, window management, accessibility, OS integration) that Flutter's mobile lineage wasn't designed for.

**Performance claims, not performance data.** "200ms cold start" and "80MB idle" are marketing numbers. What happens when you open a TypeScript project with an LSP running? What's the memory footprint with the agentic sidebar processing context? The landing page doesn't say. Every editor is fast with an empty file open.

**The agentic sidebar is a genuine UX hypothesis.** Chat panels are the default AI-in-editor pattern (Copilot Chat, Cursor's panel, Zed's assistant). A persistent sidebar that maintains context across your session is a different model — more like a pair programmer who stays seated beside you than a consultant you summon. Whether this actually changes the development experience or just moves the chat window 200 pixels left is the question.

**The extension ecosystem problem is existential.** VS Code won because of its extension marketplace, not because of its editor engine. A Flutter IDE needs a plugin architecture, an extension API, and a community willing to build for it. The landing page mentions none of this. Without extensions, it's a very fast editor for a handful of languages. With extensions, it's competing with a marketplace that has tens of thousands of existing extensions.

**"No Electron" is necessary but not sufficient.** Zed already proved you can build a screaming-fast native editor with AI integration and still struggle for market share. The problem isn't Electron vs. native — it's network effects, extension ecosystems, and team familiarity. Lumide needs a better answer to "why switch?" than "it's faster."

**The name and branding are clean.** Lumide — likely a portmanteau of "luminous" and "IDE" — avoids the try-hard naming that plagues developer tools. The landing page is sparse to a fault, but what's there is focused.

## Cross-References

- [[Zed]] — The other native-performance, non-Electron, AI-first editor. Rust instead of Flutter, same thesis, different tech stack. The comparison is unavoidable
- [[Agent Coding Workflow]] — The practitioner's daily loop that tools like Lumide are trying to reshape
- [[Intent Is the Interface]] — Reimagining the development surface; Lumide's agentic sidebar is a bet on a different interface paradigm
- [[Agent-Native Architectures (Every)]] — Design principles for agent-native tools; Lumide is attempting to be an agent-native IDE
- [[Components of a Coding Agent]] — The harness matters more than the model; Lumide is building harness, not model
- [[Loop Engineering]] — Addy Osmani on designing systems that prompt agents; an IDE with a persistent agentic sidebar is loop engineering made visible
- [[Hitomi (Data Viewer)]] — Another Flutter desktop app; evidence that Flutter can ship real desktop tools

---
*Sources: [[raw/lumide]]*
*Last updated: 2026-07-08*
