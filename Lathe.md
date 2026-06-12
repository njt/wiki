# Lathe

Lathe generates hands-on, multi-part technical tutorials on demand using LLMs, then serves them in a purpose-built local web UI. The philosophy is the inversion: **LLMs teach you, don't think for you**. Built by Deven Jarvis as an experiment in whether AI can recreate the hands-on learning experience — faded scaffolding, spaced retrieval, prediction beats — that taught him programming as a teen in the PSP homebrew community.

The system is a Go CLI that owns all durable state (`~/.lathe/tutorials/`) + a set of six LLM skills that run in your interactive coding-agent session. The skills do the model work (generating, verifying, extending, answering questions); the CLI stores and serves the results. They never cross: the binary never drives a model, the skills never write to `~/.lathe/` directly. It's a handoff model — the web UI buttons print paste-able slash commands.

## Architecture

Lathe is a **two-component system with a strict boundary**:

**Go CLI** (`main.go` → `cmd/` → `internal/`): cobra-based, embeds everything (fonts, CSS, JS, skills) via `//go:embed`. Serves a local web UI at `localhost:4242` using Go 1.22+ method-and-pattern routing. Stores tutorials as directories of `metadata.json` + `part-NN.md` files at `~/.lathe/tutorials/<slug>/`. Manages writing voices at `~/.lathe/voices/<name>.md`. Never spawns a `claude` subprocess, never calls an LLM API.

**Skills** (`.claude/skills/<name>/SKILL.md`, embedded in the binary): six Markdown files installed into agent directories via `lathe skills install`. Designed for Claude Code but shipped as raw `SKILL.md` to 6 other agents (Cursor, Codex, Gemini CLI, opencode, Cline, Windsurf) using the format as a cross-tool standard:

| Skill | Trigger | What it does |
|---|---|---|
| `/lathe` | `/lathe build a 3D slicer in Erlang` | Generates `part-01.md` with full research discipline |
| `/lathe-extend` | `/lathe-extend <slug>` | Writes the next part, continues the same voice/example/numbers |
| `/lathe-verify` | `/lathe-verify <slug>` | Follows tutorial in a `mktemp -d` scratch dir, records verified/failed/skipped |
| `/lathe-ask` | `/lathe-ask <slug> <part>` | Answers reader questions grounded in the tutorial's concrete artifact |
| `/lathe-tag` | `/lathe-tag <slug>` | Picks/backfills 2-5 lowercase search tags |
| `/lathe-voice` | `/lathe-voice` | Interviews user to author a custom writing voice |

Key source files: `internal/store/store.go:34-89` (Store with normalization), `internal/store/metadata.go:22-71` (Tutorial struct), `internal/serve/server.go:39-81` (Server + routing), `internal/serve/renderer.go:63-106` (goldmark + chroma + callout/Mermaid preprocessors), `internal/voice/voice.go:37-62` (Preamble guardrail), `internal/frontmatter/frontmatter.go:16-39` (line-scanner, no YAML dep).

## Key techniques

### Prompt engineering as infrastructure
The `/lathe` skill (`.claude/skills/lathe/SKILL.md`) is 416 lines of meticulously crafted instructions. It reads like a writing textbook: pedagogy techniques from educational psychology, enforced through prompt structure rather than code:

- **Faded scaffolding**: First code block fully worked → last blocks shift to "you've seen the pattern, now write the mirror image"
- **Spaced retrieval**: Part N≥2 opens with `> [!RECALL]` — a question forcing reconstruction of a prior concept
- **Prediction beats**: Before `## Checkpoint`, `> [!PREDICT]` asks "what output do you expect?" so the reader commits before seeing the answer
- **Closing reflection**: Every part ends with "in two sentences, why does X beat Y?" — forces construction, not recognition
- **Ground or flag**: Every load-bearing claim either carries an inline citation to a source the LLM *actually read* (deep-linked to section, not homepage), or a `[!UNVERIFIED]` callout naming what to check
- **Show-wrong-then-fix**: Introduce a concept, demonstrate the tempting-but-broken way, mock it in one sentence, show the fix. "The reader needs to *feel* why the fix matters."
- **Pre-store gate**: Before `lathe store`, the skill must explicitly state repo, versions, tags, sources, voice, and model — or give a "justified opt-out." A prompt-level lint rule.

### Voice partition system
Voices control **tone and register only** — never accuracy, research, citation, verification, or pedagogy. The invariants live in the skill and always win. Security: every voice spec gets a fixed, non-overridable `Preamble` prepended at Go code level (`internal/voice/voice.go:37-62`). A hostile custom voice file can't escape — the Preamble is code, not prompt.

Two built-ins: `plainspoken` (default — precise, no fabricated first-person, no invented credentials) and `companion` (warm, wry, first-person "friend at the keyboard"). Custom voices authored via `/lathe-voice`.

### GFM-alert callout preprocessor
Eight `> [!TYPE]` blockquote types (NOTE, TIP, WARNING, HEADS-UP, ASIDE, DESIGN-NOTE, PREDICT, RECALL, UNVERIFIED), each with distinct semantic meaning and CSS styling. `preprocessCallouts` in `internal/serve/renderer.go:162-184` uses regex to rewrite them into raw `<aside>` HTML blocks before goldmark renders. Styled callouts without a custom markdown extension.

### Handoff model
The Go binary never drives a model. Instead, web POST endpoints return `{"command": "/lathe-verify <slug>"}` via `writeHandoff` (`internal/serve/handoff.go:12-16`). The user pastes it into their coding agent. Skills do the work and call back into CLI commands (`lathe store`, `lathe verify-result`, `lathe extend-start`/`extend-commit`). Keeps the binary off metered headless API costs, keeps model work on the user's existing subscription.

### Atomic state + CSRF on loopback
`writeJSONFile` (`internal/store/metadata.go:175-205`) writes to a temp file in the same directory, then `os.Rename` atomically — no torn writes. Despite binding to `127.0.0.1` only, destructive POSTs check `Origin`/`Referer` against loopback hosts (`internal/serve/server.go:272-291`) to defend against LAN devices.

## Design decisions

**No model in Go**: The binary is pure state management. All LLM work runs in the user's interactive session. Keeps it simple, offline-capable, and avoids metered API complexity. Trade-off: less seamless UX — user must paste commands between UI and agent session.

**Opt-in verification**: Tutorials store as `unverified` by default. User explicitly runs `/lathe-verify`. No auto-running code. Trade-off: unverified tutorials sit in the library with no correctness guarantee.

**One part per invocation**: `/lathe` always produces exactly one `part-01.md`. Prevents quality degradation from writing all parts in one shot. Trade-off: more friction for series.

**Local-first, loopback-only**: Server at `127.0.0.1:4242`. No auth, no cloud, no sharing. Personal learning tool by design. Trade-off: no remote access or team features.

**"Vibecoded" but documented**: Author openly admits LLM-heavy development. The 111-line AGENTS.md with architecture notes is both documentation and a sign of vibecoded code that needed extensive explanation to be maintainable.

## Comparison notes

**vs. coding agents**: Lathe inverts the typical workflow — the LLM teaches, the human codes. Most coding agents optimize for the LLM doing the work; Lathe optimizes for the LLM *teaching*.

**vs. [[Coding Agents Continuity Not Memory]]**: Different solution to the continuity problem. Santi argues for repo-local, evidence-weighted records. Lathe's handoff model (CLI as continuity layer, stateless skill workers) is an alternative architecture.

**vs. [[The Agentic Product Standard]]**: The skill-CLI boundary is an instance of the coordinator-worker composition pattern. The pre-store gate is a prompt-level validation pattern.

**vs. [[Honey I Shrunk the Coding Agent]]**: Exemplifies the thesis that harness > model. The 416-line skill prompt matters more than which LLM runs it.

**vs. [[Printing Press]] and [[sx]]**: Similar cross-tool skill distribution approach, but Lathe is specifically for tutorial generation, not general-purpose skills.

**vs. [[Thought Refiner Skill]]**: Both are masterclasses in defining what a skill *won't* do. Lathe's voice guardrails (Preamble) show the same discipline at the code level that Thought Refiner shows at the prompt level.

Tags: #tool #project #agents #learning #prompt-engineering #tutorials #go

---
Fetched 2026-06-12 from https://github.com/devenjarvis/lathe
