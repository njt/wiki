---
url: https://github.com/waldekmastykarz/atifact
title: atifact — Convert agent logs to ATIF trajectories
author: Waldek Mastykarz
date_fetched: 2026-06-09
date_published: 2026-04 (v0.1.0), latest v0.8.0
---

# atifact — Full Repo Analysis

## Overview

`atifact` is a zero-dependency TypeScript CLI that converts agent session recordings into standardized ATIF v1.7 trajectory JSON. It supports four input formats: HAR files (OpenAI Chat, OpenAI Responses, Anthropic Messages APIs), Claude Code CLI JSONL logs, Copilot CLI JSONL logs, and Codex CLI `exec --json` logs. Output is ATIF v1.7 — a standard from the Harbor Framework project designed for agent debugging, visualization, fine-tuning, and RL pipelines.

Version: 0.8.0. License: MIT. Engine: Node.js >=22. 4,110 total lines of source + test code.

## File Structure

```
.
├── package.json          # Zero runtime deps, TypeScript 6.0.2 devDeps
├── tsconfig.json
├── README.md
├── CHANGELOG.md          # v0.1.0 → v0.8.0, detailed evolution
├── AGENTS.md             # Internal: ATIF spec version update checklist
├── skills/atifact/SKILL.md  # Claude Code skill definition
├── src/
│   ├── index.ts          # CLI entry point (347 lines): arg parsing, dispatch, output
│   ├── types.ts          # ATIF v1.7 TypeScript type definitions (119 lines)
│   ├── detect.ts         # Input format auto-detection (107 lines)
│   └── parsers/
│       ├── har.ts        # HAR file parser (1,055 lines) — the largest and most complex
│       ├── claude-code.ts # Claude Code JSONL parser (377 lines)
│       ├── copilot-cli.ts # Copilot CLI JSONL parser (546 lines) — subagent extraction
│       └── codex-cli.ts  # Codex CLI JSONL parser (331 lines) — spawn_agent stubs
├── test/
│   ├── cli.test.ts       # Full CLI integration tests (286 lines)
│   ├── detect.test.ts    # Format detection tests (53 lines)
│   ├── har.test.ts       # HAR parser tests (299 lines)
│   ├── claude-code.test.ts # Claude Code parser tests (162 lines)
│   ├── copilot-cli.test.ts # Copilot CLI parser tests (251 lines)
│   ├── codex-cli.test.ts # Codex CLI parser tests (177 lines)
│   └── fixtures/         # 18 test fixtures (HAR, JSONL variants)
└── .github/
    ├── workflows/ci.yml
    ├── workflows/publish.yml
    └── skills/bump-version/SKILL.md
```

## Architecture

### Pipeline: detect → parse → output

The CLI (`src/index.ts`) follows a three-phase pipeline:

1. **Detection** (`src/detect.ts`): Reads first 4KB of the input file and checks for format markers in the first 10 lines (JSONL) or string patterns (HAR). Order matters — Claude Code, Codex CLI, and Copilot CLI all produce JSONL, so detection uses specific type/subtype markers.

2. **Parsing** (one of 4 parsers): Each parser reads the full file, walks the events/lines, and builds an ATIF `Trajectory` object. Returns a `ParseResult` with main trajectory + optional `Map<string, Trajectory>` of subagent trajectories.

3. **Output** (`src/index.ts`): In file mode, writes main to `<prefix>.trajectory.json` and subagents to `<prefix>.trajectory.<name>.json` with trajectory_path cross-references. In `--json` mode, embeds subagents in `subagent_trajectories` array and prints to stdout. A `stripUndefined` function removes null/undefined fields for compact JSON.

### ATIF v1.7 Type System (`src/types.ts`)

The core abstraction is the `Trajectory`:
- `Agent` (name, version, model_name, tool_definitions)
- `Step[]` (step_id, source: system|user|agent, message, model_name, reasoning_content, tool_calls, observation, metrics)
- `ToolCall` (tool_call_id, function_name, arguments)
- `Observation` / `ObservationResult` (source_call_id, content, subagent_trajectory_ref)
- `Metrics` / `FinalMetrics` (prompt_tokens, completion_tokens, cached_tokens, cost_usd)

Every step from every format normalizes into this structure. The diversity is in the parsers, not the output.

## Key Techniques

### 1. HAR Multi-Turn Deduplication (har.ts:279-360)

This is the hardest problem in the codebase. HTTP Archives capture the full HTTP request/response for every API call. In multi-turn conversations, each subsequent request includes the ENTIRE conversation history (messages array grows with each turn). If the parser naively extracts user messages from every request, it duplicates them.

The solution: track `prevUserMsgCount` and only emit a user step when the count of non-tool-result user messages in the current request exceeds the previous request's count. Similarly, tool results from request N are attached to the agent step from request N-1 by matching `source_call_id` against the previous step's `tool_calls`.

The edge case that makes this subtle: tool results also appear in repeated history. The parser must filter observation candidates against `prevToolCallIds` — only results whose `source_call_id` matches a tool call from the PREVIOUS agent step get attached. This prevents replayed tool results from request 3 from being attached to agent step from request 2 (which used different tools).

The CHANGELOG records a bug fix for this in v0.6.3: "Fix HAR parser attaching replayed tool results to agent steps that don't own those tool calls."

### 2. Utility Call Filtering (har.ts:282-297)

HAR files from Copilot often include lightweight utility calls (gpt-4o-mini title generation requests). These are filtered out by checking if the model contains "mini" or "small" AND the messages array has ≤3 entries. This heuristic explicitly avoids discarding legitimate exchanges using small models with longer conversations (fixed in v0.6.2: "Fix overly broad utility call filtering").

### 3. SSE Stream Reconstruction

All three API formats use Server-Sent Events for streaming. The HAR parser reconstructs agent responses from SSE streams by:
- **Anthropic**: `content_block_start`/`content_block_delta` events accumulated by index into a Map, then processed into text, thinking (reasoning_content), and tool_use blocks
- **OpenAI Responses**: `output_item.added` + `output_text.delta`/`reasoning_summary_text.delta`/`function_call_arguments.delta` events
- **OpenAI Chat**: Streaming `tool_calls` deltas reconstructed by index into a Map, then JSON-parsed

### 4. Subagent Trajectory Extraction (copilot-cli.ts:359-483)

Copilot CLI spawns subagents via `task` tool calls. The parser:
1. Identifies `assistant.message` events with `parentToolCallId` fields
2. Groups these into separate `Trajectory` objects keyed by the parent tool call ID
3. Uses `tool.execution_complete` events with matching `parentToolCallId` for subagent tool results
4. Resolves subagent model from its tool completions
5. Sets `session_id` to `parentSessionId:subagentName` and `trajectory_id` to the parent tool call ID
6. The main trajectory references subagents via `subagent_trajectory_ref` in the observation

Codex CLI handles subagents differently: since Codex doesn't inline subagent content in the main log, the parser creates stub trajectories with empty `steps[]` and a note explaining the limitation.

### 5. Claude Code Tool Call/Result Pairing (claude-code.ts:158-261)

Claude Code JSONL has `assistant` messages with tool calls followed by `user` messages containing `tool_result` content blocks. The parser holds pending assistant steps that have tool calls and attaches observations when the matching user message arrives. Subagent messages (identified by `parent_tool_use_id`) are skipped entirely.

### 6. Format Detection by Content Inspection (detect.ts)

Never trusts file extensions. For JSONL formats, parses the first 10 lines looking for specific type/subtype markers:
- Claude Code: `type: "system"` + `subtype: "init"` + `session_id` string
- Copilot CLI: `type` starting with "session." or equals "user.message" + has timestamp
- Codex CLI: `type: "thread.started"` + `thread_id` string
- HAR: string.contains `"log"` + (`"version"` or `"entries"`)

The Claude Code detector scans the first 10 lines (not just the first) because `rate_limit_event` lines can precede the init line (fixed in v0.6.4).

## Design Decisions

### Zero dependencies — deliberate and sustained

Every dependency carries a supply-chain risk. By using only Node.js stdlib, atifact has no transitive dependencies, no `node_modules` audit, and no version compatibility matrix. The trade-off: manual JSONL line parsing, manual SSE parsing, and manual JSON Schema adherence (3,000+ lines of typed parsing code).

### One output format, many input formats

Rather than trying to serve every use case with different output schemas, atifact normalizes everything into ATIF. This is a bet that the ATIF standard will be useful — and that format conversion is the right abstraction layer. Downstream tools (claude-replay, session-analysis) don't need to understand 4 different input formats.

### File mode vs. stdout mode as two distinct output strategies

In file mode, subagents are separate files with `trajectory_path` references. In `--json` mode, they're embedded in a `subagent_trajectories` array. This is a practical compromise: files work for disk-based workflows (Obsidian, file-based tools), while stdout works for shell pipelines and jq processing.

### Default-deny for format ambiguity

Format detection failures are fatal errors (exit code 1), not silent defaults. The `--format` flag is the escape hatch. This forces users to be explicit about ambiguous inputs rather than silently producing wrong output.

### Model detection priority

Primary model is determined by counting non-utility exchanges and picking the most-used model. For Copilot CLI, model resolution has a priority chain: `session.tools_updated` → first main-agent `tool.execution_complete` with model field. The default model fallback is `"unknown"` — never a hardcoded assumption about which model was used.

### Codex CLI subagent stubs

Codex CLI `exec --json` output doesn't inline subagent content in the main log file. Rather than silently omitting subagents, atifact creates stub trajectories with empty `steps[]` and a note. This is honest about what's missing — better than pretending subagents don't exist.

## Comparison Notes

### vs. session-analysis
[[session-analysis]] analyzes agent session JSONL for wall time, tokens, and cost. atifact produces the ATIF trajectory; session-analysis could consume it. atifact is about format conversion; session-analysis is about analytics on the resulting structure.

### vs. claude-replay
[[claude-replay]] renders agent sessions as self-contained HTML replays. atifact is a potential upstream — convert raw logs to ATIF, then render. The two projects are complementary: atifact normalizes, claude-replay visualizes.

### vs. engineering-notebook
[[engineering-notebook]] creates automatic engineering diaries from Claude Code and Codex sessions. Similar input sources, different output: atifact produces structured machine-readable JSON; engineering-notebook produces human-readable summaries.

### vs. AgentsView
[[AgentsView]] is a local-first analytics dashboard for coding agents. The trajectory format atifact produces is exactly the kind of structured data a dashboard like AgentsView could consume for comparative analytics across sessions, models, and agent types.

### vs. Components of a Coding Agent
[[Components of a Coding Agent]] identifies "the harness matters more than the model" as a core insight. atifact embodies this: it's pure harness — parsing, normalizing, and structuring agent output without any model involvement. The value is entirely in the software engineering of the conversion pipeline.

## Key Numbers

- 4,110 total lines (source + tests)
- 1,055 lines: HAR parser (the most complex due to SSE parsing + dedup)
- 546 lines: Copilot CLI parser (most feature-rich due to subagent extraction)
- 18 test fixtures covering edge cases
- 4 input formats, 1 output format (ATIF v1.7)
- 0 runtime dependencies
- 13 releases from v0.1.0 to v0.8.0 (April–June 2026)
