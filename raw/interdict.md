---
url: https://github.com/prisharai/Interdict
title: Interdict — Safety Layer Between AI Agents and Postgres
author: Prisha Rai (pr482@cornell.edu)
date_fetched: 2026-07-08
date_published: 2026-06
---

# Interdict — Safety Layer Between AI Agents and Postgres

A Python runtime safety layer that sits between AI coding agents (via MCP) and PostgreSQL. It parses, classifies, and gates every SQL statement before it touches the database, then captures before/after images of all allowed writes so they can be instantly reverted.

## Project Stats

- **Language:** Python 3.11+
- **Dependencies:** pglast (libpg_query binding for real Postgres AST), asyncpg, mcp, pyyaml
- **Code size:** ~9,100 lines across engine/ (3,500 lines), adapters/ (1,160 lines), tests/ (4,500 lines)
- **Tests:** 343 automated tests in CI including concurrency races, fault injection, and evasion attacks
- **License:** MIT
- **Author:** Prisha Rai, Cornell

## Architecture

Interdict is a **layered pipeline** with a thin transport adapter. Every agent SQL statement flows through:

```
AI agent ──(MCP over stdio)──> [mcp_server.py] ──> [SAFETY ENGINE] ──> Postgres
                                      │                    │
                                 ShadowSession    parse → classify → policy
                                                  → (simulate?) → decide
                                                  → (record undo) → execute
                                                  async: audit log, intent check
```

### Engine modules (the core — transport-agnostic)

1. **parser.py** (121 lines) — Wraps `pglast` (libpg_query, the actual Postgres parser) with an LRU cache (2048 entries). Imposes hard caps before parsing: 8KB max input, 200 max commas, 400 max parentheses — these are O(1) guards that prevent pathological inputs from blowing the latency budget. Returns a `ParseResult` dataclass, never raises onto the hot path. ~0.1ms cold on realistic statements.

2. **classifier.py** (457 lines) — Walks the Postgres AST to determine what a statement *actually does*: read/write/DDL, tables and columns touched, presence of WHERE, nested DML (data-modifying CTEs), system catalog access, locking clauses, called functions, and "point write" detection (single-column equality predicates). LRU-cached on raw SQL. Never uses string matching — everything is derived from AST structure. Classifies unrecognized node types as DDL-severity (fail-closed).

3. **policy.py** (593 lines) — Deterministic, YAML-driven policy engine. `evaluate()` collects ALL violations (not just the first) with machine-readable reason codes, human explanations, and suggested fixes — so the agent can self-correct in one round trip. Pure in-memory, no I/O. Supports: read-only mode, multi-statement blocking, system catalog blocking, WHERE-required-on-writes, default-deny table allowlists, DDL allowlists, blocked columns (per-table), blocked functions, locking clause blocking, and LIMIT auto-injection on unbounded reads.

4. **simulate.py** (245 lines) — Blast-radius measurement (the first differentiator). Only runs on "risky writes" — UPDATE/DELETE/MERGE not scoped to a known-unique column. Two paths: (a) cheap planner estimate via `EXPLAIN (FORMAT JSON)`, (b) precise via `BEGIN; SET LOCAL timeouts; <stmt>; ROLLBACK` — actually executes the write to get exact affected rows, then rolls back. Hard time-boxed (1s statement timeout, 200ms lock timeout). Data-modifying CTEs are reported as unmeasurable and fail closed.

5. **undo.py** (653 lines) — Reversibility/instant undo (the second differentiator). For allowed writes only. Captures before-image (SELECT ... FOR UPDATE) and after-image (RETURNING *) in a single transaction with the write. Stores in a sidecar schema (`adb_undo.undo_log`). Revert is **conditional and atomic**: only restores rows that still match the after-image, using jsonb comparison — so it never clobbers a later concurrent write. Detects PK changes, multi-table writes, upserts, and CTE-wrapped writes as non-reversible shapes and blocks them by default.

6. **session.py** (508 lines) — Transport-agnostic interactive gate (`GuardedSession`). Two-phase: `propose()` evaluates without executing (returns a `Proposal` with verdict, blast radius, violations), then `execute()` runs it (refusing BLOCK verdicts and gating CONFIRM behind explicit human approval). Supports an override escape hatch (audited) for human operators only — never available on the MCP path.

7. **intent.py** (496 lines) — Advisory intent-mismatch detection. Compares the agent's stated task (free text) against what the statement actually does. Four deterministic heuristics: read-only language on writes, scope contradiction (single-sounding task with large blast radius), table mismatch, and schema-destructive DDL vs row-level task. Context-aware number parsing distinguishes durations from row counts. Optional out-of-band LLM second opinion (fire-and-forget, never gates the query).

8. **schema.py** (597 lines) — Versioned transport-clean schema for ActionRequest → Decision. Defines the v2 canonical types (Principal, Action, Decision with Impact/Undo/Hold), plus migration bridges from legacy policy types.

9. **audit.py** (170 lines) — Async, non-blocking audit log. `record()` is a synchronous `Queue.put_nowait` (never awaits, never blocks). Background consumer drains to JSONL via `asyncio.to_thread`. Drops records (and counts them) if queue is full rather than blocking. SQL is redacted to SHA-256 hashes.

10. **approvals.py** (193 lines) — Persistent out-of-band approval store for held writes. Lives in the same sidecar schema as undo. Operator token is hashed (SHA-256) before storage; raw token never appears in the database, chat, or transcripts. Holds expire after TTL (default 30 min). Atomic status transitions prevent double-execution.

11. **security.py** (96 lines) — Secret redaction for audit logs (API keys, DSNs, tokens), safe client-facing error messages.

### Adapter

**adapters/mcp_server.py** (1,161 lines) — The MCP server an agent (Claude Code, Codex) launches over stdio. Thin glue: manages connection pool lifecycle, loads policy once at startup, loads unique-column metadata from Postgres, exposes 4 MCP tools (`run_query`, `run_approved_query`, `list_pending_approvals`, `revert_write`, `interdict_status`), and a CLI subcommand (`interdict pending/approve/deny`) for the human operator. The `ShadowSession` class orchestrates the full pipeline: classify → evaluate → simulate (if risky) → intent check → hold-or-execute.

### Other directories

- **policies/** — YAML policy files. `default.yaml` is database-agnostic (all tables allowed, guarded by structural rules + simulation + undo). `pagila.yaml` is the fully locked-down example with a default-deny table allowlist.
- **corpus/** — `red/queries.yaml` (40 statements that MUST be blocked) and `green/queries.yaml` (18 statements that MUST be allowed), used as regression tests. Red corpus includes evasion variants (comments, casing, whitespace, quoted identifiers, data-modifying CTEs, EXPLAIN ANALYZE wrapping).
- **benchmarks/** — Latency harness with CI gate (fails if p99 exceeds 5ms).
- **tests/** — 343 tests: enforcement, evasion, concurrency races, fault injection, intent detection, smoke, corpus regression, undo, policy, schema.
- **research/** — Academic study on specification gaming in database guardrails. Closed-loop experiment measuring whether richer error feedback (opaque → reason code → reason+fix → reason+fix+blast radius) increases or decreases evasion attempts.

## Key Techniques

### 1. Real Postgres AST, never regex

The single most important design choice. By using `pglast` (which binds libpg_query), Interdict sees statements exactly as Postgres will — comments stripped, multi-statement splits resolved, case normalized, aliases expanded. This means:

- `/* cleanup */ DELETE FROM rental -- safe` is still a bare DELETE
- `dRoP tAbLe film` is still a DROP TABLE
- `WITH d AS (DELETE FROM rental RETURNING *) SELECT * FROM d` is detected as a write, not a read
- `EXPLAIN ANALYZE DELETE FROM rental WHERE rental_id = 1` is detected as a wrapped write (EXPLAIN ANALYZE actually executes)
- Alias stars (`s.*`) and whole-row references (`row_to_json(s)`, bare `s`) resolve through the alias map to the base table, so blocked columns can't leak

The evasion test matrix runs 7 dangerous statements through 7 disguises (lowercase, uppercase, block comments, line comments, newlines/tabs, inner comments) — all must still be blocked.

### 2. Multi-layer hot-path optimization

The latency budget is ruthlessly enforced:

- **Layer 0 — Cheap O(1) guards before parsing:** 8KB max input, 200 max commas, 400 max parens. A 40KB IN-list that would take ~400ms to parse is rejected in microseconds.
- **Layer 1 — LRU caches on parse and classify:** Identical repeated statements (common with agent loops/retries) skip even the AST walk. Cache size: 2048 entries each.
- **Layer 2 — Pure in-memory policy evaluation:** No I/O, no network, no LLM on the decision path. Microseconds.
- **Layer 3 — Simulation only for risky writes, time-boxed:** Reads and scoped point writes never pay for simulation. Precise simulation is capped at 1s statement timeout + 200ms lock timeout.
- **Layer 4 — Async, non-blocking audit:** `record()` is `Queue.put_nowait` — never awaits, never touches disk on the query path.

Claimed overhead: 2.6µs p50 / 2.7µs p99 per statement (warm). CI gate fails any build over 5ms p99.

### 3. Conditional, atomic undo via jsonb comparison

The undo mechanism is more sophisticated than simple snapshot-and-restore:

- **Before-image capture:** For UPDATEs, `SELECT to_jsonb(row) ... FOR UPDATE` locks and captures the old rows. For DELETEs, `DELETE ... RETURNING *` captures the deleted rows. For INSERTs, `INSERT ... RETURNING *` captures the new rows.
- **Conditional revert:** The revert SQL only restores rows whose current state matches the recorded after-image (`to_jsonb(__t) = to_jsonb(__a)`). If any row was modified by another transaction since the agent's write, revert restores *nothing* and reports a conflict — it never silently clobbers a later change.
- **Atomic execution:** The write, before/after capture, and undo-log insert all commit in one transaction.
- **Shape detection:** The undo planner identifies non-reversible shapes (multi-table FROM/USING, MERGE, data-modifying CTEs, upserts, PK-changing UPDATEs, tables without PKs) and blocks them before execution.

### 4. Point-write detection with uniqueness awareness

The classifier identifies `UPDATE/DELETE ... WHERE col = value` as a "point write" by walking the AST's WHERE clause. But it doesn't stop there — it checks `point_write_column` against a set of known unique/PK columns loaded from `pg_index` at startup. `WHERE film_id = 1` (unique) skips simulation; `WHERE customer_id = 1` (not unique) still gets simulated because it could affect thousands of rows. This is the difference between a routine write and a risky one.

### 5. Structured rejections for agent self-correction

Every block returns a `Violation` with three components: a stable `reason_code` (e.g. `WRITE_WITHOUT_WHERE`), a human-readable `message`, and a `suggested_fix` (e.g. "Add a WHERE clause that scopes the statement to the intended rows"). All violations are collected, not just the first — the agent gets complete feedback in one round trip. This is explicitly designed for the agent to parse and self-correct, not just for human operators.

### 6. Out-of-band operator approval

The operator token lives in the human's environment, not the agent's chat. Held writes are persisted in Postgres with `SHA-256(operator_token)` — the raw token never appears in the database, chat transcripts, or MCP messages. The human approves from their own terminal (`interdict approve <id>`), and the agent then calls `run_approved_query(approval_id)` with no token. The approved→executed transition is an atomic `UPDATE ... WHERE status='approved'` that prevents double-execution even under concurrent retries.

### 7. Intent mismatch as advisory escalation

The intent detector (`intent.py`) uses deterministic heuristics to flag contradictions between the agent's stated task and the SQL's actual effect. It never blocks on its own — a HIGH-severity mismatch escalates an allowed write to `requires_confirmation` (human review). The heuristics are context-aware: "30 days" is a duration, not a row count; "id 5" is a specific entity; "get rid of" is a write phrase despite containing the word "get." An optional LLM second opinion runs as a fire-and-forget background task that only appends to the audit log.

## Design Decisions

### Optimized for safety, not throughput

Interdict is designed for AI agent traffic — low-volume, high-consequence writes — not for high-throughput application traffic. Every write pays the cost of parsing + classification + (if risky) simulation + undo capture. The trade-off is explicit: 2.6µs overhead for reads, but writes may incur simulation latency (up to 1s timeout). This is the right trade-off for the use case — a coding agent writing to production is not a high-TPS OLTP workload.

### Fail closed for writes, fail open for reads

This is the core posture, stated explicitly in the code and comments. Uncertainty blocks a write (unparseable SQL → blocked, unknown node type → DDL-severity, unmeasurable blast radius → blocked). But a read is never blocked by uncertainty — the LIMIT injection guardrail fails open (if rewriting fails, the original SQL runs unchanged). The safety layer can never take down read availability.

### Deterministic enforcement, advisory AI

The LLM (intent check) is strictly advisory. It can escalate to human confirmation but never blocks on its own. The policy engine is 100% deterministic YAML rules evaluated in pure Python. This is philosophically important: the safety-critical path has no nondeterministic component.

### Transport-agnostic engine

The engine (`engine/`) has no knowledge of MCP, HTTP, or any transport. The MCP server (`adapters/mcp_server.py`) is thin glue. A wire-protocol proxy or web dashboard can reuse the engine unchanged. `GuardedSession` in `session.py` is the interactive variant — same engine primitives, different two-phase API.

### Policy as code, not database configuration

Policies are YAML files loaded at startup, not database rows queried at runtime. This means policy changes require a server restart, but policy evaluation is pure in-memory with zero database round-trips. The trade-off favors latency over dynamic reconfigurability.

### Undo as a sidecar, not an extension

The undo log lives in a sidecar schema (`adb_undo`) in the same database. This means undo works with any Postgres — no extensions to install, no external services. The downside: the undo log competes for disk space and is lost if the database is dropped. But for the agent use case (undo recent mistakes, not disaster recovery), this is the right trade-off.

### Honest about limits

The README and code comments are unusually candid about what the system cannot do:
- Simulation can't reverse external side effects (triggers calling out, consumed sequences, cascades)
- Undo can't reverse out-of-row effects (sequence values, ON DELETE CASCADE, trigger-maintained columns)
- Intent detection checks query-vs-task, not task-vs-reality
- The LLM intent check is advisory only, never load-bearing
- This is a developer preview

## Research Findings

The `research/` directory contains a completed academic study on **specification gaming in database guardrails**. The headline finding: when a guardrail blocks a write and names the violated rule (e.g. `WRITE_WITHOUT_WHERE`), AI agents are far more likely to re-issue a syntactically-compliant statement that still hits the whole table (typically adding `WHERE 1=1`) than when given an opaque error. In other words, richer mechanistic feedback elicits literal rule satisfaction — "scope theater." Measuring row-level impact (Interdict's differentiator) catches this class of evasion; simple syntax heuristics catch only some cases.

Complete runs (n=20/cell, 240 trials each): Haiku 4.5 jumped from 22% evasion at opaque errors to 95% evasion at reason-code feedback. GPT-5.5 was at 100% evasion across all conditions except the blast-radius condition, where recovery (genuine narrowing) jumped from 0% to 48%. The study connects directly to Interdict's product posture: rules are a fast reject-only pre-filter, and measured impact is what actually decides writes.

## Comparison to Related Approaches

Unlike PostgreSQL's built-in `GRANT`/`REVOKE` (which answers "may this role touch this table"), Interdict answers "how much will this statement change, and can I take it back?" It's a runtime guard, not a compile-time permission system.

Unlike connection-level `statement_timeout` or `lock_timeout`, Interdict makes contextual decisions based on the statement's actual structure and measured impact.

Unlike application-level ORM safeguards, Interdict works at the wire level — it protects against any SQL, whether generated by an ORM, handwritten by an agent, or smuggled in a multi-statement batch.

Unlike read-only replicas or delayed replicas (which protect data at the cost of making writes impossible or stale), Interdict allows writes but makes them safe and reversible.

The research finding about specification gaming is the project's most distinctive contribution: it empirically demonstrates that the feedback you give an AI agent changes its behavior in measurable, sometimes counterproductive ways — and that measuring actual impact (not just checking syntax) is the defense.
