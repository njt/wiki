---
url: https://github.com/microsoft/Webwright
title: "Webwright: A Terminal Is All You Need For Web Agents"
author: "Yadong Lu, Lingrui Xu, Chao Huang, Ahmed Awadallah (Microsoft Research)"
date_fetched: 2026-05-31
date_published: 2026-05-04
topics:
  - developer-tools
---

# Webwright — Repository Analysis

## Overview

Webwright is a ~1.5K LoC Python web agent harness from Microsoft Research that turns any coding model (Claude, GPT, OpenRouter) into a state-of-the-art browser agent. The core insight: give an LLM a terminal with Playwright installed, let it write and execute Python scripts via bash heredocs, and treat the browser as a disposable inspection tool rather than the agent's state. The persistent artifact is the code and logs in the local workspace, not the browser session.

The project ships as both a standalone CLI (`webwright`) and a Claude Code/Codex/OpenClaw/Hermes plugin. It achieves 86.7% on Online-Mind2Web (GPT-5.4) and 60.1% on Odysseys long-horizon tasks — +15.6 points over prior SOTA.

## Repository Structure

```
src/webwright/
├── __init__.py              # Protocol definitions (Model, Environment, Agent)
├── run/cli.py               # CLI entrypoint (typer), config stacking
├── agents/default.py        # Core agent loop (~460 lines)
├── environments/
│   ├── local_browser.py     # Live Playwright browser env (~570 lines)
│   └── local_workspace.py   # Shell workspace env (~300 lines)
├── models/
│   ├── base.py              # Base model with retries, JSON parsing (~590 lines)
│   ├── anthropic_model.py   # Anthropic Messages API (~190 lines)
│   ├── openai_model.py      # OpenAI Responses API (~160 lines)
│   └── openrouter_model.py  # OpenRouter
├── tools/
│   ├── image_qa.py          # Visual Q&A over screenshots
│   ├── self_reflection.py   # Two-stage screenshot judge (~610 lines)
│   ├── persistent_local_browser.py # CDP-attached Chromium manager
│   └── _model_config.py     # Tool model resolution
├── config/
│   ├── base.yaml            # Default agent config (~410 lines of prompt templates)
│   ├── model_claude.yaml    # Anthropic backend (~14 lines)
│   ├── model_openai.yaml    # OpenAI backend (~12 lines)
│   ├── local_browser.yaml   # Live browser env (~210 lines)
│   ├── persistent_browser.yaml  # CDP-attached workspace mode (~450 lines)
│   └── crafted_cli.yaml     # CLI-tool mode prompts (~400 lines)
├── exceptions.py            # InterruptAgentFlow hierarchy
└── utils/                   # Recursive merge, async runtime, logging
skills/webwright/            # Claude Code plugin skill
tests/                       # 6 unit tests
```

## Source File Analysis

### `__init__.py` — Protocol Layer

Defines three protocols (structural subtyping): `Model`, `Environment`, `Agent`. These are the core abstractions — everything is a duck-typed implementation. Notably uses `Protocol` from `typing` rather than ABCs, meaning no inheritance requirement. Also handles optional dependency shims (`dotenv`, `platformdirs`) and global config directory setup.

### `agents/default.py` — Core Loop

The DefaultAgent implements the loop:

1. Render system + instance templates via Jinja2 (with `StrictUndefined` — no silent template failures)
2. Loop: query model → parse JSON → execute actions → format observations → add to messages → repeat
3. Exit when: step limit exceeded, model sets `done: true`, or self-reflection gate passes

Key mechanisms:
- **History compaction** (`summary_every_n_steps`): Every N steps, sends the full transcript + a "compact this" prompt to the model, replaces all non-system messages with a single summary message. Preserves only the system prompt.
- **ARIA snapshot pruning** (`keep_last_n_observations`): Strips ARIA snapshots from older observation messages to bound context growth. In live-browser mode, ARIA snapshots are ~10-20K chars each.
- **Self-reflection gate** (`require_self_reflection_success`): When the model sets `done: true`, the agent checks for `final_runs/run_<id>/self_reflect_result.json` with `predicted_label == 1`. If missing or failing, `done` is demoted to `false` and the agent receives a gate error message.
- **Debug logging**: Every step writes `debug/steps/step_NNNN.json` and appends to `debug/steps.md`.
- **Bash syntax validation**: Before execution, `bash_command` is validated via `bash -n`.

### `models/base.py` — Model Backend Base

The base model class provides:
- **JSON parse retry**: Up to 3 retries on parse failure, appending format-error messages to the request. On final failure, raises `FormatError` which becomes an `InterruptAgentFlow`.
- **Rate limit + transient error retry**: Separate retry budgets for rate limits (429) and transient HTTP errors (5xx, timeouts). Logs to runtime_errors.jsonl.
- **Structured output schema**: Forces `{thought, <action_field>, done, final_response}` JSON. The action_field is configurable — `bash_command` for workspace mode, `python_code` for live-browser mode.
- **Usage tracking**: Per-request and cumulative token/count metrics available as template variables.
- **Bash validation**: Parses generated bash_command through `bash -n` before accepting.

### `models/anthropic_model.py` — Anthropic Backend

Wraps the Anthropic Messages API. Notable design:
- **Extremely aggressive retry**: 50 rate-limit retries with 30-60s random backoff, 20 transient retries with exponential backoff (1.5s base, 60s cap). This is because Claude Opus org-level ITPM caps can saturate for minutes.
- **retry-after header support**: Respects the `retry-after` response header if present.
- **System prompt extraction**: Merges multiple system messages into a single `system` field (Anthropic's API doesn't support multiple system messages).
- No structured output — relies on prompt-level JSON enforcement.

### `models/openai_model.py` — OpenAI Backend

Uses the OpenAI Responses API with `strict: true` JSON schema — the model is protocol-level constrained to output valid JSON matching the schema. This eliminates parse errors at the API level rather than retrying. Default model is `gpt-5.4`.

### `environments/local_workspace.py` — Shell Workspace

The default environment. Each action is a bash command executed via `subprocess.run` in the workspace directory. The environment provides:
- `WORKSPACE_DIR`, `BROWSER_MODE`, `BROWSERBASE_API_KEY` env vars to subprocesses
- Command timeout (default 240s)
- Output truncation (24K chars)
- Workspace file tracking (40 most recent files)
- Screenshot discovery for observation
- `final_script.py` preview in observation

### `environments/local_browser.py` — Live Browser

An alternative environment where the agent drives Playwright directly. The env owns browser/page lifecycle. Three modes:
- `local_launch`: Fresh Playwright Chromium each run
- `local_persistent`: Persistent context with user-data-dir
- `local_cdp`: Connect to existing Chrome/Edge over CDP (default for live mode)

Each action is `python_code` executed via `exec()` wrapped in an async function with `page`, `context`, `browser`, `playwright`, `task` in scope. After execution, captures URL, title, ARIA snapshot, screenshot, console output.

### `tools/self_reflection.py` — Two-Stage Judge

The most interesting architectural component. A two-stage LLM-as-judge:

1. **Stage 1 — Per-image scoring**: For each screenshot, send (system_prompt, user_prompt + image) to the model, parse `Score: 1-5` and `Reasoning: <text>`. Retries up to 3 times on parse failure. All images scored in parallel via `asyncio.gather`.
2. **Stage 2 — Aggregated verdict**: Drops all per-image reasonings into a final prompt via `{image_reasonings}`, injects the action log via `{action_history_log}`, attaches ALL screenshots, and makes one final call. Parses `Status: success|failure` from the last line.

The agent authors the four prompts once and reuses them across runs. The tool auto-discovers screenshots from the latest `final_runs/run_<id>/screenshots/` directory.

### `tools/persistent_local_browser.py` — CDP Session Manager

Manages a long-lived Chromium subprocess: spawns headless Chromium with `--remote-debugging-port=0`, parses the `DevTools listening on ws://...` line from stderr, persists session JSON. Scripts attach via `connect_over_cdp(connectUrl)` and end with `browser.close()` (which only closes the CDP connection, not the subprocess). Subcommands: `create`, `info`, `release`.

### Config System

YAML configs are stacked via `-c` flags and merged with `recursive_merge` (nested dict merge, `UNSET` sentinel to skip). Three axes:
- **Environment**: base.yaml (shell workspace), local_browser.yaml (live browser), persistent_browser.yaml (CDP workspace)
- **Model**: model_openai.yaml, model_claude.yaml
- **Prompt variant**: base.yaml (one-shot), crafted_cli.yaml (parameterized CLI tool)

Inline overrides via `-c key=value` syntax with `yaml.safe_load` for value parsing and dot-notation key nesting.

### Plugin Distribution

Ships as a Claude Code plugin (`.claude-plugin/plugin.json` + `skills/webwright/SKILL.md`), Codex plugin, OpenClaw skill, and Hermes skill. The skill replaces the JSON-wrapped bash_command loop with native Claude Code tool calls, and replaces image_qa/self_reflection with native PNG reading.

## Architecture Pattern

**Code-as-action agent loop with workspace-as-state**. The architecture is a flat loop:
```
model.query(messages) → parse JSON → execute actions → format observations → repeat
```

No multi-agent system, no graph engine, no plugin layer, no hidden orchestration. The entire loop is ~460 lines in `agents/default.py`.

The key architectural decision is **separating agent from browser**: the browser is something the agent can launch, inspect, and discard. The workspace (code files, screenshots, logs) is the durable state. This is the opposite of most web agents which treat the browser session as workspace.

## Key Techniques

1. **Bash heredocs as the action space**: Instead of a limited tool set (click, type, select), the agent writes arbitrary Python/Playwright in bash heredocs. This lets it compose loops, functions, and abstractions — a qualitative difference from step-by-step DOM interaction.

2. **Two-stage self-reflection gating**: Task completion is blocked until a separate LLM call verifies all screenshots against critical points. The gate is structural (code-enforced), not prompt-level. `done: true` is demoted to `false` if `self_reflect_result.json` doesn't exist or has `predicted_label != 1`.

3. **ARIA snapshot pruning**: Rather than summarization or RAG, the simplest possible context management: keep only the last N ARIA snapshots, strip the rest. ARIA text is ~10-20K chars per step — pruning it is a massive token savings.

4. **Config stacking with recursive merge**: Instead of monolithic config files, YAML configs compose via repeated `-c` flags. This cleanly separates model config, environment config, and prompt variant — you can swap any axis independently.

5. **Persistent CDP browser for exploration**: In persistent_browser mode, a detached Chromium subprocess survives across steps via `connect_over_cdp`/`browser.close()` (CDP-close only). Page state, cookies, open dropdowns persist. This enables multi-step exploration without rebuilding state.

6. **Prompt templates via Jinja2 with StrictUndefined**: All prompts are Jinja2 templates rendered with `StrictUndefined` — any missing variable raises immediately rather than silently rendering empty. Template variables come from model metrics, environment state, and agent config.

## Design Decisions

**Correctness over speed**: The self-reflection gate doubles token cost but enforces verification. Step limit (100) is generous. Rate-limit retries are aggressive (50 for Anthropic).

**Stateless-by-default**: Fresh browser sessions each step sacrifice efficiency for reproducibility. The final artifact (`final_script.py`) must be re-runnable from scratch.

**Simplicity over flexibility**: One agent class (`DefaultAgent`), one loop shape, one output format. The extension points are model backends and environments, not agent topologies.

**Filesystem as memory**: `plan.md` holds the task decomposition. `final_script.py` is the deliverable. `final_runs/run_<id>/` holds run artifacts. There's no database, no vector store, no RAG — just files.

**Structured output as the contract**: The model MUST output valid JSON matching `{thought, action_field, done, final_response}`. OpenAI uses strict schema; Anthropic relies on prompt enforcement with retry.

## Comparison to Related Projects

- **vs Browser Use**: Browser Use is an autonomous LLM agent loop over DOM/AX snapshots with indexed click/type actions. Webwright replaces the action space with free-form Python — the agent writes scripts, not selects actions.
- **vs Stagehand**: Stagehand is hybrid code + NL primitives. Webwright is pure code — no NL-to-action translation layer.
- **vs agent-browser (Vercel)**: agent-browser is a CLI tool another agent calls with discrete subcommands. Webwright gives the agent a terminal and lets it write code directly.
- **vs surf-cli**: surf-cli provides browser automation CLI for agents. Webwright is a full agent harness with self-verification, not just a browser interface.
- **vs Computer Use operators**: Computer Use agents consume screenshots and predict coordinates. Webwright is ~45x cheaper per the paper's data — code agents use ~12K tokens/20sec vs vision agents at 551K tokens/17min.

The core distinction: Webwright treats the browser as something you **program**, not something you **operate**.
