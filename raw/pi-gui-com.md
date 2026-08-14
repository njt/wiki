---
url: https://www.pi-gui.com/
date_fetched: 2026-08-14
---

# A native desktop for

AI coding agents

pi-gui is a Codex-style macOS and Linux desktop app for the pi coding agent. Manage workspaces, run sessions, and review agent work — all from a native interface.

Beta for macOS arm64 and Linux AppImage. Install from GitHub Releases on either platform, or Homebrew on macOS.

From quick fixes to complex refactors, pi-gui gives you a persistent desktop workspace for AI-powered coding sessions — with full visibility into what the agent is doing and why.

Features

## Everything you need in one window

A desktop-native experience built for multi-project AI coding workflows.

### Multi-workspace sessions

Open project folders as workspaces, each with independent session histories. Context-switch between projects without losing state.

### Real-time agent timeline

Watch every tool execution, code change, and reasoning step in a scrollable timeline with full input and output detail.

### Persistent session history

Sessions survive restarts. Resume any previous conversation, review transcripts, and continue exactly where you left off.

### Skills & slash commands

Extend pi-gui with workspace-specific skills and slash commands for model switching, thinking levels, settings, and custom workflows.

Architecture

## Built for durability

The desktop shell is separated from the agent runtime through a durable SessionDriver interface — making the frontend independent of backend changes and ready for future runtime swaps. Built on top of @earendil-works/pi-coding-agent.

Get started

## Install the beta your way

```
# Direct install from GitHub Releases
open https://github.com/minghinmatthewlam/pi-gui/releases/latest
# Or install with Homebrew
brew tap minghinmatthewlam/tap
brew install --cask pi-gui
# Later, update the Homebrew install
brew upgrade --cask pi-gui
# Source install is for local development
git clone https://github.com/minghinmatthewlam/pi-gui.git
cd pi-gui
# Install dependencies and run
pnpm install
pnpm dev
```
DMG installs update from GitHub Releases. Homebrew installs update with `brew upgrade --cask pi-gui`. During beta, Homebrew upgrades may require re-confirming some macOS permissions or Dock placement after reinstall-style updates.
