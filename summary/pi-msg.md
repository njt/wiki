---
url: https://github.com/zachpmanson/pi-msg
title: pi-msg
author: Zach Manson (zachpmanson)
site: github.com
date_fetched: 2026-08-06
topics:
  - coding-agents-and-frameworks
---

# pi-msg

A Go bridge that drives the Pi coding agent entirely from an XMPP chat client — 1:1 or in a group chat (MUC). It launches `pi --mode rpc` as a child process and bridges Pi's JSONL event stream to XMPP via the mellium library: the assistant's replies are relayed as chat messages, and chat messages drive the agent — plain prompts and slash commands, exactly as if typed into Pi locally.

The project is a single Go binary (~4,700 lines of source) plus one embedded TypeScript companion extension (~140 lines) that registers two XMPP-native tools (`send_reaction`, `send_file`) on the Pi side. It supports multi-account config, group chat with a two-axis trigger×authority model, ambient context buffering for untriggered room chatter, explicit `to:`-based reply routing with allowlisting, three-axis presence signaling (typing indicator, availability show, activity status), XEP-0444 emoji reactions, XEP-0363 file upload, read receipts (XEP-0184, XEP-0333), and avatar publishing (XEP-0153).

Auth uses SASL SCRAM-SHA-256 over STARTTLS. Requires Go ≥ 1.26 and a `pi` binary on PATH. MIT licensed.
