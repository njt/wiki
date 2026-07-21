---
url: https://brightsec.com/research/detecting-ansi-escape-sequence-injection-in-mcp-servers-with-dast/
title: Detecting ANSI Escape Sequence Injection in MCP Servers with DAST
author: Bright Security
date_fetched: 2026-07-21
date_published: unknown
---

# Detecting ANSI Escape Sequence Injection in MCP Servers with DAST

ANSI escape sequences are invisible control codes that tell a terminal to change colors, hide text, clear the screen, or move the cursor. A human reading rendered terminal output never sees the codes, but a language model reads every byte — and that gap is the core of the attack.

Two real-world CVEs as precedent: Kubernetes `kubectl` CVE-2021-25743 (control sequences in Event strings spoofing terminal output) and Git CVE-2024-52005 (remote sideband messages printed without neutralizing terminal controls).

In the Model Context Protocol (MCP), where servers stream text into an agent, "every field the model reads is a potential injection surface."

The article introduces the term **ANSI Escape Sequence Injection (AESI)** and covers two variants: direct-fetch AESI and stored AESI.

## What ANSI Escape Sequences Are

ANSI X3.64 standard unified terminal control across incompatible hardware terminals. An escape sequence starts with ESC (hex `0x1B`) followed by a command code. Two categories:

- **Concealment codes** — make text invisible
- **Screen and cursor codes** — clear display, move cursor, erase lines

"Rendering hides these bytes from people, but a model processes the raw text."

## Attack 1: Direct-fetch AESI

Many MCP servers expose a tool that fetches a URL and returns its contents. The attack flow:

1. Attacker hosts content laced with ANSI escape sequences and hidden instructions at a controlled URL.
2. That URL is supplied to a fetch-style tool as an argument.
3. The server retrieves content and relays it into a model-consumable field.
4. The model reads concealed instructions and acts on them.

Model-consumable fields are the specific response locations an agent ingests:
- tool results — text inside `result.content[].text`
- resource reads — text inside `result.contents[].text`
- prompt templates — text inside `result.messages[].content.text`

"A payload that surfaces anywhere else is not exploitable." This is a form of cross-prompt injection.

## Attack 2: Stored AESI

The attacker writes the payload into storage through one entrypoint and it detonates later when a different entrypoint reads that data back into a model field.

Three properties making stored AESI worse:
1. **Persistence** — payload sits in storage and fires again on future sessions, potentially against other users.
2. **Decoupling** — write and read are different entrypoints, possibly using different protocols (e.g., written via HTTP, read via MCP).
3. **Delayed exposure** — nothing looks wrong at write time; injection only activates on a later, unrelated read.

"You cannot infer these write-then-read paths from parameter names, tool descriptions, or protocol type."

## Security Impact

- **Unauthorized agent actions** — concealed instructions cause model to invoke tools, disclose data, alter analysis, or produce attacker-controlled output.
- **Human-in-the-loop bypass** — ANSI concealment hides instructions from the user supervising or approving the agent.
- **Log and audit trail manipulation** — screen-clear, cursor-movement, and line-erasure sequences spoof terminal/log-viewer output.
- **Persistent injection risk** — in stored AESI, payload remains in application data and can reappear through later MCP tools, resources, or prompts.

"One poisoned record in a shared knowledge base can influence every later agent workflow that reads it."

## Automating Detection with DAST

### Design Principles

1. **Inspect raw bytes** — any rendering, sanitizing, or normalizing before inspection destroys the signal.
2. **Only model-consumable fields count** — the classifier walks JSON paths the model actually ingests and ignores everything else.
3. **Unique trigger markers** — each payload carries its own improbable marker for black-box confirmation.
4. **Three-signal confirmation** — a field is vulnerable only when an ANSI escape byte, a fixed instruction phrase, and the payload's trigger all survive together in the same model-consumable field.
5. **Recurse into encoded values** — the classifier descends into stringified JSON so payloads wrapped inside JSON-encoded fields are still caught.

Detection is binary — the three-signal rule holds or it doesn't.

### The Payload Corpus

A small corpus of hosted payload files exercises different ANSI techniques: concealment, background-matched foreground, clear-screen, cursor-erase, and an HTML-wrapped variant, each with its own trigger. "Variety matters because a server that strips one technique may still relay another."

### Direct-fetch Detection Pipeline

Targets MCP action-request entrypoints (tools/call, resources/read, prompts/get) with at least one parameter. Treats a broad set of text parameter types (string, URL, email, phone, file-URI) as injectable.

Steps:
1. **Inject** — substitute the payload's hosted URL into an injectable text parameter and invoke the entrypoint.
2. **Capture** — record the full MCP response as raw bytes; unsuccessful or JSON-RPC error responses are skipped.
3. **Classify** — walk model-consumable fields, apply the three-signal rule.
4. **Report** — file one issue per vulnerable parameter, recording the payload URL and field path.

### Stored Detection Pipeline

Phase 1 — Probe: Assign one unique inert marker to each injection entrypoint and write it embedded in benign carrier text. Injection candidates are MCP tools/call or prompts/get entrypoints with a text parameter, or HTTP POST/PUT/PATCH endpoints with a writable text parameter.

Phase 2 — Map reflections: Read back every reflection candidate and scan raw responses for probe markers. When a marker is found, the engine records the exact JSON field path. The result is a map from injection entrypoints to read surfaces and field paths.

Phase 3 — Attack and confirm: For each mapped path, write the real AESI payload, re-read the mapped MCP reflection surfaces, and apply the classifier only at the recorded field path.

The probe phase "converts an unknown source-to-sink topology into a concrete, field-path-anchored map."

## Defenses and Takeaways

1. **Sanitize control bytes** out of fetched or stored content before it reaches a model-consumable field.
2. **Validate ingestion** — URL allow-lists and input validation on write endpoints.
3. **Treat all external and stored text as untrusted** the moment it can reach the model.
4. **Scan continuously** — automated, byte-level, field-aware detection in CI and DAST against MCP surfaces.

"Effective detection has to model the *data flow*, not just individual requests."

## Conclusion

"AESI shows that AI agents inherit every ambiguity of the formats they consume: a 1970s cursor code becomes an injection primitive the moment a model, not a terminal, reads the bytes." As MCP feeds agents more untrusted text, future attacks will come less from novel exploits than from "old representation quirks reaching a consumer never meant to interpret them."

"Detection belongs in DAST because survival depends on what the live server fetches, stores, strips, and resurfaces — none of it visible from the code. The concealment that makes AESI dangerous to a human reviewer is exactly what makes it tractable to automate: the model sees every byte, and so should the scanner."
