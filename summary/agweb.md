---
url: https://github.com/paul-nameless/agweb
title: "agweb — A browser for coding agents on macOS"
author: paul-nameless
date_fetched: 2026-11-26
date_published: 2026-11
topics:
  - coding-agents-and-frameworks
  - developer-tools
---

agweb is a macOS browser built on WebKit (Safari's engine) designed to be driven by coding agents rather than people. It ships as one always-visible window that never steals focus, plus a `playwright-cli`-style shell command that agents use to open pages, click, fill forms and read content. Multiple agents work at once, each in its own named workspace with its own tabs, grouped in a sidebar and shown together in a ⌘D dashboard.

The pitch is a trade Playwright never made: keep the browser *headed* (so anti-bot walls and visual debugging work) but make it perfectly unobtrusive. Dialogs, file pickers, downloads and tab changes are answered in code, not by windows popping up. Idle tabs are throttled, then their web content processes are SIGKILLed entirely (590 MB for five idle pages vs 1.6 GB for Playwright's Chrome), and reloaded transparently on next use.

Agent ergonomics are the real product: snapshots give each element a `[ref=eN]` handle that stays valid while it's on the page, every action reply includes a diff of what changed (so no re-snapshot per step), `markdown` strips base64 images (758 KB → 23 KB on a Google results page) and caps output at 50 KB / 2000 lines. Secrets are typed via `--secret-cmd 'pass show site'` so they never enter the agent's transcript; passkeys, per-profile SOCKS/HTTP proxies, uBlock list ad-blocking, fake camera/mic/geolocation, and MP4 demo recording with a human-like pointer round it out. The repo is ~12,000 lines of Swift (an app, a core library, and the CLI), with a published benchmark history and an unusually candid CLAUDE.md documenting WebKit SPI hacks and known gaps (no CDP, no Chrome extensions).
