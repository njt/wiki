---
url: https://github.com/DietrichGebert/ponytail
title: "Ponytail — Lazy Senior Dev Mode for AI Coding Agents"
author: Dietrich Gebert
date_fetched: 2026-06-21
date_published: 2024-11-15
topics:
  - developer-tools
---

# Ponytail — Full Source Analysis

## Overview

Ponytail is a multi-platform AI coding agent plugin that makes agents write drastically less code through a "lazy senior developer" persona. It is not an agent framework — it's a personality-modifying ruleset injected as system prompt context. The core insight: before writing ANY code, the agent climbs a "ladder" of progressively more effortful solutions, stopping at the first rung that works: (1) does this need to exist? (YAGNI), (2) stdlib does it? (3) native platform feature? (4) already-installed dependency? (5) can it be one line? (6) only then write minimum code.

The project ships as a Claude Code plugin, with adapters for 14+ coding agent hosts (Codex, Copilot CLI, Gemini CLI, OpenCode, pi, Antigravity, OpenClaw, Cursor, Windsurf, Cline, Aider, Kiro, Zed, CodeWhale). It provides both always-on session activation (ruleset injected every turn) and interactive slash commands (`/ponytail`, `/ponytail-review`, `/ponytail-audit`, `/ponytail-debt`, `/ponytail-gain`, `/ponytail-help`).

## Architecture

### Core Ruleset Layer

The heart of ponytail is **`AGENTS.md`** (28 lines) — a compact instruction file that agents without plugin support can read directly. The full behavioral specification lives in **`skills/ponytail/SKILL.md`** (~100 lines with YAML frontmatter), which includes the intensity table (lite/full/ultra), the ladder, safety boundaries, output format, and the "one check" rule. Five companion skills provide the interactive commands: `ponytail-review`, `ponytail-audit`, `ponytail-debt`, `ponytail-gain`, `ponytail-help`.

### Host Adapter Layer (hooks/)

Three Node.js lifecycle hooks (no external dependencies, pure Node stdlib):

- **`ponytail-activate.js`** — SessionStart hook. Writes a `.ponytail-active` flag file, emits the ruleset as hidden context, detects missing statusline config and nudges the agent to set it up. Handles off mode (skip entirely).
- **`ponytail-mode-tracker.js`** — UserPromptSubmit hook. Parses user input for `/ponytail` commands and deactivation phrases ("stop ponytail", "normal mode"), updates the flag file, emits mode-change context.
- **`ponytail-config.js`** — Shared config module. Resolution order: `PONYTAIL_DEFAULT_MODE` env var → `~/.config/ponytail/config.json` → `'full'`. Paths respect XDG, Windows APPDATA, and `CLAUDE_CONFIG_DIR`.
- **`ponytail-instructions.js`** — Reads the SKILL.md file, strips YAML frontmatter, and filters mode-specific rows (intensity table, worked examples) to the active level. Falls back to a baked-in compact version if the file is missing.
- **`ponytail-runtime.js`** — Shared runtime: writes flag files, detects Codex vs Copilot vs Claude Code environments, emits host-appropriate hook output (JSON for Codex/Copilot, raw text for Claude Code).

### Multi-Host Adapter Files

Each host gets a thin adapter pointing at the shared files:

| Host | Adapter | Mechanism |
|------|---------|-----------|
| Claude Code | `.claude-plugin/plugin.json` + `hooks/claude-codex-hooks.json` | SessionStart + UserPromptSubmit hooks |
| Codex | `.codex-plugin/plugin.json` | Same hooks, different plugin manifest |
| OpenCode | `.opencode/plugins/ponytail.mjs` | Server plugin via `experimental.chat.system.transform` |
| pi | `pi-extension/index.js` | `before_agent_start` event injects rules into system prompt |
| Gemini CLI | `gemini-extension.json` | Points `contextFileName` at `AGENTS.md` |
| Copilot CLI | `.github/plugin/` | Plugin marketplace, instruction fallback via `AGENTS.md` |
| Cursor/Windsurf/Cline/Kiro | `.cursor/rules/`, `.windsurf/rules/`, `.clinerules/`, `.kiro/steering/` | Static rule files copied into project |
| MCP | `ponytail-mcp/index.js` | MCP server with `ponytail_instructions` tool + `ponytail` prompt |

### Pi Extension (pi-extension/index.js)

The most sophisticated adapter. Registers 6 slash commands (`/ponytail`, `/ponytail-review`, `/ponytail-audit`, `/ponytail-gain`, `/ponytail-debt`, `/ponytail-help`), persists mode across turns via session entries, detects deactivation commands on input, and injects the ruleset before every agent start via `before_agent_start` event. Reuses the shared `ponytail-config.js` and `ponytail-instructions.js` modules.

### Command Layer (commands/*.toml, skills/*/SKILL.md)

Six slash commands, each defined in two formats:
- `commands/*.toml` — Claude Code/Codex/Gemini command format
- `skills/*/SKILL.md` — Skill format with YAML frontmatter (name, description, argument-hint, license)
- `.opencode/command/*.md` — OpenCode command format

Commands delegate to skills for execution; the ponytail mode command handles intensity switching directly.

### Statusline (hooks/ponytail-statusline.sh, hooks/ponytail-statusline.ps1)

Reads the `.ponytail-active` flag file and prints a colored statusline badge: `[PONYTAIL]` (full mode) or `[PONYTAIL:ULTRA]` (other modes). Bash for macOS/Linux, PowerShell for Windows.

### MCP Server (ponytail-mcp/)

Alternative access path for hosts whose only injection point is the MCP prompt menu. Registers a `ponytail` prompt and a `ponytail_instructions` tool (readOnly, openWorldHint: false). Uses Zod for schema validation. Separates instruction selection (`instructions.js`) from transport (`index.js`) for testability.

### Benchmark Suite (benchmarks/)

Three-tier evaluation:

1. **Single-shot** (`promptfooconfig.yaml`): One prompt, one completion, three models (Haiku/Sonnet/Opus), three arms (no-skill/caveman/ponytail). Showed 80-94% less code but had a conversational-baseline problem.
2. **Behavior gate** (`behavior.js`, `behavior.yaml`): Tests that ponytail actually *produces* its claimed behaviors (hardware calibration knob, full requested explanations, one runnable check). Unit-testable without API keys.
3. **Correctness gate** (`correctness.js`): Actually *executes* generated code against test cases. Proves "less code is not broken code." Spawns Node/Python with harnessed assertions.
4. **Agentic** (`benchmarks/agentic/`): Real headless Claude Code sessions editing `tiangolo/full-stack-fastapi-template`. 12 feature tasks + 6 safety tasks, n=4, 4 arms (baseline/ponytail/caveman/yagni-oneliner). Measured by `git diff` LOC, tokens, cost, time, and safety. This is the honest benchmark — it answers Colin Eberhardt's critique (#126) that the single-shot baseline was inflated by chatty model output.

### Documentation & Examples

- `docs/platform-native.md` — Reference table of platform-native alternatives for common dependencies (HTML elements, CSS capabilities, JS browser APIs, Node.js stdlib, Python stdlib, database features). The "skip the wrapper" pattern.
- `docs/agent-portability.md` — Which hosts use which adapter files, and the adapter rule (keep adapters thin, point at shared files).
- `examples/` — 11 worked before/after comparisons (debounce, email validation, CSV sum, etc.) showing verbatim model output with and without ponytail.

## Key Techniques

### The Ladder Heuristic (Not a Prompt, a Decision Tree)

The core innovation: a specific, ordered heuristic that the agent applies before every code decision. Unlike vague instructions ("be concise" or "write less"), the ladder is a concrete algorithm: check each rung in order, stop at the first one that works. This is why it beats "YAGNI + one-liners" as a prompt — the ladder has a consistent decision procedure; the prompt sometimes lands and sometimes doesn't.

### Intensity Levels with Filtering

The three intensity levels (lite/full/ultra) are implemented by filtering rows from a single SKILL.md file. Mode-specific content (the intensity table, worked examples) is keyed by mode name; `filterSkillBodyForMode()` strips rows whose label doesn't match the active mode. Non-mode-specific rules (the ladder, safety boundaries) are always present. This means a single source of truth with programmatic filtering, not three separate rule files.

### Flag File as Persistent State

Mode persistence across sessions is handled by a simple flag file (`.ponytail-active` in the Claude config directory). The statusline reads it, the activation hook writes it, and the mode tracker updates it on command. No database, no config parsing — just `fs.writeFileSync`/`fs.readFileSync`.

### Host Abstraction via `writeHookOutput()`

Different hosts expect different output formats from lifecycle hooks: Claude Code wants raw text, Codex wants `{systemMessage, hookSpecificOutput}`, Copilot wants `{additionalContext}`. `ponytail-runtime.js` detects the host via environment variables (`CLAUDE_PLUGIN_ROOT`, `PLUGIN_DATA`, `COPILOT_PLUGIN_DATA`) and emits the correct format. This is the only host-specific code; the instruction builder is shared.

### Safety Boundary as Explicit Exclusion

The ruleset explicitly enumerates what is NEVER simplified: input validation at trust boundaries, error handling preventing data loss, security, accessibility, hardware calibration. This is not just a disclaimer — it's a specific list that the benchmark verifies with adversarial test cases (path traversal, SQL injection, malformed CSV). The agentic benchmark proves this works: ponytail is 100% safe (20/20 safety runs) while the bare "one-liner" prompt drops a guard (95%).

### One Runnable Check Rule

"Lazy code without its check is unfinished." Non-trivial logic (a branch, a loop, a parser) must leave ONE runnable check behind — an assert, a demo/self-check, or one small test file. No frameworks, no fixtures. This is verified by the behavior gate's `onecheck` probe.

### Comment Convention for Deferred Complexity

`// ponytail: global lock, per-account locks if throughput matters` — deliberate simplifications are marked with a structured comment naming the ceiling and the upgrade path. The `/ponytail-debt` skill greps these into a ledger so "later" doesn't silently become "never."

### Over-Engineering Review (ponytail-review)

A dedicated review skill that hunts ONLY complexity, not correctness. Five tags: `delete:`, `stdlib:`, `native:`, `yagni:`, `shrink:`. One line per finding. Ends with `net: -N lines possible.` or `Lean already. Ship.` This is a novel review dimension — most code review looks for bugs; this one looks for things to remove.

### Measured Honesty

The benchmark results page (`2026-06-18-agentic.md`) is unusually honest:
- Publishes the contamination bug they found (SessionStart hook firing on baseline arm)
- Shows where ponytail does NOT win (irreducible backend CRUD, near-identical across all arms)
- Admits limitations (one model, n=4, safety is a floor not a proof)
- The gain skill refuses to compute per-repo savings ("the unbuilt version was never written, so there is no real baseline")

## Design Decisions

**Decision: Ruleset injection, not a framework or agent.** Ponytail doesn't wrap API calls or intercept tool use. It's pure system prompt augmentation. This makes it host-portable — any agent that reads system instructions can use it. The trade-off: it only works if the model follows instructions well.

**Decision: Thin adapters, thick shared core.** Every host adapter (Claude Code hooks, pi extension, OpenCode plugin, Gemini extension, MCP server) is a thin wrapper around the same `ponytail-instructions.js` builder and `skills/` files. Adding a new host means writing ~50 lines of glue, not duplicating the ruleset.

**Decision: Filter one file, don't maintain three.** The lite/full/ultra variants are derived from a single SKILL.md by filtering mode-tagged rows. This avoids drift between variants at the cost of more complex filtering logic.

**Decision: Flag file over config key.** Persisting mode in a flag file (rather than reading from settings.json) means the statusline can read it without parsing JSON, and multiple host adapters can share the same state file without knowing about each other's config formats.

**Decision: Benchmark the real thing.** The agentic benchmark uses actual Claude Code headless sessions on a real open-source repo, not synthetic single-shot prompts. This was expensive to build but produces defensible numbers. The contamination bug they found (and published) is evidence the methodology works — a simpler benchmark would have silently inflated the results.

**Decision: Node.js for hooks, no dependencies.** All lifecycle hooks are plain Node.js with zero npm dependencies. This avoids install complexity for the most critical path (session start). The MCP server is the only component with dependencies (`@modelcontextprotocol/sdk`, `zod`).

**Decision: AGENTS.md as universal fallback.** For hosts that can't run hooks or load skills, copying `AGENTS.md` into the project gives always-on ponytail behavior. This is the lowest-common-denominator adapter — no mode switching, no commands, but the ruleset works.

## Comparison Notes

- **vs. Caveman**: Caveman makes the agent's prose terse. Ponytail makes the agent's code minimal. The benchmark shows caveman writes 20% less code but uses MORE tokens (+7%) because terse output ≠ fewer thinking tokens. They're compatible — "pair with Caveman for terse prose" (SKILL.md).
- **vs. "YAGNI + one-liners" prompt**: Colin Eberhardt correctly argued a short prompt might do the same job. The agentic benchmark proved it doesn't — the prompt is inconsistent (sometimes beats baseline, sometimes at/below it), and it's the only arm that dropped a safety guard. The ladder heuristic is more reliable than generic "write less" instructions.
- **vs. .cursorrules / CLAUDE.md**: Those are static instruction files that apply to all coding. Ponytail is layered on top: it governs *how* code is written, not *what* code to write. It can be toggled, has intensity levels, and provides targeted review/audit/debt commands.
- **vs. Code review tools**: Traditional review tools find bugs and style issues. `/ponytail-review` finds things to delete — it's a complementary dimension. A normal review + ponytail-review gives both correctness and simplicity coverage.
- **vs. Linting rules (eslint complexity, etc.)**: Linters catch complexity after it's written. Ponytail prevents it from being written. They're complementary, not competitors.

## Tags

#tool #coding-agent #plugin #claude-code #yagni #simplicity #agent-skill #benchmark
