---
url: https://github.com/aispace-sh/aispace-client
title: aispace
author: aispace-sh (Luigi Agosti)
date_fetched: 2026-09-08
date_published: 2026-09-08
source_type: GitHub repository
topics:
  - developer-tools
---

aispace is a single-binary Go CLI for "bot-friendly file drops" — temporary file storage designed to be driven by AI agents rather than humans. It uploads files (or stdin) to a hosted service with a raw streaming POST, optionally mints separately-expiring public share links, and can encrypt payloads client-side with age X25519 so the storage operator never sees plaintext. The server, billing, and deployment are operated separately; this repo holds only the client, an agent skill, and integration examples.

The design is automation-first: exact JSON output (errors go to stderr as structured JSON), a share URL printed last on its own line so shell scripts and agents can capture it, five stable exit codes (0 success, 2 usage, 3 auth, 4 quota, 5 rate-limit), and no interactive prompts. Config resolves flag > env > file > default; the API key travels as `AISPACE_KEY` or via `aispace login`.

Under the hood it's thin and idiomatic: `cmd/` (cobra commands) over `internal/api/` (a ~600-line HTTP client) over `internal/config` and `internal/duration`, with only two direct dependencies — `filippo.io/age` and `spf13/cobra`. Notable engineering: per-phase HTTP timeouts (dial/TLS/header/transfer-inactivity) with no whole-request deadline, a reset-on-progress stall detector, a deliberately conservative retry policy that never replays downloads, and overflow-safe duration parsing.

An agent gets it as a Codex-compatible skill (`skills/aispace/SKILL.md`), an OpenAI/Anthropic tool schema, and a Python handler in `docs/LLM_USAGE.md`, all steering the same CLI.
