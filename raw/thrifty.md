---
url: https://skills.2389.ai/plugins/thrifty/
date_fetched: 2026-07-05
backfilled: true
---

#### building-multiagent-systems

v1.0.0Architecture patterns for multi-agent systems with orchestrators, sub-agents, and tool coordination

Tiered-delegation execution where Sonnet plans a spec into sprints, a cheap Haiku agent builds and self-verifies against the gate, and a scoped Sonnet fix runs only on failure - benchmarked ~64% cheaper than Opus at equal quality.

`npx skills add 2389-research/thrifty``/plugin install thrifty@2389-research`Full plugin documentation and usage guide

**Tiered-delegation task execution for Claude Code.** A planner model writes the spec

into sprints, a cheap model executes and self-verifies against the gate — same gate

quality as the strong model, **~64% cheaper** (≈⅓ the cost) (benchmarked).

Offload each sprint of work to theweakest model that can do it correctly.

Most of the work in a task is pattern-following execution, not reasoning. thrifty

concentrates planning in one model and pushes execution down to a cheap one, with the**gate (tests / a checklist) as the trust contract** — independently re-run, never

self-reported. This generalizes "tests as the source of truth" to any task, code or not.

The configuration that won the cost/quality bake-off (spec → working code, 7 tasks across

JS / Python / Go / prose, gate-verified — see `eval/RESULTS.md` and`experiments/`):

```
spec ──▶ Sonnet  writes contract (pins cross-sprint + genuinely-ambiguous decisions)
                 + sprints.jsonl (one self-contained unit of work each)
            │
            ▼
         Haiku   one cached agent builds every sprint, runs the gate, and self-fixes
            │
            ▼
         Sonnet  a SCOPED patch — only if a specific failure is left (in practice: never;
                 the Haiku agent self-fixed to green on all 7 tasks)
```
**Result: ~64% cheaper than Opus building the same spec, at equal gate quality** — on*large, multi-unit* builds (the win grows with size). On trivial builds the planning

overhead makes thrifty cost *more* than a single capable agent, so there's a crossover

below which you should just use one strong model directly (see Usage). Two

things make it work: a single cached agent reuses the contract across turns (cheap, no

cold-call bloat), and the contract **pins every decision that crosses a sprint boundary**

so the executor never invents a system-level choice (leave one ambiguous and the executor

writes contradictory tests). Opus is *not* in the execution loop.

thrifty ships as a Claude Code **plugin** (bundles all six skills):

```
/plugin marketplace add 2389-research/thrifty
/plugin install thrifty@thrifty
```
Then trigger it with **"thrifty"**, **"delegate this"**, or **"tiered build"**.

The subagent-based skills (`thrifty`,`thrifty-plan/brief/execute/check`) need no external runtime. The lean`thrifty-dispatch`flow shells out to`skills/thrifty-dispatch/dispatch.py`, so it additionally requiresPython 3on`PATH`.

To hack on it locally instead, symlink the skills into `~/.claude/skills/`:

```
for d in skills/*/; do ln -s "$PWD/$d" ~/.claude/skills/; done
```
thrifty generalizes three existing systems to arbitrary tasks:

Where `local_code_gen` writes 600-line byte-pinned contracts for a tiny local

model (qwen3.6 / gemma), thrifty deliberately sits well above that floor. **Haikuis far stronger**, so the contract pins only what is 

| Skill | Role | Model | Runs as | 
|---|---|---|---|
| `thrifty` | orchestrator (subagent flow) | — | this session | 
| `thrifty-dispatch` | orchestrator (lean dispatch flow — default) | — | this session | 
| `thrifty-plan` | director's planning discipline | Sonnet | this session | 
| `thrifty-brief` | expand a unit spec into a brief (split tier) | Sonnet | dispatched subagent | 
| `thrifty-execute` | execute one unit | Haiku | dispatched subagent | 
| `thrifty-check` | verify + fix one unit | Sonnet | dispatched subagent | 

`local_code_gen`'s
  contract → sprint flow). Brief-writing runs in parallel and the architect's context
  stays lean. Best for Rough rule: direct below ~5 sprints, split above — but it's per-task, and the subtle

sprints can stay direct even in a split run. Authority follows the tier: the architect

owns the contract (cross-sprint), the brief-writer owns its unit (within-sprint), so a

within-sprint fix stays with a Sonnet brief-writer and only contract defects reach the

architect.

Say **"thrifty"**, **"delegate this"**, or **"tiered build"** on a multi-part task.

**Which flow runs:** the agent should **try the lean dispatch flow ( thrifty-dispatch)first** — it's the benchmarked, cheapest path — and fall back to the subagent flow below

`python3` / `claude -p` can't be spawned) or`thrifty` skill now points the agent at dispatch by default.For the subagent flow, the orchestrator will: frame the goal, plan (write `CONTRACT.md` + per-sprint briefs

with acceptance criteria, confirmed with you), dispatch Haiku executors in parallel

where dependencies allow, verify each unit by criterion type** (run the gate forrunnable criteria; spend a Sonnet read only when a gate fails or there areassertional criteria to judge), apply a bounded tiered fix loop, then do a finalcoherence pass and report.**

So for code-with-tests the common path costs no checker tokens; Sonnet is spent only

on failures and on things only a reader can judge.

Artifacts land in `docs/thrifty/<task-slug>/` (`CONTRACT.md`, `briefs/`,`LEDGER.md`) so a run is auditable and resumable.

Not every task is "one file per agent." The architect picks a mode by how cohesive

the final artifact must be:

A single artifact with no cross-sprint seams skips the contract entirely — one brief,

one execute, one check (or just don't use thrifty).

The checker diagnoses (`pass` / `local` / `execution` / `brief`); the orchestrator

decides and counts:

A regression guard rolls back any fix that breaks a previously-passing criterion.

The benchmarked architecture and the full cost/quality investigation (including the

approaches we tried and rejected) are in `eval/RESULTS.md`; reproducible

scripts + captured data are in `experiments/`.

Recommended: the lean dispatch flow (It is the benchmarked architecture and the one the ~64% cost claim is measured on. Reach for the subagent substrate only when you specifically need per-unit`thrifty-dispatch`).

`thrifty-dispatch`: Sonnet writes`contract.md` + `sprints.jsonl`, then `dispatch.py` runs bare `claude -p` calls that
  `thrifty-*` skills above (Sonnet architect**Why it costs more (structural, not a bug):** every subagent spawn carries ~40k of

  Claude Code harness context, and each subagent's report lands back in the orchestrator's

  context — which is then re-read every turn, so cost compounds with the number of units.

  The dispatch flow avoids both (bare calls, outputs to disk, manifest-only reads).

**Executor-tier caveat:** unlike dispatch, this path **cannot force the executor model in  code** — it asks the orchestrator to dispatch with 

`model: "haiku"`, which a runtime may`thrifty`'s Step 3.)
v0.1 — **benchmarked.** The lean dispatch flow is ~64% cheaper than Opus building the same

spec, at equal gate quality, across 7 tasks (JS / Python / Go / prose). Evidence:`eval/RESULTS.md` (main findings + full journey) and`experiments/` (reproducible scripts + captured cost data).

Get started in seconds

`npx skills add 2389-research/thrifty`
            `Claude Code, Cursor, Codex…`
            `Skills auto-trigger when relevant`
            `/plugin marketplace add 2389-research/claude-plugins`
            `/plugin install thrifty@2389-research`
            `Skills auto-trigger when relevant`
