---
url: https://github.com/alibaba/open-code-review
title: OpenCodeReview — AI-Powered Code Review CLI
author: Alibaba Group
date_fetched: 2026-06-12
date_published: 2025
topics:
  - guardrails-and-feedback-loops
---

# OpenCodeReview — Full Analysis

## Summary

OpenCodeReview (OCR) is a Go CLI tool (~4,100 LOC of core logic) that performs AI-powered code review by sending git diffs to configurable LLMs via a tool-using agent. Originally Alibaba's internal code review assistant — serving tens of thousands of developers and catching millions of defects over two years — it was open-sourced after production validation at scale. The key insight: it combines deterministic engineering (file selection, rule matching, comment relocation) with an LLM agent (dynamic decision-making, context retrieval) in a hybrid architecture.

## Architecture

### Entry Point
- `cmd/opencodereview/main.go` — dispatches subcommands: `review`, `config`, `llm`, `rules`, `viewer`, `version`
- `cmd/opencodereview/review_cmd.go` — assembles all dependencies (template, rules, tools, LLM client) and launches the Agent

### Core Agent Loop (`internal/agent/agent.go`, 1,488 lines)
The Agent is the heart of the system. Its Run() pipeline:
1. **Diff parsing** — via `internal/diff/` — uses git diff/show with 3 context lines, handles workspace/range/commit modes, filters by extension/path/pattern
2. **Per-file concurrent subtasks** — each changed file gets its own goroutine bounded by a user-configurable semaphore (default 8). Each subtask runs:
   a. **Plan Phase** (optional, skipped if change < threshold): LLM analyzes the diff and produces a structured JSON review plan with severity-ranked issues and recommended tool calls
   b. **Main Task Loop**: LLM tool-use loop calling `file_read`, `file_find`, `file_read_diff`, `code_search`, `code_comment`, `task_done`. Max 30 tool requests per file, 3 max consecutive empty rounds before giving up
   c. **Review Filter Phase**: separate LLM call that checks generated comments against the diff and removes ones provably wrong from diff evidence alone

### Tool System (`internal/tool/`)
Five tools available to the LLM agent:
- `file_read` — read arbitrary file content with line ranges (500 line max)
- `file_find` — find files by name/glob pattern
- `file_read_diff` — access diffs of other changed files for context
- `code_search` — git grep across the repo with case-sensitive/regexp/Perl regexp modes (100 results max, 10s timeout)
- `code_comment` — submit review findings; parsed and line-number-resolved against the diff

Tools are registered in a `Registry` that freezes after setup (concurrent-read safe). Tools are defined in `internal/config/toolsconfig/tools.json` and converted to LLM tool definitions (OpenAI/Anthropic format) with per-phase filtering.

### LLM Abstraction (`internal/llm/client.go`, 759 lines)
Unified `LLMClient` interface with two implementations:
- **OpenAIClient** — uses `openai-go/v3` SDK, handles system/user/assistant/tool role mapping, reasoning_content extraction
- **AnthropicClient** — uses `anthropic-sdk-go`, maps to Claude's content block format, sets cache control on last system block and last tool definition

Token counting uses `tiktoken-go` with `cl100k_base` default, `o200k_base` for o1/o3/o4 models. Caches tokenizers per encoding name.

### Context Compression (`internal/agent/agent.go:1249-1430`)
Dual-threshold strategy:
- **60% soft threshold**: triggers async background compression in a goroutine
- **80% hard threshold**: synchronous compression, blocks the main loop

Messages are partitioned into three zones: frozen (first 2 messages), compress (middle rounds), active (most recent K rounds fitting in budget). Compressed rounds are summarized via a separate LLM call (MEMORY_COMPRESSION_TASK) and the summary is injected into the second frozen message as `<previous_review_summary>`. Async jobs can be superseded; if the main loop hits hard threshold before async completes, it cancels the async job and compresses synchronously.

### Comment Pipeline
Comments go through multiple processing stages:
1. **Parse** — `ParseComments()` in `tool/code_comment.go` extracts structured comment objects from LLM tool call arguments
2. **Line resolution** — `diff.ResolveComment()` matches `existing_code` snippets against diff hunks to determine exact line numbers
3. **Re-location** — `diff.ReLocateComment()` calls the LLM with RE_LOCATION_TASK to extract a better code snippet when text matching fails
4. **Collection** — comments accumulate in a `CommentCollector`
5. **Filter** — `executeReviewFilter()` runs a separate LLM pass to remove comments that are provably wrong from the diff alone
6. **Async processing** — code_comment processing can run on a separate `CommentWorkerPool` (bounded goroutine pool) so the main LLM loop isn't blocked

### Session Persistence (`internal/session/`)
Every review session writes a JSONL file to `~/.opencodereview/sessions/<encoded-repo-path>/<uuid>.jsonl` with:
- `session_start` / `session_end` records
- `llm_request` / `llm_response` / `llm_error` records
- `tool_call` records
- UUID chain via `parentUuid` for full conversation reconstruction

### Viewer (`internal/viewer/`)
Built-in web server (`ocr viewer`) to browse persisted sessions. Go stdlib HTTP server with embedded templates, host-header allowlist to prevent DNS rebinding attacks accessing session data.

### Rule System (`internal/config/rules/`)
Four-layer priority: custom (`--rule` flag) > project (`.opencodereview/rule.json`) > global (`~/.opencodereview/rule.json`) > embedded system defaults. System defaults ship with per-language rule files (Go, Java, Python, Rust, C/C++, Kotlin, TypeScript, YAML, JSON, etc.) embedded via `//go:embed`. Rules are matched by glob patterns using `doublestar`, with brace expansion for `*.{go,py}` syntax.

### Template System (`internal/config/template/`)
Task prompts are JSON templates loaded from embedded `task_template.json` with `{{placeholder}}` substitution. Templates define:
- MAIN_TASK, PLAN_TASK, MEMORY_COMPRESSION_TASK, RE_LOCATION_TASK, REVIEW_FILTER_TASK — each with role/content messages
- MAX_TOKENS (58888), MAX_TOOL_REQUEST_TIMES (30), PLAN_MODE_LINE_THRESHOLD (50), etc.

### Diff Engine (`internal/diff/`)
Three modes:
- **Workspace**: `git diff HEAD` + staged diff + untracked file synthetic diffs
- **Range**: `git diff merge-base(from,to)..to` 
- **Commit**: `git show <commit>`

Custom `.gitignore` pattern matching (no external library). Hardcoded exclusion of `.idea/`, `.vscode/`, `vendor/`, `node_modules/`, etc. plus `.gitignore` patterns.

### Telemetry (`internal/telemetry/`)
OpenTelemetry integration with OTLP gRPC exporters for traces and metrics. Optional stdout exporters. Records spans for diff parsing, subtask execution, LLM requests. Metrics for review duration, files reviewed, comments generated.

## Key Techniques

1. **Hybrid deterministic+agent**: Unlike pure prompt-based code review (e.g., a Claude Code skill), OCR uses deterministic code for file selection, rule matching, and line resolution while reserving the LLM for dynamic judgment and context retrieval. This addresses the "position drift" and "incomplete coverage" problems of pure language-driven review.

2. **Dual-threshold async context compression**: The 60%/80% soft/hard compression with async background work is a clever optimization. It avoids blocking the main loop for compression in common cases while having a hard fallback when context grows too fast. The async job can be superseded (the pendingJob pointer is atomically swapped), so stale compressions don't clobber newer state.

3. **Three-zone message partitioning**: `partitionMessages()` divides conversation history into frozen (always kept), compress (summarized), and active (most recent K rounds) zones. This preserves system prompts and recent context while compressing the middle. The frozen zone is always messages[0:2] — the system prompt + first user message.

4. **Review filter as adversarial check**: The REVIEW_FILTER_TASK uses a separate LLM call to remove comments that are provably wrong based on diff evidence alone. The prompt explicitly tells the model "you need to falsify, not verify" and "if a claim involves information outside the diff, do not make a determination." This is a clever two-pass quality filter.

5. **Comment re-location via LLM**: When simple text matching can't find a comment's target in the diff, `ReLocateComment()` calls the LLM with a specialized prompt to regenerate a better `existing_code` snippet. The original is preserved; if re-location still fails, the original is restored. This is a fallback that keeps comments from being lost due to minor format mismatches.

6. **Per-file subagent parallelism**: Each changed file gets its own LLM conversation — separate context, separate tool loop. This is architecturally different from sending all diffs in one prompt. It prevents context pollution across files and naturally supports concurrency. The trade-off is that cross-file issues may be missed.

7. **cache_control on last block**: When using Anthropic, the last system block and last tool definition get `ephemeral` cache control. This means the system prompt and tool definitions are cached across rounds, reducing input token costs for the same file's conversation.

8. **Template-based prompt engineering**: Prompts are not hardcoded — they're in `task_template.json` with `{{placeholder}}` substitution. This means the prompt engineering can be iterated without recompilation, and different languages/organizations can ship different templates.

## Design Decisions

**Optimized for**: determinism and coverage at scale. The architecture ensures every file gets reviewed (concurrent per-file subtasks), line numbers are accurate (deterministic resolution + LLM re-location fallback), and bad comments are filtered (review filter pass).

**Sacrificed**: cross-file analysis. Since each file gets its own LLM conversation, the agent can't naturally detect patterns that span files. The `change_files` context and `file_read_diff` tool mitigate this somewhat, but the architecture fundamentally biases toward per-file analysis.

**Trade-offs in comment processing**: code_comment can run async (worker pool) or sync. Async reduces latency for the main loop but complicates error handling and means comments from failed subtasks may still be collected.

**Language choice**: Go was chosen over Python/Node (likely for deployment simplicity — single binary) and over Rust (faster development). The concurrency model maps naturally to the per-file subtask pattern.

**No streaming**: The LLM abstraction uses request-response, not streaming. This simplifies the code but means the user can't see partial results and timeouts are harder to tune.

**Embedded everything**: Templates, rules, and even prompt files are embedded via `//go:embed`. This means the binary is fully self-contained — no runtime file dependencies beyond user config.

## Comparison Notes

**vs. AI PR Reviewer / CodeRabbit**: Those are SaaS products; OCR is self-hosted CLI. OCR's per-file concurrent agent architecture contrasts with the more common single-prompt review approach. OCR gives you session JSONL for debugging; SaaS tools give you a dashboard.

**vs. Claude Code Skills for code review**: A Claude Code skill for review sends the full diff set to Claude and asks for comments. OCR decomposes the problem: file selection is deterministic, per-file context is isolated, comment positions are algorithmically resolved, and a filter pass removes hallucinated findings. The skill approach is simpler but suffers from the "position drift" problem OCR explicitly solves.

**vs. General linting tools**: OCR doesn't replace linters — it complements them. Linters find pattern-based issues deterministically; OCR finds semantic and logical issues that require understanding intent. OCR's embedded per-language rules (`rule_docs/*.md`) are closer to linter rules than general review.

**vs. [[StrongDM Factory Techniques]]**: SMF's "code as opaque weights, validated by harness not review" is the opposite philosophy. OCR IS the harness, performing deep review with tool access. Two different points on the trust spectrum.

## Project Stats
- ~4,100 LOC core logic (agent.go: 1,488, client.go: 759, system_rules.go: 416, git.go: 348)
- Go 1.25, Apache 2.0 license
- Dependencies: Anthropic SDK, OpenAI SDK, tiktoken-go, OpenTelemetry, doublestar (glob matching)
- Cross-platform: distributes as Go binary + npm wrapper
