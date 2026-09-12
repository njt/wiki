# atifact

A zero-dependency TypeScript CLI that converts agent session logs into standardized ATIF v1.7 trajectory JSON — the ingestion layer of the agent observability stack. It normalizes four disparate input formats (HAR files from OpenAI/Anthropic APIs, Claude Code JSONL, Copilot CLI JSONL, Codex CLI JSONL) into a single structured schema with steps, tool calls, observations, token metrics, and subagent trajectories. The output feeds downstream tools for debugging, visualization, fine-tuning, and analytics.

---

## Architecture

atifact follows a clean three-phase pipeline: **detect → parse → output**.

**Detection** (`src/detect.ts:4-107`) reads the first 4KB of any input file and identifies the format by content inspection, not file extension. For JSONL formats, it parses the first 5-10 lines looking for distinctive type/subtype markers (Claude Code's `system`/`init`, Copilot CLI's `session.*` or `user.message`, Codex CLI's `thread.started`). HAR detection uses simple string matching for `"log"` + `"version"`/`"entries"`. Detection failures are fatal — no silent fallback.

**Parsing** is handled by four independent parsers, each producing the same `ParseResult` type (`src/types.ts:107-110`): a main `Trajectory` plus an optional `Map<string, Trajectory>` of subagent trajectories. The parsers share no code — each is a complete, self-contained implementation for its format. The HAR parser (`src/parsers/har.ts`, 1,055 lines) is the largest and most complex; it handles three API sub-formats (Anthropic Messages, OpenAI Responses, OpenAI Chat Completions) and must deduplicate multi-turn conversations.

**Output** (`src/index.ts:285-341`) has two modes: file mode writes separate trajectory files with `trajectory_path` cross-references between main and subagent trajectories; `--json` mode embeds subagents in a `subagent_trajectories` array and writes a single JSON object to stdout. A `stripUndefined` function removes null/undefined fields for compact output.

The core type system (`src/types.ts`) defines the ATIF v1.7 schema as TypeScript interfaces: `Trajectory`, `Agent`, `Step`, `ToolCall`, `Observation`, `ObservationResult`, `Metrics`, `FinalMetrics`, and `SubagentTrajectoryRef`. Every parser normalizes into this exact structure.

## Key Techniques

### HAR Multi-Turn Deduplication (`har.ts:279-360`)

This is the hardest problem in the codebase. HTTP Archives capture the full request/response for every API call. In multi-turn conversations, each request replays the ENTIRE conversation history — so extracting messages naively produces duplicates. The solution:

1. **User message dedup**: Track `prevUserMsgCount` across exchanges. Only emit a user step when the count of non-tool-result user messages increases from the previous exchange.

2. **Tool result ownership**: When request N contains tool results, attach them to the agent step from request N-1 (the step that issued those tool calls). But request N also replays tool results from request N-1 — so filter candidates against `prevToolCallIds`, only matching results whose `source_call_id` belongs to the previous agent step's tool calls.

The CHANGELOG records a v0.6.3 fix for this: "Fix HAR parser attaching replayed tool results to agent steps that don't own those tool calls." The test suite validates this with a `dedupedToolResults` case that checks agent steps without tool calls don't accidentally pick up replayed observations.

### SSE Stream Reconstruction

All three API formats stream responses via Server-Sent Events. The HAR parser reconstructs structured content from event deltas by tracking content blocks in indexed Maps:

- **Anthropic** (`har.ts:630-756`): `content_block_start` creates entries by index; `content_block_delta` appends text/thinking/input_json_delta to the matching block; final processing separates text, reasoning_content, and tool_use blocks
- **OpenAI Responses** (`har.ts:760-905`): `output_item.added` creates items; type-specific deltas (`output_text.delta`, `reasoning_summary_text.delta`, `function_call_arguments.delta`) append to matched items
- **OpenAI Chat** (`har.ts:909-1017`): Streaming tool calls are accumulated by index in a Map, with partial JSON arguments concatenated and parsed at the end

### Utility Call Filtering (`har.ts:282-297`)

HAR files from Copilot include lightweight utility calls (gpt-4o-mini title generation). These are filtered by a two-part heuristic: (1) model contains "mini" or "small", AND (2) messages array has ≤3 entries. The length check prevents discarding legitimate exchanges that happen to use small models for longer conversations — a bug that was present until v0.6.2.

### Claude Code Tool Call/Result Pairing (`claude-code.ts:158-261`)

Claude Code JSONL alternates between `assistant` messages (with tool calls) and `user` messages (with `tool_result` content blocks). The parser holds pending assistant steps and attaches observations when the matching `user` message arrives. Subagent messages (identified by `parent_tool_use_id`) are skipped — they're internal infrastructure, not user-visible conversation.

### Copilot CLI Subagent Extraction (`copilot-cli.ts:359-483`)

When Copilot CLI spawns subagents via `task` tool calls, the parser:
1. Identifies `assistant.message` events with `parentToolCallId` (subagent messages)
2. Groups them into separate `Trajectory` objects keyed by parent tool call ID
3. Resolves subagent model from `tool.execution_complete` events with matching `parentToolCallId`
4. Sets `session_id: "parentId:subagentName"` and `trajectory_id: parentToolCallId`
5. Main trajectory references subagents via `subagent_trajectory_ref` in observations

Codex CLI (`codex-cli.ts:280-307`) takes a different approach: it creates stub trajectories with empty steps and an explanatory note, because Codex doesn't inline subagent content in the main log.

### Token Accounting for Prompt Caching

All parsers handle prompt caching tokens carefully. The pattern across formats: `prompt_tokens = input_tokens + cached_tokens + cache_creation_tokens`, `cached_tokens = cache_read_input_tokens`, and cache creation tokens go into `metrics.extra`. This matters because Anthropic and OpenAI report cache tokens differently, and downstream cost analysis needs the breakdown.

## Design Decisions

**Zero runtime dependencies** — the most striking choice. No npm packages for JSON parsing, SSE parsing, or schema validation. ~4,100 lines of handwritten parsing code instead. The trade-off is clear: eliminate supply-chain risk at the cost of maintaining parsers for four evolving formats.

**One output format, four input formats** — normalizes everything into ATIF v1.7 rather than trying to serve each use case with a different output schema. This is a bet on the ATIF standard and on format conversion as the right abstraction layer.

**Default-deny for format ambiguity** — detection failures are fatal (exit code 1). The `--format` flag is the explicit escape hatch. No silent assumptions about what format a file might be.

**Subagent dual strategy** — file mode writes separate files with cross-references; `--json` mode embeds everything. This serves two different consumption patterns: disk-based workflows vs. shell pipelines.

**Primary model by counting** — the most-used non-utility model becomes the agent's `model_name`. For Copilot CLI, a priority chain: `session.tools_updated` → first main-agent `tool.execution_complete`. Falls back to `"unknown"` — never a hardcoded assumption.

**Codex stubs are honest** — rather than silently omitting Codex subagents (whose content isn't in the log), atifact creates empty stub trajectories with an explanatory note. This preserves the structure without fabricating content.

## Comparison Notes

**vs. [[session-analysis]]**: session-analysis consumes JSONL directly for analytics (wall time, tokens, cost). atifact is the format normalization layer — it could feed structured trajectories to session-analysis, which would then only need to understand ATIF, not 4 different formats.

**vs. [[claude-replay]]**: claude-replay renders agent sessions as HTML replays. atifact + claude-replay form a pipeline: atifact converts raw logs to structured JSON, claude-replay visualizes them. Complementary, not competing.

**vs. [[engineering-notebook]]**: Both consume the same input sources (Claude Code, Codex CLI sessions). atifact produces machine-readable structured JSON; engineering-notebook produces human-readable summaries. Different consumers, same producer ecosystem.

**vs. [[AgentsView]]**: AgentsView is a dashboard for agent analytics. atifact's ATIF output is exactly the structured data format a tool like AgentsView would consume for cross-session, cross-model, cross-agent comparisons.

**vs. [[Components of a Coding Agent]]**: The core insight — "the harness matters more than the model" — applies perfectly to atifact. There's no AI model involved; all the value is in the software engineering of the parsing pipeline. Pure harness.

**vs. [[AI Pricing]]**: AI Pricing provides per-token model pricing data. atifact captures the usage metrics that AI Pricing's rates would be applied to for cost analysis.

**vs. [[har-extractor]]**: Both parse HAR files, opposite use cases. atifact's 1,055-line HAR parser extracts API conversation trajectories with multi-turn deduplication and SSE reconstruction; har-extractor's 63-line implementation extracts web assets from browser DevTools HARs using a URL-to-path heuristic. Same container format, different payloads, 16× line-count difference.

---
*Source: [[summary/atifact]] — full repo clone and deep analysis*
*Last updated: 2026-08-01*
*Tags: #tool #project #agents #analytics*
