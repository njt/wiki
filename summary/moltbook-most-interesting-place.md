---
url: https://simonwillison.net/2026/Jan/30/moltbook/
title: "Moltbook is the most interesting place on the internet right now"
author: Simon Willison
date_fetched: 2026-05-15
date_published: 2026-01-30
---

Simon Willison's piece on Moltbook, a social network where AI agents interact with each other, bootstrapped via OpenClaw's skills system. Covers the mechanics (heartbeat-driven checkins, skill-based installation, Submolt forums), examples of bot activity (Android automation, security scanning, content filtering anomalies), and the central safety question: can we build a safe version of this?

## Background: OpenClaw

Formerly Clawdbot, then Moltbot, now OpenClaw — an open-source digital personal assistant by Peter Steinberger. ~2 months old at time of writing, 114,000+ GitHub stars. Uses a skills system: zip files with markdown instructions and optional scripts. Willison flags prompt injection as his "current pick for most likely to result in a Challenger disaster."

## How Moltbook Works

Installation: share a skill URL with your agent:
```
https://www.moltbook.com/skill.md
```

The markdown file includes commands to create skill directories and curl endpoints for API interactions (account registration, posting, commenting, creating Submolt forums like m/blesstheirhearts and m/todayilearned).

Heartbeat system: bots check in every 4+ hours by fetching and following instructions from heartbeat.md. Willison flags the rug-pull risk.

## Bot Activity Examples

- Automating an Android phone via ADB over Tailscale
- Security scanning: 552 failed SSH logins, Redis/Postgres/MinIO on public ports
- Webcam watching via streamlink and ffmpeg
- Content filtering anomaly: Claude Opus 4.5 agent reports corruption when trying to explain PS2 disc protection

## The Safety Question

Willison hasn't installed OpenClaw himself, referencing his 2023 warning about "a rogue digital assistant." Despite concerns, people report real value: one user had Clawdbot buy a car by negotiating with dealers via email; another used it to transcribe voice messages via FFmpeg and Whisper API.

People buy dedicated Mac Minis to run OpenClaw in isolation. The "lethal trifecta" still applies when connecting to personal data.

Central question: "whether we can figure out how to build a safe version of this system." Normalization of Deviance means people keep taking bigger risks. DeepMind's CaMeL proposal is promising but 10 months old without a convincing implementation.

Tags: ai, tailscale, prompt-injection, generative-ai, llms, claude, ai-agents, ai-ethics, lethal-trifecta, skills, peter-steinberger, openclaw
