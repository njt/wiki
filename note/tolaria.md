# tolaria

A desktop app for managing markdown knowledge bases, built on Tauri 2 (Rust backend, React frontend). Files-first, git-first, offline-first, AI-agent compatible. Think of it as an Obsidian alternative that's open source (AGPL-3.0), keyboard-centric, and designed from the ground up to work with AI agents (Claude, Codex CLI, Gemini). The creator maintains 10,000+ personal notes in it.

---

## Key Quotes

(No standout quotes -- the README is structured around design principles rather than narrative prose.)

## Key Themes

#knowledge-management #markdown #desktop-app #tools #Obsidian-alternative

The eight design principles (files-first, git-first, offline-first, open-source, standards-based, types-as-navigation, AI-compatible, keyboard-centric) read like a manifesto for how knowledge management should work. Every principle is a reaction against something: files-first against database lock-in, git-first against proprietary sync, offline-first against subscription requirements.

The AI-agent compatibility is the forward-looking feature. If your knowledge base is the context your agent operates in (see [[Memory Mechanism]] for the theory), then the editor needs to produce files that agents can read and write. Tolaria's markdown + YAML frontmatter format is exactly what agent frameworks expect.

The Tauri 2 choice (Rust backend instead of Electron) is a meaningful technical decision -- smaller binary, lower memory footprint, better native integration than Electron-based alternatives.

## Critical Analysis

At 10.6k stars and 2,547 commits, this has genuine traction, but it's competing with Obsidian (which has an enormous plugin ecosystem and community) and VS Code (which developers already have open). The question is whether "AI-agent compatible" is a strong enough differentiator to pull users from established tools.

The AGPL-3.0 license is a double-edged sword: it ensures the code stays open, but it discourages commercial adoption and integration into proprietary tools.

The git-first approach is philosophically appealing but practically challenging for non-developers. Git is a terrible UX for knowledge management if you don't already understand branching and merging. Obsidian solved this with their sync service; Tolaria leaves it to the user.

For the specific use case of "knowledge base that AI agents can read and write" -- managing CLAUDE.md files, agent memory stores, documentation for AI context -- this is a cleaner tool than Obsidian. The design constraints align perfectly with what agent frameworks need.

---
*Sources: [[summary/tolaria]]*
*Last updated: 2026-05-14*
