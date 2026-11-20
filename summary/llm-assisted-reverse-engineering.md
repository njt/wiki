---
url: https://www.slowchronicle.com/posts/llm-assisted-reverse-engineering
title: "LLM assisted reverse engineering"
author: ioma8 (slowchronicle)
date_fetched: 2026-11-20
date_published: 2026-11-19
topics:
  - agent-coding-workflow
  - software-engineering-craft
---

A veteran hacker (two decades of hardware and software RE) describes how LLMs transformed his reverse-engineering practice in 2026. The stack: DeepSeek as the model, Ghidra with an MCP bridge as the decompiler, radare2 and Kuna as faster CLI alternatives, and a set of agent skills — most importantly his own "hypothesis debugging" skill that forces a stuck agent into a scientific-method loop.

The workflow: identify the binary, have the agent document every finding to markdown files (as compaction insurance and self-notes), then map the structure. When disassembly stalls, escalate — dynamic debugging, injected logging probes, computer-use or emulator access, or a human-directed change of approach. The most distinctive advice is to read the agent's conversation traces yourself to spot blind alleys before it wastes hours.

He credits LLM assistance with six months of results "which would take me years each on its own": custom firmware for synths, mapped device protocols (cameras, biofeedback hardware), a cracked Alphasmart NEO, abandonware license cracks. A closing note on the blurring line between RE and plain software development — once a proprietary format is deciphered, it becomes a normal build.
