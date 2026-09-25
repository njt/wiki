# SILO-BENCH — Anatomy of the Coordination Harness

SILO-BENCH's repo (jwyjohn/acl26-silo-bench) is the implementation behind the paper on the Communication-Reasoning Gap: 30 distributed-algorithm tasks where each LLM agent holds only a private data shard, plus a protocol-parameterized harness that runs 2–100 agents across P2P, broadcast, and shared-file-system communication. What the code reveals beyond the paper is that the whole benchmark is deliberately built on the *dumbest possible infrastructure* — JSON files, regex-parsed XML tool calls, a synchronous round loop — because the experiment is about what agents coordinate, not how the runtime performs.

---

## Architecture

A ~5,400-line Python project (Python 3.13, uv, Pydantic v2) with a clean three-layer split:

- **`benchmark_generator/`** (~2,200 lines) — `main.py` plus one file per paradigm (`paradigm_i.py`, `paradigm_ii.py`, `paradigm_iii.py`). Each generator produces task JSONs with data shards, per-agent prompts, ground truth (`per_agent_values`), and an *executable verification expression string* built by `_build_verification_logic()`. 180 pre-generated tasks ship in `benchmarks/` as `{LEVEL}-{ID}_n{COUNT}.json`.
- **`src/engine.py`** (562 lines) — the shared execution engine with exactly three entry points: `init_case()`, `run_round()`, `evaluate()`. It is protocol-agnostic; `_get_protocol_tools()` does `importlib.import_module(f"src.{protocol}.tools")` to load the protocol plugin at runtime.
- **`src/{msg,broadcast,sfs}/`** — one plugin per protocol, each containing `tools.py` (tool implementations plus an `execute_tool()` dispatcher), plus thin 20–30-line `init_case.py`/`run_round.py`/`evaluate.py` CLI wrappers that all delegate to the engine.

Support modules: `models.py` (Pydantic: `CaseMetadata`, `AgentState`, `Context`, `ToolCall`), `utils/prompts.py` (per-protocol tool documentation), `utils/metrics.py` (S, P, C, D), `utils/parsing.py` (XML tool-call parser), `batch_run.py` (ProcessPoolExecutor matrix runner). No database, no server, no queues — every case is a directory tree: `rounds/round-NNNNNN/agent-NNN/` holding `context.json`, `state.json`, `submission.json`, plus `env/` holding the round's protocol state.

## Key techniques

- **Stateless round-based simulation.** Agents do not run concurrently as processes; `run_round()` iterates agents sequentially, one LLM call per agent per round. Communication is strictly lagged: P2P messages are written to `round-N/env/messages/*.json`, and `receive_messages` scans *previous* rounds for unread messages marked `read:false`. SFS reads come from the *previous* round's `shared_kv.json` while writes land in the current round's copy. This one-round visibility delay eliminates all race conditions by construction — a causal ordering guarantee for free, directly relevant to [[Multi-Agent Systems Have a Distributed Systems Problem]].
- **Protocol-as-plugin via the filesystem.** The three protocols differ only in their `tools.py` and their system-prompt block; the engine, metrics, and evaluation are shared. The entire communication stack — messages, broadcasts, files — is just JSON files read and written per round.
- **XML tool calls, not native function calling.** `utils/parsing.py` extracts tool-call blocks with four regexes plus a hand-rolled type converter (int → float → bool → JSON → str). Tool results are fed back as a *user* message wrapped in XML result tags. Malformed output gets a corrective user message telling the agent the format. This is deliberately provider-agnostic.
- **Round-ending tools as the control mechanism.** `wait` and `submit_result` halt tool execution for the round (a hard `break` in the engine loop); everything else can be batched in one response. Once `state.submitted` is set, the agent's context and submission are simply copied forward each round — a terminated agent costs zero further tokens.
- **The P metric encodes the paper's thesis.** `compute_partial_correctness()` is level-tailored: tolerance-based numeric matching for Level I, per-element positional matching for Level II, and — most tellingly — **longest-increasing-subsequence length over the expected order** (binary-search LIS) for Level III. LIS measures "how much of the globally correct answer did you produce," which is the instrument that isolates the reasoning-integration failure: P much greater than S means information was gathered but not synthesized.
- **Infinite retry as a design choice.** `utils/llm.py` is a tenacity `@retry(wait=wait_fixed(2))` with no retry cap and a 3600-second HTTP timeout — the harness trades possible infinite hangs for never losing a 100-agent × 100-round run to a transient API failure.

## Design decisions

- **Deterministic replay over real-time interaction.** File-per-round persistence makes every run fully auditable (every LLM call, tool call, and result is logged to per-agent JSONL), at the cost of no true asynchrony. Real distributed systems don't have synchronized rounds; the benchmark knowingly abstracts that away to isolate coordination strategy from timing effects.
- **Ground truth baked into generation.** The generator computes expected outputs algorithmically (e.g., shuffled-then-sorted shards for Distributed Sort, task III-21), so correctness is exact and per-agent — enabling both the unanimous-convergence success criterion and per-agent partial scores.
- **Role-free by prompt, not by architecture.** All agents get the same system-prompt template (identity, objective, "ALL agents must submit"), differing only in agent ID. Any leadership or topology that emerges is emergent, which is what makes the observed topology formation a finding rather than an artifact.
- **Trade-offs.** The harness sacrifices scale-realism (sequential rounds, no partial failures, no dropped or reordered messages, Byzantine behavior impossible) and doesn't measure latency. It optimizes hard for reproducibility, cost accounting (token totals tracked per round in `ExecutionInfo`), and cheap parallel execution via `--workers`.

## Comparison notes

This is the executable counterpart to [[Silo-Bench — The Communication-Reasoning Gap]], which covers the paper's findings; the repo adds the how: the LIS-based partial-correctness metric, the lagged-round communication model, and the XML tool protocol. Against [[Multi-Agent Systems Have a Distributed Systems Problem]], SILO-BENCH is notable for *solving* the shared-state problem by fiat — one-round visibility and single-writer JSON snapshots mean the concurrency-control failures Meiklejohn documents can't occur here, which is precisely what lets the benchmark attribute failures to reasoning rather than infrastructure. And relative to [[Towards a Science of Scaling Agent Systems]] — which studies when to add agents across task types — SILO-BENCH supplies the complementary negative result: even on squarely parallelizable tasks, the integration stage fails as agent count grows, so scaling agents cannot substitute for an agent that can synthesize distributed state.

#project #tool #agents #distributed-systems #benchmarks

---
*Sources: [[raw/acl26-silo-bench]], [[summary/acl26-silo-bench]]*
*Last updated: 2026-09-25*
