# Webwright

Webwright is a ~1.5K LoC web agent harness from Microsoft Research that turns any coding model into a state-of-the-art browser agent by giving it a terminal with Playwright. Instead of predicting one DOM action at a time, the model writes complete Python/Playwright scripts via bash heredocs, runs them, inspects screenshots, and iterates. The final deliverable is a re-runnable `final_script.py` — not a trace of actions. Achieves 86.7% on Online-Mind2Web and 60.1% on Odysseys long-horizon tasks (+15.6 over prior SOTA), and ships as a Claude Code/Codex/OpenClaw plugin.

---

## Architecture

The core is a flat loop in `agents/default.py` (~460 lines):

```
model.query(messages) → parse JSON → execute actions → format observations → repeat
```

No multi-agent system, no graph engine, no hidden orchestration. Three core abstractions defined as `Protocol` in `__init__.py`:

- **Model** — LLM backend (OpenAI Responses API with strict JSON, Anthropic Messages API, OpenRouter)
- **Environment** — where actions execute (shell workspace or live browser)
- **Agent** — the loop that connects them

**Two environments** ship out of the box:

- `local_workspace` (`environments/local_workspace.py`): Each action is a bash command executed via `subprocess.run`. The agent writes Playwright scripts as heredocs. The browser is spawned fresh per script — no persistent state.
- `local_browser` (`environments/local_browser.py`): A live Playwright session where each action is `python_code` executed via `exec()` with `page`/`context`/`browser`/`playwright` in scope. Three browser modes: `local_launch`, `local_persistent`, `local_cdp` (connect to real Chrome/Edge).

**Config stacking** (`config/__init__.py`): YAML configs compose via repeated `-c` flags with recursive merge. Three axes are independent — you can swap environment, model, and prompt variant without touching other config:

```bash
webwright -c base.yaml -c model_claude.yaml -t "task" --start-url https://...
webwright -c base.yaml -c local_browser.yaml -c model_openai.yaml -t "task"
webwright -c base.yaml -c crafted_cli.yaml -c model_openai.yaml -t "task"
```

---

## Key Techniques

### Code-as-action via bash heredocs

Instead of a fixed tool set (click, type, select), the model writes arbitrary Python in heredocs (`python - <<'PY' ... PY`). This lets it compose loops, functions, and abstractions — a qualitative difference from step-by-step DOM interaction. The action field switches between `bash_command` (workspace) and `python_code` (live browser).

### Self-reflection completion gate

The most distinctive architectural choice. When the model sets `done: true`, the agent does NOT accept it. Instead, it checks `final_runs/run_<id>/self_reflect_result.json` for `predicted_label: 1`. If missing or failing, `done` is demoted to `false` and the agent receives a gate error telling it exactly what's wrong.

The self-reflection tool (`tools/self_reflection.py`, ~610 lines) is a two-stage LLM judge:
1. **Per-image scoring** (parallel): Each screenshot scored 1-5 with reasoning, retried 3× on parse failure
2. **Aggregated verdict**: All reasonings + action log + all screenshots → one final call → parse `Status: success|failure`

The agent authors the four judge prompts once and reuses them across runs.

### ARIA snapshot pruning

Rather than summarization or RAG, the simplest possible context management: `keep_last_n_observations: 1` strips ARIA snapshots from all but the most recent observation. ARIA text is ~10-20K chars per step — pruning it saves massive tokens. Older observations keep URL, title, and output but lose the snapshot.

### History compaction

Every N steps (`summary_every_n_steps: 20`), the full transcript is sent to the model with a "compact this" prompt. The result replaces all non-system messages with a single summary. This is a separate LLM call that can't fail the run (errors are silently caught).

### Persistent CDP browser

`tools/persistent_local_browser.py` manages a detached Chromium subprocess. Scripts attach via `connect_over_cdp` and end with `browser.close()` — which only closes the CDP connection, keeping the subprocess alive. Page state, cookies, open dropdowns survive across steps, enabling multi-step exploration.

### Prompt templates with StrictUndefined

All prompts are Jinja2 templates rendered with `StrictUndefined` — any missing variable raises immediately. Template variables come from model metrics (token counts, usage), environment state (workspace_dir, browser_mode), and agent config.

---

## Design Decisions

**Correctness over speed**: The self-reflection gate doubles token cost but enforces verification. Step limit is generous (100). Anthropic rate-limit retries are aggressive (50 retries, 30-60s backoff).

**Stateless-by-default**: Fresh browser sessions each step sacrifice efficiency for reproducibility. The final artifact (`final_script.py`) must be re-runnable from scratch.

**Filesystem as memory**: `plan.md` holds task decomposition. `final_script.py` is the deliverable. `final_runs/run_<id>/` holds artifacts. No database, no vector store, no RAG — just files.

**Structured output as contract**: Model MUST output `{thought, action_field, done, final_response}` JSON. OpenAI uses strict schema at the API level; Anthropic relies on prompt enforcement with up to 3 parse retries.

**Plugin-first distribution**: Ships as Claude Code plugin, Codex plugin, OpenClaw skill, and Hermes skill. The skill replaces the JSON-wrapped loop with native Claude Code tool calls and replaces image_qa/self_reflection with native PNG reading.

**Three config variants for prompt engineering**:
- `base.yaml` — one-shot task script
- `persistent_browser.yaml` — same but with CDP-attached exploration
- `crafted_cli.yaml` — parameterized CLI tool with argparse + Google-style docstrings

---

## Comparison Notes

- **vs [[Browser Use]]**: Browser Use predicts indexed click/type actions over DOM snapshots. Webwright replaces the action space with free-form Python — the agent writes scripts, not selects from a menu.
- **vs Stagehand**: Stagehand is hybrid code + NL primitives with optional LLM translation. Webwright is pure code — no NL-to-action translation layer, no `act()`/`extract()` abstractions.
- **vs [[surf-cli]]**: surf-cli provides browser automation CLI for agents to call. Webwright is a full agent harness with plan→explore→execute→self-verify workflow, not just a browser interface.
- **vs [[Computer Use is 45x More Expensive Than Structured APIs]]**: This project IS the counterexample. Webwright's code-as-action approach uses ~12K tokens/20sec vs vision agents at 551K tokens/17min. The gap is architectural, not model-dependent.

The core distinction: Webwright treats the browser as something you **program**, not something you **operate**. The browser is disposable — the workspace is the durable state.

---

*Sources: [[raw/webwright-microsoft]]*
*Last updated: 2026-05-31*
*Tags: #tool #project #agents #browser-automation #playwright #web-agent*
