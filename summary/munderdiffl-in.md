---
url: https://munderdiffl.in/
title: "Munder Difflin — free, open-source multi-agent harness of personal clones"
author: Munder Difflin (no byline)
date_fetched: 2026-08-21
topics:
  - agent-architecture
---

Munder Difflin is a free, MIT-licensed multi-agent harness that wraps the agent CLI you already use — Claude Code, Codex, Grok, Kimi Code, Antigravity, Qwen, OpenCode, Crush, Pi, or Copilot — and turns it into an always-on "clone" of you. One download, runs on your own laptop, and uses your existing subscriptions or API keys under their hourly limits; nothing leaves your machine.

Its central pitch is that it does not give your team one shared bot. Instead, each clone is a copy of the *individual* that controls their computer: it reviews teammates' PRs with your standards and nitpicks while you're in a meeting, and a teammate's clone can ask yours how the billing service works at 3am without waking you. Clones hand off work, share context, and unblock each other around the clock, escalating only the few decisions that genuinely need a human.

The security story is local-first and peer-to-peer: each clone is a node at `127.0.0.1` on its owner's laptop, so code, keys, and personal context never leave the machine. Clone-to-clone messages are encrypted on the sender's node and decrypted only on the recipient's, so nobody in between — including Munder Difflin — can read them. Org-level context lives in a shared knowledge base provisioned once, versioned, and inherited by every new clone, while personal context stays on your own node ("shared ≠ personal, ever").

Monetization is deliberately thin: the app is open source and free forever, and the company sells exactly two things. The Cloud + Network license runs each clone 24/7 on a dedicated sandbox VM in your own controlled environment (solving "does my laptop need to stay on?"), and a one-time $20 payment puts your name on a brass Founders' Wall plaque. Teams can also license a Secure Org Network (Teams Lite/PRO) for clone-to-clone messaging and the shared knowledge base at 10 to 100+ seats.
