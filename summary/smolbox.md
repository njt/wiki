---
url: https://remyhax.xyz/posts/smolbox/
title: Agentic AI in a Smolbox
author: remyhax
site: remyhax.xyz
date_fetched: 2026-08-21
date_published: unknown
topics:
  - security-and-sandboxing
---

# Agentic AI in a Smolbox

Smolbox runs entirely in a browser tab: a full x86_64 virtual machine (Alpine Linux) under WebAssembly, plus an LLM that runs via WebGPU and is exposed to the VM as tool calls. You can optionally mount a local folder read-only at `/mnt/host` using the browser file picker. The result is "very similar to Claude Code or Codex, except that there's no server side processing" — the sandbox, the model, the tool calls, and the file access all live in the user's tab, private and collecting no data.

The author is explicit about authorship: they knew exactly what they wanted and how to validate it, and "AI put all the pieces together exactly as I asked with zero ambiguity." Smolbox is framed as a proof of concept for two claims — that this was always possible, and that "the state of AI security" is doing a disservice to the next generation of users by locking agentic AI behind subscriptions, servers, and guardrail-dodging.

The emotional core is an origin story. As a teenager in the 2000s, the author reverse-engineered a Verizon MMS↔email bridge to browse the web over unlimited text messages, then used that access to find a path-traversal vulnerability in the phone's ringtone loader and overwrite the built-in World Clock app with a mini-golf game. The lesson they draw: a random developer's small, slow, kludgy service "made something *possible*," and that is what shaped their entire career.

Smolbox is slow, kludgy, and "not particularly great at anything other than being private and not collecting any data whatsoever." But it makes agentic AI *possible* on a school-issued iPad, a library computer, or a hand-me-down laptop — no credit card, no install, no special permissions — which is precisely the path the author argues a resourceful teenager would otherwise find through hacked accounts and Telegram vendors instead.
