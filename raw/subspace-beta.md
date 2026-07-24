---
url: https://github.com/spacedock-dev/subspace-beta
title: Subspace Beta
author: Spacedock
date_fetched: 2026-07-25
date_published: 2025
---

# Subspace Beta — Full Analysis

## What It Is

Subspace is a tool that lets AI coding agents open a Markdown file in a native terminal TUI for human review, then returns structured, validated feedback to the agent. It's a bridge between AI agents and human reviewers — a way for an agent to say "human, look at this" and get back actionable, machine-readable annotations.

Distributed as plugins for Claude Code and Codex (via spacedock-dev/marketplace), with a required proprietary binary (`subspace-tui` v0.10.0-beta.3) installed via Homebrew. The repo itself (~614 lines of bash + SKILL.md + plugin.json) is Apache-2.0 licensed; the binary is not.

## File Tree

```
.
├── README.md                           (112 lines)
├── compatibility.json                  (7 lines)
├── LICENSE                             (Apache-2.0, plugin files only)
├── .claude-plugin/plugin.json          (12 lines)
├── .codex-plugin/plugin.json           (23 lines)
├── assets/
│   ├── review-one-file.gif
│   └── anchored-feedback.png
└── skills/r/
    ├── SKILL.md                        (131 lines)
    └── scripts/
        ├── invocation-common           (213 lines) — shared lifecycle
        ├── review-zellij               (51 lines)
        ├── review-tmux                 (42 lines)
        ├── review-herdr                (43 lines)
        ├── review-cmux                 (58 lines)
        ├── review-ghostty              (32 lines)
        ├── review-apple-terminal       (53 lines)
        └── validate-one-file-result    (122 lines) — canonical validator
```

## Architecture

### Layer 1: LLM-Hosted Skill (SKILL.md)

A 131-line markdown skill file loaded by the AI agent host (Claude Code or Codex). It defines:
- Input contract: exactly `<file.md>` or `<file.md> <terminal>`
- Terminal auto-detection logic (fixed priority: Zellij → tmux → Herdr → CMUX → Ghostty → Apple Terminal)
- Permission model: must obtain host permission before probing any terminal
- Result reporting: feedback is input, never authorization. The word "Decision" is explicitly forbidden.
- What the skill does NOT do: plan, probe, version-check, scrub, retry, fallback, route workflows, grant authority

### Layer 2: Dispatch Scripts (Bash)

Each terminal has a dedicated entry point (`review-<terminal>`) that sources `invocation-common` for the shared lifecycle. Two dispatch models:

**Synchronous (blocking):**
- tmux: `display-popup -E` — blocks until popup closes
- Zellij: `new-pane --blocking --floating --close-on-exit` — blocks until pane exits

**Asynchronous (FIFO-based):**
- Apple Terminal: `osascript do script` launches a child capsule, completion signaled via named FIFO
- Ghostty: `open -n -a Ghostty.app --args -e <capsule>` with FIFO completion
- Herdr: pane split + `pane run` with FIFO
- CMUX: `new-surface` + `respawn-pane` with FIFO, plus explicit cleanup (`close-surface`, `focus-panel`)

### Layer 3: Proprietary Binary (subspace-tui)

The closed-source binary handles:
- Briefing construction (a structured document describing what to review)
- Artifact snapshotting at child start
- Native TUI rendering (presumably a terminal UI for reading + annotating)
- Atomic result creation (mode-0600 JSON)
- Continuation support (open reviews that can be resumed)

### Shared Lifecycle (invocation-common)

A 213-line bash library sourced by all terminal entries. It performs in this exact order:

1. **Artifact validation**: Must be absolute path, readable regular file (not symlink), parent directory searchable
2. **Canonicalization**: Resolves up to 40 symlink levels, verifies the result is canonical
3. **Revision pinning**: SHA-256 hash of the artifact
4. **Binary resolution**: Finds `subspace-tui`, canonicalizes path, verifies executable
5. **Version check**: Runs `subspace-tui --version`, requires exact match to `0.10.0-beta.3`
6. **Validator check**: Verifies `validate-one-file-result` exists and is executable
7. **Host preflight**: Delegates to terminal-specific checks
8. **Scratch creation**: `mktemp -d` with umask 077, mode-0700 verified
9. **Async preparation** (if applicable): Creates FIFO + child capsule script
10. **Dispatch**: Delegates to terminal-specific launch
11. **Result collection**: Either reads FIFO or polls for result file (250 × 20ms timeout)
12. **Result capture**: Copies result to capture.json, runs validator
13. **Trusted delivery**: Outputs validated bytes to stdout
14. **Cleanup**: Removes all scratch files and directory on success; preserves on failure

### Result Contract (validate-one-file-result)

A 122-line jq-based validator that enforces the `review-v1-result` schema. Two valid result shapes:

**Feedback (closed review):**
```json
{
  "type": "review-v1-result",
  "status": "feedback",
  "briefing": "briefing:single-file:<hash>",
  "artifact": {"id": "artifact:primary", "uri": "<path>", "mediaType": "text/markdown", "rev": "sha256:<hash>"},
  "annotations": [...],
  "actor": "person:reviewer"
}
```

**Open (continuation):**
```json
{
  "type": "review-v1-open-result",
  "status": "open",
  "briefing": "briefing:single-file:<hash>",
  "artifact": {...},
  "package": "<absolute-path>",
  "log": "<package>/briefing.review.jsonl"
}
```

Annotations are either `comment` (body text, no selectors required) or `suggestion` (original+proposed text, selectors required). Selectors use the W3C `TextQuoteSelector` model with exact/prefic/suffix fields.

## Key Techniques

### Strict No-Fallback Terminal Detection

Each terminal has required environment signals. If ANY signal from the highest-priority family is present but ALL required signals aren't, the system fails with a diagnostic — it never falls through to the next terminal. This is deliberately wasteful: it prioritizes clear error messages ("ZELLIJ_SESSION_NAME present but ZELLIJ_PANE_ID missing") over silently trying the next option.

### Artifact Revision Pinning

Before any human interaction, the system hashes the file with SHA-256. The validator later checks that the artifact snapshot inside the result package matches this hash. This prevents a class of bugs where the file changes between when the agent hands it off and when the human reviews it.

### Capsule Pattern for Async Terminals

For terminals that can't block (Apple Terminal, Ghostty, Herdr, CMUX), the system constructs a self-contained "capsule" script that:
1. `cd`s to the artifact directory
2. Runs `subspace-tui --actor person:reviewer --result <path> <artifact>`
3. Reports exit status through a named FIFO
4. Has trap handlers for HUP/INT/TERM that report status codes 129/130/143

The capsule is mode-0700, the FIFO is mode-0600. This turns an inherently async operation (launching a GUI terminal app) into a reliable completion signal.

### Canonical Path Resolution

The `invocation_canonical_file` function resolves symlinks up to 40 levels deep (rejecting deeper chains), handles both absolute and relative symlink targets, and produces a clean canonical path. The system then REJECTS the path if it differs from the supplied path — requiring the caller to provide an already-canonical path.

### Dual Validation of Result Integrity

The validator doesn't just check JSON schema. It also:
- Verifies `cmp -s result capture` (byte-for-byte identical)
- Checks file permissions (mode-0600 for result, mode-0700 for package dir)
- Validates the briefing hash derivation (`sha256("subspace:one-file-invocation:v1\0<result-path>")`)
- For open results: validates the package directory structure, snapshot revision, briefing.json, and review log

## Design Decisions

### Proprietary Core, Open Integration

The hardest part (the TUI, Briefing construction, continuation state) is closed-source. The integration layer (plugin manifests, terminal dispatch, validation) is open. This is the opposite of most "open core" models — the secret sauce is the binary, not the plugin.

### Single File Only

The skill accepts exactly one Markdown file. No directories, no multiple files, no other formats. This constraint simplifies the Briefing model, the artifact identity, and the result contract. It also means the system is strictly a review tool, not a general artifact-viewer.

### Neutral Authority

The skill explicitly refuses to record a "Decision" — the result is always "feedback." An invoking workflow decides what to do with it. This is architectural, not just documented: the result schema has no "approved"/"rejected" field, and the skill text forbids calling any outcome an approval or verdict.

### No Automatic Remediation

The skill doesn't apply suggestions, fix files, or route workflows. It only reports. This keeps the tool composable — any workflow system can consume the result and decide its own response.

### Permission Model

Before probing any terminal, the skill must obtain permission. It states exactly what it will do (pin artifact, inspect ONE caller, create ONE private directory, open ONE review surface, wait for result, validate, return, clean). For manual permission mode, this becomes a single approval dialog. For automatic mode, zero stops. The skill explicitly says "do not ask a separate natural-language yes/no question."

### Terminal Hosts as Equal Citizens

Despite differing capabilities (blocking vs non-blocking, native vs third-party), each terminal gets a dedicated entry point with equal standing. There's no "primary" terminal with others as fallbacks — just a fixed detection priority. The shared lifecycle handles the differences through the `invocation_async` flag.

## Comparison to the Wiki

[[Local Review]] also does local git branch review with markdown export for coding agents, but Subspace is different: it's for human review DURING an agent session, not offline branch review. Local Review exports markdown for agents to consume later; Subspace returns structured JSON for the agent to consume immediately.

[[cmux]] is one of Subspace's supported terminals — a macOS-native terminal built on libghostty designed for managing multiple AI coding agent sessions. Subspace integrates with cmux's surface API for review pane management.

[[The End of Code Review]] and [[Agentic Code Review]] discuss the spectrum from mandatory human review to fully automated review. Subspace occupies an interesting middle ground: it enables human review in an agentic workflow without granting the human approval authority. The review is input, not a gate.

[[Orchestrating AI Code Review at Scale]] (Cloudflare) uses 7 specialized AI agents + a coordinator judge. Subspace takes the opposite approach: one human reviewer, no AI agents in the review process, no judgment — just structured feedback.

[[Aviator Verify]] replaces code review with intent-based verification (did the agent do what was agreed?). Subspace doesn't verify intent; it collects human feedback that a workflow might later use for verification. They're complementary layers.
