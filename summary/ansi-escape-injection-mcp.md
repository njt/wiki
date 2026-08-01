---
url: https://brightsec.com/research/detecting-ansi-escape-sequence-injection-in-mcp-servers-with-dast/
title: "Detecting ANSI Escape Sequence Injection in MCP Servers with DAST"
author: Bright Security
date_fetched: 2026-07-21
---

ANSI escape sequences are invisible terminal control codes (cursor movement, screen clear, color changes) that a human never sees in rendered output but that a language model reads as raw bytes. Bright Security coins the term **ANSI Escape Sequence Injection (AESI)** for attacks exploiting this gap in MCP-based AI agents, where servers stream untrusted text into model-consumable fields.

Two attack variants are described. **Direct-fetch AESI** targets MCP servers that expose tools to fetch URLs: an attacker hosts content laced with concealed instructions at a controlled URL, supplies it to a fetch tool, and the server relays the payload into `result.content[].text` or equivalent fields where the model reads and acts on hidden instructions. **Stored AESI** is worse — the attacker writes a payload into application storage through one entrypoint, and it detonates later when a different MCP tool, resource, or prompt reads that data back into a model field, potentially affecting multiple users across sessions.

The security impact spans unauthorized agent actions, bypass of human-in-the-loop supervision (concealed instructions are invisible to the approving human), log and audit-trail manipulation via screen-control sequences, and persistent injection where one poisoned record in a shared knowledge base influences every future agent workflow.

The article proposes a DAST-based detection approach with five design principles: inspect raw bytes (rendering destroys the signal), only classify model-consumable JSON field paths, use unique trigger markers per payload for black-box confirmation, require three-signal confirmation (ANSI byte + instruction phrase + trigger marker all survive together), and recurse into stringified JSON. Detection is binary — the three signals are all present in a model-consumable field or they aren't.

For stored AESI, a three-phase pipeline first probes every write entrypoint with inert markers, maps which read surfaces reflect them back, then attacks only confirmed source-to-sink paths. The key insight is that "survival depends on what the live server fetches, stores, strips, and resurfaces — none of it visible from the code."
