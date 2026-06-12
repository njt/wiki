# OpenCodeReview

OpenCodeReview (OCR) is Alibaba's open-source AI code review CLI — a Go tool that sends git diffs to configurable LLMs via a tool-using agent. Battle-tested inside Alibaba for two years across tens of thousands of developers and millions of detected defects before being open-sourced. Its core insight: combine deterministic engineering (file selection, rule matching, line resolution) with an LLM agent (dynamic judgment, context retrieval) rather than relying on pure language-driven review.

---

## Architecture

OCR is a **hybrid deterministic+agent** CLI. A single Go binary wraps the full pipeline: diff extraction → concurrent per-file agent subtasks → comment collection → output.

### Pipeline overview

```
git diff → ParseDiffs → Filter files → [Per-file goroutine]
                                          ├─ Plan Phase (LLM)
                                          ├─ Main Tool Loop (LLM + 5 tools)
                                          ├─ Review Filter (LLM removal pass)
                                          └─ Collect comments
```

- **Entry**: `cmd/opencodereview/main.go` dispatches `review`, `config`, `llm`, `rules`, `viewer` subcommands
- **Agent**: `internal/agent/agent.go` (1,488 lines) — orchestrates the full pipeline
- **Diff engine**: `internal/diff/git.go` — workspace/range/commit modes with `.gitignore`-aware filtering
- **LLM layer**: `internal/llm/client.go` (759 lines) — unified interface over Anthropic and OpenAI SDKs
- **Tools**: `internal/tool/` — 5 tools the LLM can call during review

### Per-file subagent concurrency

Each changed file gets its own goroutine with a separate LLM conversation. A bounded semaphore (default 8 concurrent) controls parallelism. This isolates context per file — the agent doesn't see diffs from other files unless it explicitly calls `file_read_diff` or `code_search`.

### Three-phase file review

1. **Plan Phase** (optional, `PLAN_TASK`): LLM analyzes the diff and outputs a structured JSON plan with severity-ranked issues and recommended tool calls. Skipped when change lines < `PLAN_MODE_LINE_THRESHOLD` (50).

2. **Main Tool Loop** (`MAIN_TASK`): The LLM iterates calling tools — reads files, searches code, submits comments — until it calls `task_done` or hits `MAX_TOOL_REQUEST_TIMES` (30). If 3 consecutive rounds produce no valid tool results, the loop exits.

3. **Review Filter** (`REVIEW_FILTER_TASK`): A separate LLM call checks generated comments against the diff. The prompt explicitly instructs the model to *falsify*, not verify — only remove comments provably wrong from diff evidence.

### Five tool capabilities

| Tool | Purpose |
|------|---------|
| `file_read` | Read any file with line ranges (500 line max) |
| `file_find` | Find files by name/glob |
| `file_read_diff` | Access diffs of other changed files |
| `code_search` | `git grep` across repo (100 results, 10s timeout) |
| `code_comment` | Submit review findings |
| `task_done` | Signal completion |

Tools are registered in a `Registry` that freezes after setup — the LLM can't register new tools mid-review.

---

## Key Techniques

### Dual-threshold async context compression

As the conversation grows, `addNextMessage()` in `agent.go:1149` checks two thresholds against `MAX_TOKENS`:

- **60% soft**: triggers background goroutine compression (non-blocking)
- **80% hard**: synchronous compression, blocks the main loop

Messages are `partitionMessages()`-ed into three zones:
- **Frozen** (messages[0:2]): always preserved (system prompt + first user message)
- **Compress** (middle rounds): summarized by a separate `MEMORY_COMPRESSION_TASK` LLM call into `<previous_review_summary>` XML
- **Active** (K most recent rounds): preserved intact, fitting within budget

Async jobs can be superseded — if the main loop hits the hard threshold before the background job completes, it cancels the async and compresses synchronously.

### Comment positioning pipeline

Unlike pure-LLM review where line numbers are hallucinated, OCR uses a multi-stage approach:

1. **Text matching**: `diff.ResolveComment()` in `internal/diff/hunk.go` matches the comment's `existing_code` snippet against diff hunks
2. **LLM re-location**: if matching fails, `diff.ReLocateComment()` calls the LLM with `RE_LOCATION_TASK` to regenerate a better code snippet, then retries matching — the original snippet is preserved as fallback
3. **Final resolution**: `diff.ResolveLineNumbers()` in `review_cmd.go` applies all resolved positions

### Four-layer rule cascade

`internal/config/rules/system_rules.go` implements a priority stack:

1. Custom (`--rule` flag)
2. Project (`.opencodereview/rule.json`)
3. Global (`~/.opencodereview/rule.json`)
4. System defaults (embedded per-language rules in `rule_docs/`)

Rules use `doublestar` glob matching with brace expansion (`*.{go,py}` → `*.go`, `*.py`). First match wins.

### Template-based prompt engineering

All prompts live in `internal/config/template/task_template.json` with `{{placeholder}}` substitution — iterable without recompilation. The template defines five task types (MAIN, PLAN, MEMORY_COMPRESSION, RE_LOCATION, REVIEW_FILTER), each with role/content messages and timeouts.

### Session persistence for debugging

Every review session writes a JSONL file to `~/.opencodereview/sessions/` with UUID-chained records (session_start → llm_request → llm_response → tool_call → session_end). A built-in web viewer (`ocr viewer`) lets you browse and replay sessions with host-header allowlist security.

### Anthropic cache optimization

When using Anthropic, the last system block and last tool definition get `ephemeral` cache control — reusing cached system prompts and tool schemas across rounds within the same file's conversation.

---

## Design Decisions

**Optimized for: determinism and coverage.** File selection is deterministic (git diff + pattern filters). Each file gets its own review. Line numbers are algorithmically resolved with LLM fallback. Bad comments get a second-pass filter.

**Sacrificed: cross-file analysis.** Since each file has its own LLM conversation, the agent can't naturally detect patterns spanning files. The `change_files` context and `file_read_diff` tool mitigate this but the architecture fundamentally biases per-file.

**Single binary, embedded everything.** Templates, rules, and prompt files use Go's `//go:embed` — zero runtime file dependencies beyond user config.

**No streaming.** LLM calls are request-response, not streaming. Simpler code but no partial progress and harder timeout tuning.

---

## Comparison Notes

**vs. AI PR Reviewer / CodeRabbit**: SaaS with dashboards. OCR is self-hosted CLI with per-file concurrent agents and full session JSONL for debugging.

**vs. Claude Code review skill**: A skill sends all diffs to Claude in one prompt. OCR decomposes — deterministic file selection, isolated per-file context, algorithmic line resolution, filter pass for hallucinations. The skill is simpler; OCR is more reliable at scale.

**vs. [[Guardrails and Feedback Loops]]**: OCR is an implementation of the "deterministic enforcement, not instructions" principle. File filtering, rule matching, and position resolution are all deterministic engineering around the LLM.

**vs. [[Broomy]]**: Broomy runs coding agents side-by-side with built-in code review. OCR is a dedicated review tool with a different architecture — concurrent per-file subagents rather than a single agent with a review skill.

**vs. [[StrongDM Factory Techniques]]**: SMF validates code by harness (does it build/pass/deploy?). OCR validates by deep analysis (does the logic make sense?). Complementary approaches on different points of the trust spectrum.

---

Tags: #tool #project #agents #code-review #quality

*Sources: [[raw/alibaba-open-code-review]]*
*Last updated: 2026-06-12*
