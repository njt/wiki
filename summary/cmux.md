---
url: https://cmux.com/
title: cmux
author: Manaflow
date_fetched: 2026-06-21
date_published: unknown
---

# cmux — Raw Ingest

Fetched from https://cmux.com/ on 2026-06-21.

## Product

cmux is a macOS-native terminal application designed for developers working with AI coding agents and multitasking workflows. Built on **libghostty** (the terminal rendering engine from Ghostty) — explicitly "not a fork of Ghostty" but uses its rendering library similarly to how apps use WebKit.

Written in **native Swift + AppKit** — "no Electron." GPU-accelerated rendering via libghostty. macOS only.

## Features

- **Vertical tabs in a sidebar** showing git branch, working directory, ports, and notification text
- **Notification rings** — panes visually highlight when processes need attention, with unread badges, a notification popover, and macOS desktop notifications (triggered via OSC 9/99/777 escape sequences or the cmux CLI)
- **In-app browser** that can be split alongside the terminal, with a scriptable API
- **Split panes** (horizontal and vertical) within each tab
- **CLI and socket API** for automation and scripting
- **Keyboard shortcuts** for workspaces, splits, browser, and more (terminal keybindings come from a Ghostty config file at `~/.config/ghostty/config`; cmux-specific shortcuts are customizable in Settings)

## Terminal Compatibility

Any agent that "runs in a terminal works out of the box," including Claude Code, Codex, OpenCode, Gemini CLI, Kiro, Aider, Goose, Amp, Cline, Cursor Agent, and others.

## Maker

Created by **Manaflow** (references `manaflow-ai` on GitHub, contact `founders@manaflow.com`). No individual founders named.

## Pricing

**Free** — source code available on GitHub. No paid tiers.

## Positioning vs. Alternatives

- **vs. Ghostty**: cmux is a different app using Ghostty's rendering engine as a library
- **vs. tmux**: tmux "is a terminal multiplexer that runs inside any terminal" whereas cmux is a native macOS GUI app with vertical tabs, split panes, embedded browser, and socket API — no config files or prefix keys needed
- **vs. iTerm2, Warp, VSCode**: Multiple community testimonials describe switching from these tools

## Other

Translated into 20+ languages. Site includes changelog, blog, nightly builds, community links (GitHub, X/Twitter, Discord), and standard legal pages.
