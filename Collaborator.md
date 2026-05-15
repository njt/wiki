# Collaborator

A native desktop app (Electron) for agentic development that arranges terminals, context files, and running code on an infinite canvas. No context switching, no tab hunting -- your agents and your work, side by side. Early-stage, available for macOS, Windows, and Linux.

---

## Key Themes

#devtools #agentic-coding

The architecture: Electron 40 with multi-webview, React 19, xterm.js backed by persistent node-pty sidecar, Monaco Editor for code, BlockNote/TipTap for markdown, D3 for graph visualization. All data stored locally on disk. No accounts required.

Key concepts:

**Navigator** -- resizable sidebar with file tree and workspace switcher. Supports multiple workspaces, hierarchical or chronological views, search via Cmd+K.

**Canvas** -- infinite pan-and-zoom surface with dot grid. Tiles snap to grid. Zoom range 33-100%.

**Tiles as live views** -- file tiles are bound to files on disk (update when renamed, close when deleted, reload when changed). Terminal tiles are bound to persistent PTY sessions that persist independently.

The canvas metaphor addresses the spatial problem that [[Zero Alignment]] identifies: when you're running multiple agents, you need to see them all at once, not switch between tabs. [[Collaborator]] is the "sweet UI" answer where others (tmux, Dorothy) are the utilitarian answer.

## Critical Analysis

The infinite canvas approach to development is appealing. Spatial arrangement is a genuine cognitive tool -- placing related terminals and files near each other creates visual context that tab-switching destroys.

The concern: Electron for a development environment means significant memory overhead, especially with multiple terminals and Monaco editors open. The "all data stored locally" is a plus for privacy but limits collaboration. And at early stage, the question is whether the canvas metaphor has enough staying power to sustain a product, or whether it's a novelty that fades once the workspace gets large enough that you can't find things.

---
*Sources: [[raw/collaborator]]*
*Last updated: 2026-05-14*