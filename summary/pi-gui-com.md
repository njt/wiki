---
url: https://www.pi-gui.com/
title: "pi-gui — A Native Desktop for the Pi Coding Agent"
author: minghinmatthewlam (GitHub)
date_fetched: 2026-08-14
date_published: unknown
---

pi-gui is a "Codex-style" native desktop app for the `pi` coding agent, in beta for macOS arm64 and Linux (AppImage), installed from GitHub Releases or Homebrew cask. It wraps `@earendil-works/pi-coding-agent` behind a desktop shell that manages workspaces, runs sessions, and reviews agent work.

Four features define the surface: **multi-workspace sessions** (each project folder is a workspace with independent session history, so context-switching doesn't lose state); a **real-time agent timeline** (every tool execution, code change, and reasoning step in a scrollable view with full input and output detail); **persistent session history** (sessions survive restarts — resume a conversation, review transcripts, continue where you left off); and **skills & slash commands** (workspace-specific extensions for model switching, thinking levels, settings, and custom workflows).

The architectural claim is durability through decoupling: a `SessionDriver` interface separates the desktop shell from the agent runtime, keeping the frontend independent of backend changes and "ready for future runtime swaps." Distribution runs the full spectrum — GitHub Releases (DMG), Homebrew cask (`brew install --cask pi-gui`), and source (`pnpm install && pnpm dev`). During beta, Homebrew upgrades may require re-confirming macOS permissions or Dock placement after reinstall-style updates.
