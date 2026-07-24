# DrSkill

`brew doctor` for your AI coding agent's skill loadout. DrSkill scans every coding agent on your machine or in your repo, resolves each one's effective skill set and MCP server configurations, and checks the whole set for problems: shadowed skills, description collisions, prompt injection surfaces, broken symlinks, token budget violations, misconfigured MCP servers, and tool poisoning. It reads files only — never installs, edits, or deletes a skill. Every finding comes with a fix command or an ack command. Zero LLM calls and zero MCP connections unless you opt in.

---

## Architecture

DrSkill is a Python CLI (~5,000 lines of core logic in `src/drskill/`) built around a **three-tier scan model** and a **check registry pattern**.

### Three-Tier Scan

1. **Static (always runs):** Reads config files and skill directories. No network, no LLM. Covers 23 of the ~29 checks.
2. **MCP connect (`--mcp-connect`):** Connects to configured MCP servers, runs the MCP handshake, enumerates tools, and writes committed snapshots to `.drskill/cache/mcp-tools/`. Opt-in. Enables tool collision, tool poisoning, and tool-description-change detection.
3. **Deep (`--deep`):** Sends skill pairs flagged as ambiguous by the description-overlap heuristic to an LLM (via dspy/LiteLLM) for judgment. Budgeted (default 25 calls per run). Verdicts are cached as JSON in `.drskill/cache/` — a directory meant to be committed, so one person pays the API cost and the whole team benefits.

### Check Registry Pattern

Each check is registered via a `@check("check-id")` decorator in `checks/__init__.py`. Checks are pure functions `(World, Config) -> list[Finding]`. The `run_all()` function imports all check modules (triggering registration), calls every one, and merges findings with identical fingerprints into single findings spanning multiple harnesses.

### Core Data Flow

```
HarnessDef[] (from data/harnesses.toml, 70+ agents)
  → discover() → RawInstance[] (every SKILL.md from every search path)
  → build_world() → World (deduplicated Contributors with shadow markers)
  → run_all() → Finding[] (all registered checks)
  → filter_findings() → active + acked (against ledger)
  → apply_verdicts() → reshaped findings (deep judgments applied)
  → report.render() → terminal output
```

### Harness Definition Table

`data/harnesses.toml` catalogs 70+ coding agents (Claude Code, Codex, Pi, Cursor, Gemini CLI, Cline, Copilot, and ~65 more vendored from `vercel-labs/skills`). Each entry specifies skill search paths, MCP config paths, search order, recursion behavior, and verification status (`paths_verified`, `precedence_verified` — confirmed against each harness's own docs or source).

## Key Techniques

### Content-Addressable Ack Fingerprinting

Every finding carries a SHA-256 fingerprint of `check_id | sorted(contributor content hashes) | extra_key`. An ack silences a finding only while that fingerprint matches. If a skill's content changes, its hash changes → fingerprint changes → finding resurfaces. This makes acks mean "I've reviewed this exact situation" rather than "never check this pair again."

### Committed Verdict Cache

LLM judgments from `--deep` are stored as JSON in `.drskill/cache/` and committed to version control. Every scan reads this cache, with or without `--deep`. When every pair in an overlap cluster is judged distinct, the warning downgrades to a visible note. A skill with an active injection finding never earns this downgrade — "a skill suspected of prompt injection does not get to talk its way out of an overlap warning."

### MinHash Near-Duplicate Detection

Hand-rolled MinHash using `zlib.crc32` (not Python's salted `hash()`) with 5-word shingles and 128 hash functions. Jaccard similarity estimate via signature comparison. Default threshold: 0.85.

### Injection Checks: Flag Surfaces, Never Verify Intent

Seven static pattern-matching checks cover: invisible/bidirectional Unicode, instruction-override phrasing, mandatory bundled scripts, network egress in scripts, credential path references, remote-fetch directives, and long encoded blobs. Every finding quotes exact lines and ends with "(static flag: drskill shows the evidence; it cannot verify intent)." The ack ledger is the escape hatch.

### Scope-Aware Ack Routing

Acks are automatically routed to the right ledger: machine-level-only findings → `~/.drskill.toml` (honored in every project); anything touching a project skill → project's `drskill.toml`. MCP config findings use file location (not scope) to route.

### Secret Detection Without Storing Secrets

`mcp.py:looks_secret()` detects credential-shaped values in MCP env blocks via prefix matching (sk-, ghp_, AKIA, etc.), entropy heuristics, and public-material exclusion. The detected *value* is immediately discarded — only the variable name appears in models and fingerprints.

### Token Budget Awareness

`budget-catalog-tokens` warns when a harness's total catalog tokens (skill names + descriptions + MCP tool definitions, counted with tiktoken's `o200k_base`) exceed a configurable ceiling. `budget-body-tokens` warns per-skill. These are the startup context cost — what the agent loads before the user types a word.

## Design Decisions

### Optimized For: Safety and Non-Destructiveness

DrSkill never edits, installs, or deletes a skill. It never writes API keys. MCP connections only enumerate tools (never call them). Deep mode sends only names and descriptions to the LLM. The env file (`~/.drskill/env`) is read-only to drskill and never read from inside a project (because a scanned repo is untrusted content).

### Optimized For: CI and Team Workflows

Exit codes are carefully tuned: 0 for clean/acked, 1 for errors, 2 for warnings under `--ci`. Without `--ci`, warnings alone exit 0 — so a local scan doesn't fail your shell. `scan` never prompts, so scripts and agents can call it safely.

### Optimized For: Incremental Adoption

Every finding ends in a command. `drskill ack` is the primary workflow — acknowledge what you've reviewed, and the tool stays quiet until the content changes. `drskill review` provides an interactive walk-through with single-key actions. The tool meets you where you are.

### Trade-Off: Heuristics Over Precision

The description and instruction checks are heuristic. Thresholds are tuned against real public skill corpora (`scripts/corpus.py`) to stay quiet on well-written skills, but they will miss paraphrased conflicts and flag some judgment calls. The escape hatch is the ack ledger, not more sophisticated detection.

### Trade-Off: The "?" Uncertainty System

Each harness has `paths_verified` and `precedence_verified` booleans. Findings only carry `?` on the specific harness facet they depend on (shadowing → precedence, broken symlinks → paths). This is surgically precise but means a harness with both facets unverified might show `?` on some findings and not others, which could confuse.

### Trade-Off: The Committed Cache Trust Model

Both the verdict cache and the ack ledger are unsigned files in the repo. Anyone who can commit can silence a warning through either one. The README acknowledges this: "Review a change to `.drskill/cache/` the way you review a change to `drskill.toml`."

## Comparison Notes

DrSkill sits at the intersection of several concerns the wiki tracks separately:

- **vs. general linters** like ESLint or ruff: DrSkill understands agent-specific semantics — skill shadowing resolution, MCP tool collision across servers, prompt injection surfaces in natural-language SKILL.md files. It's a domain-specific linter for the agent loadout.

- **vs. [[Malicious Agent Skills in the Wild]]** (academic measurement): DrSkill is operational — you run it in your repo today. The academic paper measured the ecosystem; DrSkill protects your specific loadout.

- **vs. [[Bounding the Blast Radius — Prompt Injection Defenses]]** (defense taxonomy): DrSkill is detection, not defense. It flags surfaces; it doesn't prevent exploitation. It complements runtime defenses by catching problems before deployment.

- **vs. [[Guardrails and Feedback Loops]]** (hub page): DrSkill exemplifies the hub's thesis that "linters beat prompts." It's deterministic enforcement over deterministic scan, not LLM-negotiated safety.

- **vs. [[Steering Claude Code]]** (instruction taxonomy): That page covers what skills *are* and how they load; DrSkill covers how to *audit* them — the quality-and-safety layer on top of the instruction-delivery taxonomy.

- **vs. [[Agent Skills Library (dzhng)]]** and [[Audit Skills for AI Coding Agents (metacircu1ar)]]: Those are curated skill libraries; DrSkill is the tool you run *on* libraries like those to catch problems before agents execute them.

- **vs. [[ANSI Escape Sequence Injection in MCP Servers]]**: DrSkill's `mcp-tool-poisoning` check covers the same class of attack (hidden instructions in tool descriptions) but scans tool name/description/schema doc strings for a broader set of injection surfaces (hidden Unicode, credential paths, encoded blobs, remote-fetch, tool steering), not just ANSI escapes.

- **vs. [[Security and Sandboxing]]** (hub): DrSkill is part of the *pre-deployment* security layer — catching misconfigurations and injection surfaces before agents load them. Sandboxing is the *runtime* layer.

---

*Sources: [[raw/drskill]]*
*Tags: #tool #project #agents #security #devtools*
*Last updated: 2026-07-25*
