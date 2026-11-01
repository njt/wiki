# Luna Doctor

`luna-doctor` is an offline diagnostic CLI (Python 3.9+, ~12K lines, zero dependencies beyond stdlib) that answers a question no error string can: when your coding-agent sessions stall, hang, or 429 on the [LunaRoute](https://lunaroute.com) gateway, *whose wall did the traffic hit?* LunaRoute bills concurrent requests, not tokens, and its failure mode is deliberately quiet — it waits, silently demotes overflow to a background tier, holds the stream open with SSE keep-alives while producing nothing, and lets the harness kill the stall with a misleading client-side error. The tool reads the log stores the machine already has, reconstructs what was asked at what tier, and names the owner of each wait. It matters as a case study in two things: observability for agent traffic from local logs alone, and a diagnostic tool whose epistemology (provenance labels, refusal to guess, refusal to generate load) is enforced in code.

---

## Architecture

Single-package Python CLI, no runtime dependencies (`pyproject.toml` requires only setuptools):

- `luna_doctor/cli.py` (3,932 lines) — the CLI, the log readers and the analyser. The `Request` dataclass is the core abstraction: one model request with harness, session, tier, demotion reason, outcome class, and a `usage_raw` dict that keeps each harness's own token fields **unmerged**.
- `luna_doctor/sources.py` (1,508 lines) — the data-collection layer with a six-word provenance vocabulary: `measured` (from local logs), `declared` (config files), `catalog` (GET /v1/models, disk-cached), `account` (GET /v1/capacity, disk-cached), `configured`, `unknown`. Every fact carries which file/key or endpoint it came from.
- `luna_doctor/attribution.py` — pure functions placing the local half beside the account half to name the wall. Three invariants in code: never add the halves (a local peak is a lower bound, not a term in a sum), never state an account number as a gateway fact, never guess — "unknown" names the command that fetches the missing half.
- `luna_doctor/_fix_block.py` and `_chart_block.py` — `fix` and `chart`, kept in separate files specifically so their snippets can be audited on their own; `cli` loads them off disk from its own directory.
- `luna_doctor/report.py` / `render_pretty.py` / `style.py` — one `Report` model, two renderers: `plain` (no ANSI, no emoji, no column padding — paste-safe at any width) and `pretty` (decorated). `--json` constructs no Style object at all.
- `luna_doctor/scene.py` / `png.py` / `font.py` — the chart is rendered twice from one shape list: an SVG and a PNG rastered with a hand-drawn 5×7 bitmap font, so the image has no embedded fonts, no scripts, no CDN.
- `luna_doctor/baseline.py` — the before/after window comparison, with the rule that under a sample floor the verdict is `inconclusive` and no string ever says "since", "because" or "fixed".

## Key techniques

- **Measured, not assumed thresholds.** The `CONTRACT` dict in `cli.py` carries only constants verified against real stores (25.2s pre-header wait, 300s client idle budget, 10s keep-alive cadence). The admission deadline is deliberately *absent* from the contract because it is per model (120.2s glm-5.3, 60.3s deepseek-4.1-flash); `per_model_deadlines()` measures it from each window's own logs and reports sample counts, never lending one model's number to another.
- **Error-string classification as reverse engineering.** `classify()` maps raw client strings to outcome classes, with the comments documenting the reverse-engineered gateway contract: `scheduler admission timeout` is the gateway, `Stream ended without finish_reason` is pi's own idle budget, `x-lunaroute-warning: overflowed-to-background` marks silent demotion. Ordering matters — a context-length 400 is checked before `model_not_found` because both carry `param: model`.
- **Refusal to generate load.** The 0.3.0 changelog records *deleting* the `load-test` and `compare` commands: "a diagnostic run must not add to the pressure it is asked to explain." The width has exactly two honest sources — the account API and a `--plan` declaration — and neither opens a session.
- **The skill package as tested contract.** `tests/test_skill_package.py` parses every subcommand's `--help` and fails if any skill file names a flag the parser doesn't carry, links a dead path, or pushes the router past its line/word budget. The prose an agent loads cold is verified against the CLI it will type into.
- **Honest nulls.** A missing denominator prints `-- (no requests)`, not 0%; a capacity value from neither source prints `unknown` with the command that would fetch it; `--plan` is labelled `(declared plan <name>)` and set aside when the account reports its own number.

## Design decisions

The trade-off everywhere is **honesty over usefulness**: a quieter, less decisive report is chosen over a confident wrong one. Token totals are never merged across harnesses because codex's `input_tokens` includes cached tokens while pi's `input` excludes `cacheRead` — "one merged 'input tokens' number is therefore a silent wrong answer." The `fix` command never writes a file; it emits an auditable command plus a verification loop (`analyze --baseline prev`). Codex requests are counted in the combined snapshot but excluded from outcome classes, gated to the harnesses `--harness` names. The chart deliberately omits a universal deadline and any advisory figure, because those are per-model and per-plan. The cost is that `unknown` appears often, and the user must sometimes run a second command to complete the picture — by design.

## Comparison notes

- It inverts the posture of [[Managing AI Coding Costs at Scale]]: that playbook optimises spend inside token-billing vendors with gateways as infrastructure; here the gateway *is* the vendor and the billing unit (concurrency) is invisible, so the tool reconstructs the meter from the client side — the mirror image of Databricks's top-down cost governance.
- Compared with [[The Three Pillars of Observability]], it is a stance on agent observability: no agent, no daemon, no telemetry pipeline — just harness session logs the machine already wrote, with provenance as the organising principle. It is the zero-infrastructure pole of the observability database debate.
- Its SKILL.md-as-verified-artifact practice sharpens what [[Matt Pocock Skills — Skills for Real Engineers]] and [[Claude Code Mastery]] treat as craft: the playbooks always end in a verdict so an agent cannot finish without naming whose wall it was, and a unit test guarantees the skill's command index matches the parser — skills as regression-tested interface, not prompt folklore.
- The five-verdict attribution ("you", "your org", "the gateway", "not concurrency", "unknown") echoes the evidence-with-summary discipline of [[Review AI Coding-Agent Work Without Losing the Next Step]] and the verdict-first structure of [[Aviator Verify]]: both refuse to let an agent assert without saying what it checked.
- Its refusal to generate load, delete of its own load-test commands, and "never open concurrent requests of your own to test a theory" invariant put it in the same family as [[AI Handles Incidents, Engineers Lose Touch with Their Systems]] — diagnostics that don't perturb the system they diagnose.

#tool #project #agents #observability #cli

---
*Sources: [[raw/luna-doctor]], [[summary/luna-doctor]]*
*Last updated: 2026-11-02*
