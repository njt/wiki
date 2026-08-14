# pi-gui

A native macOS/Linux desktop app that puts the open-source Pi coding agent inside a Codex-style GUI — workspaces, a live agent timeline, and persistent, resumable sessions — built on a `SessionDriver` interface that decouples the desktop shell from the agent runtime. Where Pi and its forks have been terminal-first, pi-gui makes the argument that the *surface* matters as much as the harness.

---

## Key Quotes

> "pi-gui is a Codex-style macOS and Linux desktop app for the pi coding agent. Manage workspaces, run sessions, and review agent work — all from a native interface."

The positioning is explicit: Codex has a proprietary desktop app; pi-gui wants the open-source Pi to get one too. "Codex-style" is doing real work here — it's borrowing a known-good UX metaphor rather than inventing one, the same way [[Codex-maxxing]] treats Codex's side panel as a work surface.

> "The desktop shell is separated from the agent runtime through a durable SessionDriver interface — making the frontend independent of backend changes and ready for future runtime swaps."

This is the most interesting sentence on the page. The shell is not Pi-specific — it's a runtime-agnostic viewer that happens to target Pi first. "Future runtime swaps" quietly signals the GUI could front Claude Code, Codex, or omp tomorrow.

> "Watch every tool execution, code change, and reasoning step in a scrollable timeline with full input and output detail."

The timeline is the review surface. Unlike a terminal TUI scrolling raw text, a timeline is structured — each event a typed object — which is what makes "full input and output detail" renderable rather than merely dumpable.

> "Sessions survive restarts. Resume any previous conversation, review transcripts, and continue exactly where you left off."

Persistence as a first-class feature, not a CLI flag. Terminal agents already write JSONL session files (Pi's do; [[Oh My Pi (omp)]] stores append-only JSONL with tree branching). pi-gui's claim is that the GUI makes that history *navigable*.

## Key Themes

**#tool** for the native-desktop surface of the Pi coding agent. **#concept** the SessionDriver decoupling — the UI as a runtime-agnostic viewer. **#pattern** the real-time agent timeline as a structured review surface versus the terminal's raw scrollback. **#concept** the Codex-style desktop as a converging UX: Codex has it natively, pi-gui backports it to open source.

## Critical Analysis

**The surface is the argument.** For all the energy spent on harness design — tools, edit formats, memory, orchestration — the Pi ecosystem has stayed terminal-first. pi-gui stakes out the opposite bet: a desktop app with workspaces, timelines, and resumable sessions is how an agent becomes a daily driver rather than a novelty. This is the "side panel as work surface" insight from [[Codex-maxxing]], backported to an open-source agent.

**SessionDriver is the real architecture, and it's undersold.** Separating the shell from the runtime makes pi-gui less "a GUI for Pi" than "a GUI that currently targets Pi." That's a meta-harness instinct in a single-agent wrapper — the same decoupling [[Traycer]] formalizes with its versioned RPC protocol and [[Introducing Omnigent]] wraps in a uniform API. The landing page gives it one sentence; it deserves the top billing the features get.

**The GUI/TUI split is the live experiment in the Pi family.** [[Pi Coding Agent]] is minimal-core-with-extensions, [[Oh My Pi (omp)]] went terminal-maximalist (~55K lines of in-process Rust), and now pi-gui goes native-desktop — three surfaces over the same agent runtime. Which one wins adoption will say more about developer taste than technical merit, and because all three are open over the same core, the ecosystem gets to run every bet at once.

**Early-stage reality.** Beta for macOS arm64 and Linux AppImage only — no Windows, no Intel macOS, no notarized distribution beyond Homebrew's cask machinery. The caveat that Homebrew upgrades may require re-confirming macOS permissions or Dock placement is a small but honest admission that a beta desktop app on macOS is still fighting the platform's trust model. The install matrix (DMG / cask / source via pnpm) is developer-shaped: this is a tool for people comfortable reading a README, not a consumer product.

**What the page doesn't say.** No diff view, approval gate, or sandboxing — the things [[Bram]] and [[Broomy]] make central. pi-gui is a *viewer and launcher*, not an enforcement layer. That's a defensible scope, but it leaves "review agent work" thinner than the marketing implies: seeing what the agent did is not the same as gatekeeping what it does.

## Cross-References

- [[Pi Coding Agent]] — the agent pi-gui wraps; the desktop surface extends the embedding bet Pi already makes with SDK/RPC mode
- [[Oh My Pi (omp)]] — the terminal-maximalist Pi fork; pi-gui is the GUI pole of the same spectrum
- [[Codex-maxxing]] — "side panel as work surface" is the Codex-native pattern pi-gui backports
- [[Claude Sidecar]] — a different desktop answer (parallel window over OpenCode) to the same "agents need more than a terminal" question
- [[Bram]] — a desktop shell that adds enforcement; pi-gui stops at visibility

---
*Sources: [[raw/pi-gui-com]], [[summary/pi-gui-com]]*
*Last updated: 2026-08-14*
