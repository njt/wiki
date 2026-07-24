---
url: https://github.com/dbreunig/drskill
title: drskill
author: dbreunig (David Breunig)
date_fetched: 2026-07-25
date_published: 2026-07-25
version: 0.6.2
---

# drskill — Full Repository Analysis

## Overview

`drskill` is `brew doctor` for your AI coding agent's skill loadout. It is a Python CLI that scans all coding agents detected on a machine or in a repo, resolves each agent's effective skill set and MCP server configurations, and checks the whole set for problems: shadowing, duplicates, description collisions, injection surfaces, broken links, token budget violations, MCP misconfigurations, and tool poisoning. Every problem it reports ends in a fix command or an ack command. It reads files only — it never installs, edits, or deletes a skill. It makes zero LLM calls unless you opt in, and it never connects to MCP servers unless you opt in.

## Repository Structure

```
src/drskill/
  __init__.py        — empty
  cli.py             — Typer-based CLI (~715 lines): scan, ack, show, review, list, audit, cache, init
  models.py          — Pydantic data models: RawInstance, Contributor, Finding, Deployment, etc.
  pipeline.py        — Orchestration: run_scan() ties discovery → resolution → checks → deep → verdicts
  discovery.py       — Walks harness search paths to find SKILL.md files and broken symlinks
  resolution.py      — build_world(): deduplicates instances, parses frontmatter, marks shadows
  harnesses.py       — HarnessDef model + detection: 70+ coding agents defined in data/harnesses.toml
  ledger.py          — drskill.toml parsing, ack append (text-preserving), config merging
  deep.py            — Deep tier logic: pair keys, verdict cache CRUD, flagged_pairs, apply_verdicts
  deep_llm.py        — dspy/LiteLLM integration: judge and rewriter programs, lazy-imported
  report.py          — Rich-based terminal rendering of findings and harness tables
  tokens.py          — Token counting: tiktoken o200k_base with fallback to len//4
  mcp.py             — Static MCP config discovery: 5 config formats, secret detection
  mcp_connect.py     — Live MCP handshake: spawns servers, enumerates tools, snapshots
  interactive.py     — Terminal keypress reading for `drskill review`
  suites.py          — Suite assignment: matches skills to plugin caches on disk
  state.py           — Seen-state tracking: which findings the user has already seen
  text.py            — Text utilities
  checks/
    __init__.py      — Check registry, fingerprint(), make_finding(), run_all()
    shadowing.py     — name-shadow, double-load
    duplicates.py    — exact-duplicate, near-duplicate (MinHash + Jaccard)
    spec.py          — SKILL.md spec violations (name mismatch, missing description, etc.)
    filesystem.py    — broken-symlink
    lockfile.py      — lockfile-drift
    budget.py        — budget-catalog-tokens, budget-body-tokens
    heuristics.py    — description-overlap, missing-activation, generic-description, opposing-imperatives
    injection.py     — 7 injection checks (unicode, encoded-blob, override, remote-fetch, egress, credential-read, mandatory-script)
    mcp.py           — Static MCP checks (config, shadowing, divergence, secrets, unpinned, insecure, dead-server)
    mcp_tools.py     — mcp-tool-collision, mcp-tools-unreviewed
    mcp_injection.py — mcp-tool-poisoning (scans tool descriptions from committed snapshots)
  traces/
    __init__.py
    model.py         — Invocation data model
    common.py        — Shared trace utilities
    pipeline.py      — Orchestration for `drskill audit`: discover, extract, filter
    cache.py         — Trace extraction cache at ~/.drskill/cache/audit/
    claude_code.py   — Claude Code trace parser
    codex.py         — Codex trace parser
    pi.py            — Pi trace parser
    copilot.py       — Copilot trace parser
    report.py        — Audit report rendering
  data/
    harnesses.toml   — Definition table for 70+ coding agents

tests/               — ~30 test files covering checks, CLI, traces, discovery, MCP, models, etc.
docs/superpowers/    — Design specs and plans from development
```

Total project size: approximately 5,000–6,000 lines of Python (core src), plus ~30 test files.

## Architecture

### Three-Tier Scan

1. **Static (always runs):** Reads config files and skill directories. No network, no LLM.
2. **MCP connect (`--mcp-connect`):** Connects to configured MCP servers, runs the handshake, enumerates tools, writes snapshots. Opt-in.
3. **Deep (`--deep`):** Sends skill pairs flagged as overlapping to an LLM for judgment. Opt-in, budgeted (default 25 calls).

### Core Data Flow

```
HarnessDef[] → discover() → [RawInstance]
                              ↓
  harnesses.toml          build_world() → World
                              ↓
                         run_all() → [Finding]
                              ↓
                    ledger.filter_findings() → active, acked
                              ↓
                    deep.apply_verdicts() → reshaped findings
                              ↓
                          report.render()
```

### Check Registry Pattern

Checks are registered via a `@check("check-id")` decorator. Each check is a pure function `(World, Config) -> list[Finding]`. The `run_all()` function imports all check modules (triggering registration), then calls every registered check and merges findings with identical fingerprints.

### Fingerprinting System

Every finding carries a `fingerprint` — a SHA-256 hash of `check_id | sorted(contributor content hashes) | extra_key`. An ack silences a finding only while that fingerprint still matches. If a skill's content changes, its hash changes, the fingerprint no longer matches, and the finding resurfaces. This makes acks mean "I've reviewed this exact situation" rather than "never check this pair again."

### Verdict Cache

Deep judgments are stored as JSON files in `.drskill/cache/` under the project root (or `~/.drskill/cache/` in global mode). This directory is *committed to version control*. Every scan reads it, so one person runs the LLM judgments and every teammate and CI run gets the verdicts for free. A verdict lasts until either skill's description changes.

### Harness Definition Table

The file `data/harnesses.toml` defines 70+ coding agents (Claude Code, Codex, Pi, Cursor, Gemini CLI, Cline, Copilot, and ~65 more vendored from `vercel-labs/skills`). Each entry specifies:
- `id`, `display_name`
- `project_paths`, `global_paths` — where skills live
- `search_order` — project-first, global-first, or "none" (Codex keeps both copies visible)
- `recursive` — whether the harness walks subdirectories
- `paths_verified`, `precedence_verified` — which behavior claims were confirmed against docs/source
- `mcp_*` — MCP config file locations and format

A finding's harness list carries `?` suffixes for unverified facets that the finding actually depends on. Shadowing depends on precedence; broken symlinks depends only on paths. The uncertainty annotation is precise, not blanket.

## Key Techniques

### MinHash for Near-Duplicate Detection

Hand-rolled MinHash using `zlib.crc32` (not Python's salted `hash()`). Word shingles of 5 words, 128 hash functions, Jaccard similarity estimate by comparing signature arrays. Default threshold 0.85.

### Secret Detection Without Storing Secrets

`mcp.py:looks_secret()` detects credential-shaped values in MCP env blocks using prefix matching (sk-, ghp_, AKIA, etc.), entropy heuristics, and public-material exclusion. The detected value is immediately discarded — only the variable *name* appears in models and fingerprints.

### Scope-Aware Ack Routing

Acks are routed based on contributor scope. Machine-level skills (all contributors have `scope="user"`) → `~/.drskill.toml`. Any project-level contributor → project `drskill.toml`. Every project scan honors acks from both ledgers, so you decide once per machine. MCP configs use file location to route: home-side files → machine ledger.

### Self-Calibrating Lockfile Verification

Upstream `npx skills` computes hashes differently. If none of the hashes in a lockfile match what drskill computes, it won't accuse every skill of drift — instead it prints one warning saying hashes could not be verified. Per-skill drift warnings only appear once drskill confirms its algorithm agrees with the lockfile producer by matching at least one hash.

### Injection Checks: Flag Surfaces, Don't Verify Intent

All 7 injection checks are static pattern matchers. Each one quotes exact lines from the skill file. Every finding ends with "(static flag: drskill shows the evidence; it cannot verify intent)." The ack ledger is the user's escape hatch. No LLM call is involved.

Specific patterns:
- **injection-unicode:** Detects bidirectional control characters and zero-width characters (U+200B, U+202A-U+202E, U+2066-U+2069), excluding ZWJ/ZWNJ (used in emoji and writing systems)
- **injection-override:** 12 regex patterns for instruction-override phrasing like "ignore all previous instructions" or "do not tell the user"
- **injection-mandatory-script:** Skill text + bundled file name on the same line with mandatory framing ("must first run scripts/x")
- **injection-egress:** 17 patterns for network calls in script files (not prose): curl, requests.post, fetch(), axios, etc.
- **injection-credential-read:** References to credential paths (~/.ssh, ~/.aws, .pem) in scripts; .env reads are only a warning
- **injection-remote-fetch:** Detects `curl | sh` patterns and "download and follow" directives in skill prose
- **injection-encoded-blob:** Base64 runs ≥120 chars or hex runs ≥129 chars (URLs are stripped first)

### dspy/LiteLLM Integration for Deep Mode

The deep tier uses dspy (Stanford's DSPy framework) with LiteLLM for provider-agnostic model access. Two dspy programs:
- **ConflictJudge:** Classifies skill pairs as distinct, description_collision, or scope_overlap
- **DescriptionRewrite:** For collision pairs, rewrites one description to add a distinguishing "use when" condition

The model is configured in the ledger and defaults to `anthropic/claude-haiku-4-5`. Any LiteLLM-supported provider works.

### Trace Audit: Cross-Harness Usage Analysis

`drskill audit` reads local session traces from Claude Code, Codex, Pi, and Copilot to report which skills and MCP tools actually got invoked. It caches extracted data at `~/.drskill/cache/audit/` and provides drill-down showing the user query that triggered each invocation, with exact trace file and line numbers.

## Design Decisions

### Optimized For: Safety and Non-Destructiveness

- Never edits, installs, or deletes skills
- Never writes API keys
- MCP connections only enumerate tools, never call them
- Secret values detected but never stored
- Deep mode sends only names and descriptions to the LLM, never full skill bodies

### Optimized For: CI Integration and Team Workflows

- Exit codes: 0 (clean/acked), 1 (errors), 2 (warnings under --ci)
- Warnings exit 0 locally but fail CI with the `--ci` flag
- `scan` never prompts — safe for scripts and agents
- Committed cache means one person pays the API cost

### Trade-Off: Heuristics Over Precision

The description and instruction checks are heuristic. The thresholds are tuned against real public skill corpora but will miss paraphrased conflicts and flag some judgment calls. The escape hatch is the ack ledger, not more sophisticated detection.

### Trade-Off: Approximate Token Counting

Uses tiktoken's `o200k_base` encoding — a reasonable estimate but won't match every harness's actual tokenizer.

### Interesting Weakness: The "?" Uncertainty System

Each harness has `paths_verified` and `precedence_verified` boolean facets. A finding only inherits the uncertainty it actually depends on. Shadowing depends on precedence → only precedence-unverified harnesses get `?`. Broken symlinks depend only on paths → only paths-unverified harnesses get `?`. This is surgical but vulnerable to harnesses where both facets differ from the default — the `?` tells you "something might be wrong" but not what.

### Interesting Weakness: The Committed Cache Trust Model

The verdict cache and ack ledger share the same trust model: both are unsigned, so anyone who can commit to the repo can silence a warning through either one. This is acknowledged in the README.

## Comparison to Similar Tools

- **Unlike `brew doctor`** which is Mac-specific, drskill is cross-platform and works with any coding agent
- **Unlike generic YAML/TOML linters**, drskill understands the semantics of skill sets — shadowing resolution, description overlap, harness precedence rules
- **Unlike traditional SAST tools** (which flag code vulnerabilities), drskill flags *agent instruction* vulnerabilities — prompt injection surfaces, credential exposure, egress
- **Unlike `npx skills`** (which installs skills), drskill only reads and reports — it's the doctor, not the surgeon
- **Unlike `Malicious Agent Skills in the Wild`** (which is research measurement), drskill is an operational tool you run in your own repos

## Dependencies

- **Core:** typer, rich, pydantic, pyyaml, tomli-w, tiktoken
- **Deep (optional):** dspy ≥ 3
- **Connect (optional):** mcp ≥ 1.2
- **Dev:** pytest

## Tests

~30 test files covering: check output validation, CLI behavior, discovery, MCP config parsing, MCP connect, deep mode, trace parsing, interactive mode, smoke tests, conformance tests with fixture skill trees.
