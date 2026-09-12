# Ponytail

A multi-platform AI coding agent plugin that forces the simplest solution that actually works. It's a "lazy senior dev" persona injected as system prompt context — the agent climbs a decision ladder before writing any code: YAGNI → stdlib → native platform → installed dep → one line → minimum code. Ships as a Claude Code plugin with adapters for 14+ coding agent hosts. Thoroughly benchmarked: on real headless Claude Code sessions editing a real FastAPI+React repo, it cuts **54% of code** (mean across 12 feature tasks) without dropping a single safety guard.

---

## Architecture

Ponytail is **ruleset injection**, not a framework. It doesn't wrap API calls or intercept tools — it augments the system prompt with a specific decision heuristic.

### Core layers

- **`AGENTS.md`** — 28-line compact ruleset for hosts without plugin support. The universal fallback.
- **`skills/ponytail/SKILL.md`** — ~100-line full behavioral spec with YAML frontmatter. Includes the ladder, intensity table, safety boundaries, output format, and the "one check" rule.
- **Companion skills**: `ponytail-review` (over-engineering review), `ponytail-audit` (repo-wide audit), `ponytail-debt` (harvest deferred shortcuts), `ponytail-gain` (benchmark scoreboard), `ponytail-help` (reference card).

### Host adapters (`hooks/`)

Three Node.js lifecycle hooks with zero npm dependencies:

| File | Event | What it does |
|------|-------|-------------|
| `ponytail-activate.js` | SessionStart | Writes flag file, emits ruleset, detects missing statusline |
| `ponytail-mode-tracker.js` | UserPromptSubmit | Parses `/ponytail` commands, updates flag file |
| `ponytail-config.js` | (shared) | Resolves default mode: env var → config file → `'full'` |

Mode persistence uses a simple flag file (`.ponytail-active`) rather than a config key — the statusline reads it without JSON parsing, and multi-host adapters share the same state.

### Mode filtering

The three intensity levels (lite/full/ultra) come from a single SKILL.md file. `filterSkillBodyForMode()` in `ponytail-instructions.js` strips rows whose mode label doesn't match the active level. Non-mode-specific rules (the ladder, safety boundaries) are always present. One source of truth, programmatic filtering — no three separate rule files.

### Host abstraction

`ponytail-runtime.js` detects the host via environment variables (`CLAUDE_PLUGIN_ROOT`, `PLUGIN_DATA`, `COPILOT_PLUGIN_DATA`) and emits host-appropriate output: raw text for Claude Code, `{systemMessage, hookSpecificOutput}` for Codex, `{additionalContext}` for Copilot.

### 14+ host adapters

Thin wrappers pointing at the shared core:

- **Claude Code**: SessionStart + UserPromptSubmit hooks, plugin marketplace
- **Codex**: Same hooks, different plugin manifest
- **OpenCode**: Server plugin via `experimental.chat.system.transform`
- **pi**: `before_agent_start` event injects rules, 6 slash commands
- **Gemini CLI / Antigravity**: Extension manifest pointing `contextFileName` at `AGENTS.md`
- **Copilot CLI**: Plugin marketplace + instruction fallback
- **MCP**: `ponytail_instructions` tool + `ponytail` prompt via stdio MCP server
- **Cursor/Windsurf/Cline/Kiro/Zed/CodeWhale**: Static rule files copied into project

## Key Techniques

### The ladder — a decision tree, not a vibe

The core innovation is a concrete algorithm, not a vague instruction. Before any code: (1) does this need to exist? (2) stdlib? (3) native platform? (4) installed dep? (5) one line? (6) minimum code. Two rungs work → take the higher one and move on. This is why it beats "YAGNI + one-liners" as a plain prompt — the ladder is a consistent procedure; the prompt sometimes lands and sometimes doesn't.

### Safety as explicit exclusion

The ruleset enumerates what is NEVER simplified: input validation at trust boundaries, error handling preventing data loss, security, accessibility, hardware calibration. The benchmark verifies this with adversarial test cases — ponytail kept the path-traversal check (the ~3 lines that matter) while the bare "one-liner" prompt dropped it once in four runs.

### `ponytail:` comment convention

Deliberate simplifications are marked: `// ponytail: global lock, per-account locks if throughput matters`. The comment names the ceiling and the upgrade path. `/ponytail-debt` greps these into a ledger so deferred shortcuts don't silently rot.

### One runnable check

Non-trivial logic leaves ONE check behind — an assert, a `__main__` self-check, or one small test file. No frameworks, no fixtures. Verified by the behavior gate.

### Over-engineering review

`/ponytail-review` hunts ONLY complexity, not bugs. Five tags: `delete:`, `stdlib:`, `native:`, `yagni:`, `shrink:`. One line per finding. A complementary review dimension — pair with normal correctness review.

### Benchmarking rigor

The agentic benchmark (2026-06-18) is unusually honest:
- Real headless Claude Code sessions on `tiangolo/full-stack-fastapi-template`
- 4 arms (baseline, ponytail, caveman, yagni-oneliner), n=4, measured by `git diff`
- Published their own contamination bug (SessionStart hook firing on baseline arm)
- Shows where ponytail does NOT win (irreducible backend CRUD is near-identical)
- Gain skill refuses to compute per-repo savings ("the unbuilt version was never written")

## Design Decisions

**Ruleset injection, not a framework.** Pure system prompt augmentation — host-portable, works with any instruction-following model. Trade-off: no programmatic enforcement, model-dependent.

**Thin adapters, thick shared core.** Every host adapter is ~50 lines of glue around the same `ponytail-instructions.js` builder and `skills/` files.

**Filter one file, don't maintain three.** Lite/full/ultra derived from a single SKILL.md by filtering mode-tagged rows.

**Flag file over config key.** Persistence via `.ponytail-active` rather than settings.json — statusline reads it without JSON parsing, multi-host adapters share state.

**Node.js for hooks, zero deps.** Lifecycle hooks use pure Node stdlib — no install complexity on the critical session-start path.

**AGENTS.md as universal fallback.** For hosts that can't run hooks, copying one file gives always-on behavior.

## Comparison Notes

- **vs. Caveman**: Caveman makes prose terse; ponytail makes code minimal. Compatible — "pair with Caveman for terse prose." Caveman writes 20% less code but uses MORE tokens (+7%) because terse output ≠ fewer thinking tokens.
- **vs. "YAGNI + one-liners" prompt**: Inconsistent — sometimes beats baseline, sometimes at/below it. Only arm that dropped a safety guard (95% safe vs ponytail's 100%). The ladder heuristic is more reliable than generic "write less."
- **vs. CLAUDE.md / .cursorrules**: Those govern *what* to do; ponytail governs *how* to do it. Layered, togglable, with intensity levels.
- **vs. Linting**: Linters catch complexity after it's written; ponytail prevents it from being written. Complementary.
- **vs. [[Coding Agents and Complexity Budgets]]**: Related concept — complexity budgets are an explicit accounting scheme; ponytail is the implementation discipline that stays within budget.
- **vs. [[The Lindy Effect]]**: Ponytail's decision ladder (stdlib → native → dep → one line → code) is Lindy operationalized at the code level — prefer what's already there over what you'd have to add. Lindy governs technology selection; ponytail governs implementation selection.

---

*Source: [[summary/ponytail]]*
*Repository: https://github.com/DietrichGebert/ponytail (MIT, v4.7.0)*
*Last updated: 2026-06-21*
