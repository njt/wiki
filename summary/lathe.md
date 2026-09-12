---
url: https://github.com/devenjarvis/lathe
title: Lathe — LLM-Generated Hands-On Technical Tutorials
author: Deven Jarvis
date_fetched: 2026-06-12
date_published: 2026-05
topics:
  - misc
---

# Lathe — Full Analysis

## What it is

Lathe generates hands-on, multi-part technical tutorials on demand using LLMs, then serves them in a purpose-built local web UI designed for pleasant reading. The philosophical inversion: LLMs *teach you* rather than *think for you*. Built by Deven Jarvis as an experiment in using AI to recreate the hands-on learning experience that taught him programming as a teen in the PSP homebrew community.

## Architecture

Lathe is a **two-component system** with a strict boundary:

1. **Go CLI binary** (`lathe`) — owns all durable state. Stores tutorials in `~/.lathe/tutorials/<slug>/` (each a directory with `metadata.json` + `part-NN.md` files), serves a local web UI at `http://localhost:4242`, manages writing voices at `~/.lathe/voices/`. Never drives a model — never spawns a `claude` subprocess, never calls an LLM API.

2. **LLM skills** — six `SKILL.md` files (embedded in the binary and installed into agent directories) that run in the user's interactive coding-agent session:
   - `/lathe` — generates `part-01.md` (416 lines of meticulous prompt engineering)
   - `/lathe-extend` — writes the next part of a series
   - `/lathe-verify` — follows the tutorial end-to-end in a `mktemp -d` scratch dir, records results
   - `/lathe-ask` — answers reader questions grounded in the tutorial's concrete artifact
   - `/lathe-tag` — picks/backfills search tags
   - `/lathe-voice` — interviews the user to author a custom writing voice

The boundary enforces a **handoff model**: the web UI buttons and CLI commands print paste-able slash commands (`/lathe-verify <slug>`). The user pastes them into their coding agent session. The skills do the model work and call back into `lathe` CLI commands (`lathe store`, `lathe verify-result`, `lathe extend-start`/`extend-commit`, `lathe tag`, `lathe voice add`) to mutate state. The binary never drives a model; the skills never write to `~/.lathe/` directly.

File layout:
```
main.go                           cobra entrypoint
cmd/
  root.go, serve.go, store.go, list.go, rm.go, open.go
  verify.go, extend.go             print handoff commands
  verify-result.go, extend-start.go, extend-commit.go  skill→CLI write hooks
  tag.go, version.go, skills.go, voice.go
internal/
  buildinfo/                       ldflags-injected version
  frontmatter/                     Parse()/Strip() — no YAML dep, line scanner
  skills/                          //go:embed data mirror + catalog + Cursor translation
  voice/                           embedded presets + List/Resolve/Add/Remove + guardrail Preamble
  config/                          ~/.lathe paths + config.json
  store/                           Store(), Delete(), metadata.json read/write, normalization
  serve/                           net/http server, goldmark+chroma renderer, HTML templates, CSS
  extend/                          NextPartFilename helper
```

Skills are edited in `.claude/skills/<name>/SKILL.md`, then `mage skills` generates a tracked mirror under `internal/skills/data/` (because `go:embed` ignores dot-prefixed paths).

## Key techniques

### Prompt engineering as infrastructure
The `/lathe` skill is 416 lines of meticulously crafted instructions that read like a writing textbook crossed with a style guide. Key techniques within it:

- **Faded scaffolding**: First code block is fully worked (reader copies, runs, sees output). Last code blocks shift to fill-the-seam: name the pattern, ask reader to write the next instance using it.
- **Spaced retrieval (RECALL callout)**: Part N≥2 opens with a `> [!RECALL]` prompt asking the reader to reconstruct a load-bearing concept from a prior part. Forces retrieval, not recognition.
- **Prediction beats (PREDICT callout)**: Before a Checkpoint, asks "what output do you expect?" so the reader commits to an answer before seeing it.
- **Closing reflection**: Every part ends with "in two sentences, why does X beat Y? Write the answer that satisfies a sceptical colleague." Forces construction, not recognition.
- **Ground or flag**: Every load-bearing claim must be either cited to a source the LLM actually read (deep-linked to section/anchor, not homepage), or flagged `[!UNVERIFIED]` with what to check.
- **Show-wrong-then-fix**: When introducing a concept, demonstrate the tempting-but-broken way first, mock it in one sentence, then show the fix. "The reader needs to *feel* why the fix matters."
- **Pre-store gate**: Before `lathe store`, the skill must explicitly state (or give a justified opt-out for) repo, versions, tags, sources, voice, and model. This is a prompt-level lint rule preventing silent metadata drops.

### Voice partition system
Writing voices control **tone and register only** — never accuracy, research, citation, verification, or pedagogy. The invariants live in the skill and always win on conflict. Two built-in voices:
- `plainspoken` (default): honest, precise, no invented persona, no fabricated first-person war stories
- `companion`: warm, wry, first-person "friend at the keyboard"

Custom voices are authored via `/lathe-voice`. Security boundary: every voice spec returned by the read path gets a fixed, non-overridable `Preamble` prepended at Go code level (`voice.Wrapped()`) that states the guardrails. A hostile custom voice file can't escape the framing — the Preamble is code, not prompt.

### Callout preprocessor
Eight `> [!TYPE]` blockquote types, each with distinct semantic meaning and CSS styling: NOTE, TIP, WARNING, HEADS-UP, ASIDE, DESIGN-NOTE, PREDICT, RECALL, UNVERIFIED. Before goldmark renders markdown, `preprocessCallouts` uses regex to rewrite them into raw `<aside>` HTML blocks with CSS classes (`callout-note`, `callout-warn`, etc.). This gives styled callouts without a custom markdown extension.

### Atomic file writes
`writeJSONFile` writes to a temp file in the same directory (`os.CreateTemp`), then `os.Rename` into place. Prevents torn writes from leaving corrupt `metadata.json` or `verify-result.json`.

### CSRF on loopback
Despite binding to `127.0.0.1` only, destructive POST endpoints (`/-/delete`, `/-/verify`, `/-/extend`, etc.) check `Origin`/`Referer` headers against `localhost`/`127.0.0.1`/`::1`. Defends against other devices on the LAN POSTing to the predictable port.

### Client-side search/filter with progressive enhancement
List page is server-rendered as a flat newest-first list with `data-*` attributes on each card. All search (title/topic/tags/repo/tools), status/type/tag/version filtering, and sorting happens in the browser via an inline `<script>`. Without JS, every card stays visible (`.hidden` is only ever set by the script).

### Fully offline UI
Fonts (Fraunces, Newsreader, JetBrains Mono — latin-subset woff2), mermaid.js, KaTeX (math typesetting with 20 font variants), and all CSS are `go:embed`'d into the binary. Zero external requests. LaTeX math uses goldmark's `passthrough` extension at AST level (so `$` inside code blocks is never treated as math), rendered client-side by KaTeX.

### SKILL.md as cross-tool standard
Lathe treats `SKILL.md` (name + description frontmatter) as a cross-tool standard. Every target except Cursor gets the raw skill verbatim — Claude Code, Codex, Gemini CLI, opencode, Cline, and Windsurf all read it as-is. Cursor is the lone translation case (its commands are slash-invoked as `/<slug>`).

### Git remote normalization
`NormalizeRepo` canonicalizes any git remote URL form (`https://`, `git@`, `ssh://`, with/without `.git`, with/without port) into a stable `host/org/repo` grouping key. Handles the scp-style `git@github.com:org/repo.git` edge case by distinguishing scheme URLs (where `:` after host is a port) from scp short forms (where `:` is the host/path separator).

## Design decisions

### No model in Go — the handoff model
**What they chose**: The Go binary is pure state management + static serving. All LLM work happens in the user's interactive coding-agent session via skills that call back into the CLI.

**Why**: Keeps the binary off metered headless runs (Claude Code's `claude -p` is metered as of 2026-06-15; interactive sessions are not). Keeps model work on whatever subscription the user already has. Keeps the binary simple — no API client, no streaming, no retry logic.

**Trade-off**: Less seamless UX. The user must paste commands between the web UI and their agent session, rather than clicking a button and having the tutorial generate inline.

### Verification is opt-in, not automatic
**What they chose**: Tutorials store as `unverified` by default. The user explicitly invokes `/lathe-verify <slug>` to have the LLM follow the tutorial in a scratch dir.

**Why**: Running arbitrary code automatically is dangerous. The user should see and approve the tool calls. Also, verification only works when the tutorial's toolchain is installed — auto-running would produce confusing failures.

**Trade-off**: Unverified tutorials sit in the library with no indication of whether they actually work.

### One part per invocation
**What they chose**: `/lathe` always produces exactly one `part-01.md`. Additional parts require `/lathe-extend`.

**Why**: Gives the reader control over pacing and direction. Avoids the LLM writing all 6 parts in one shot (where quality degrades with length). Each part gets the same research discipline.

**Trade-off**: More friction to get a complete series.

### Local-first, loopback-only
**What they chose**: Server binds to `127.0.0.1` only. No authentication. No cloud sync. No sharing.

**Why**: Tutorials are personal learning artifacts. This is a deliberate scope constraint for v1. The loopback-only binding + CSRF checks provide reasonable security for a local tool.

**Trade-off**: No remote access, no team features, no multi-device sync.

### "Teaches you, doesn't think for you"
**What they chose**: The entire product is built around the pedagogical philosophy that the human should type the code, make mistakes, and learn from them. The LLM is a guide, not a surrogate.

**Why**: The author learned programming through hands-on tutorials and found that LLMs doing the work for you takes away the learning. This is an experiment in whether LLMs can be effective *teachers* rather than *doers*.

**Trade-off**: Lathe tutorials are slower to work through than just having an LLM write the code. That's the point — but it's not for everyone or every use case.

### Vibecoded but stable
**What they chose**: The author openly admits the project was "vibecoded" (built with heavy LLM assistance). The README has a section titled "Be honest, did you vibecode this?".

**Why**: The scope and risk are low — it's a personal learning tool. The author has been using it daily and it's proven stable. "I expect the next few point releases to be some intentional code/architecture clean up."

**Trade-off**: Some architectural choices may be suboptimal. The AGENTS.md file is unusually detailed (111 lines of architecture notes) — a sign of vibecoded code that needed extensive documentation to be maintainable.

## Comparison notes

### vs. Coding Agents Continuity Not Memory
Lathe's handoff model (CLI owns state, skills do work) is a different take on the continuity problem. Santi argues for repo-local, evidence-weighted continuity records with a resume-work-finalize lifecycle. Lathe solves it differently: the CLI is the continuity layer, and skills are stateless workers that read state and call back to mutate it.

### vs. The Agentic Product Standard
Lathe's skill-CLI boundary is an instance of the coordinator-worker composition pattern: skills coordinate the LLM's work, the CLI is the worker that owns durable state. The "pre-store gate" (explicitly state or opt out of every metadata field) is a prompt-level validation pattern.

### vs. Honey I Shrunk the Coding Agent
Lathe exemplifies the thesis that "the harness matters more than the model." Its prompt engineering (the 416-line skill, the voice partition, the callout system) is more important than which model runs it. The README recommends "the biggest thinking model you have access to," but the architecture doesn't depend on it.

### vs. Prompt-driven tutorial tools
Most AI tutorial tools focus on generating content for *publication* — blog posts, documentation, course material. Lathe is unusual in being designed for *personal* use, with an explicit prohibition on publishing AI-authored tutorials as human-written. The voice system's guardrails (no impersonation, no fabricated credentials, LLM authorship disclosed) enforce this.

### vs. Coding agent skills ecosystem
Lathe's skills are installable across 7 different coding agents (Claude Code, Cursor, Codex, Gemini CLI, opencode, Cline, Windsurf) using the `SKILL.md` format as a cross-tool standard. This is similar to what `sx` (team package manager for AI coding assistant assets) and `Printing Press` (CLI generator) are doing, but Lathe is specifically about tutorial generation rather than general-purpose skills.

Tags: #tool #project #agents #learning #prompt-engineering #tutorials #go
