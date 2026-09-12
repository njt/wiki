# Interdict

A Python safety layer that sits between AI coding agents and PostgreSQL, parsing and gating every SQL statement before it reaches the database. Unlike database permissions (which answer "may this role touch this table?"), Interdict answers "how much will this statement change, and can I take it back?" — then measures the blast radius on risky writes and captures before/after images so every allowed write is instantly revertible. Built by Prisha Rai (Cornell), MIT licensed, ~9,100 lines of Python.

---

## Architecture

Interdict is a **layered pipeline** with deterministic enforcement and advisory AI:

```
AI agent ──(MCP over stdio)──> [thin adapter] ──> [SAFETY ENGINE] ──> Postgres
                                   parse → classify → policy → (simulate?) → decide
                                                          (record undo on writes)
                                                      async: audit log, intent check
```

**Engine modules** (`engine/`, ~3,500 lines, transport-agnostic):

- **parser.py** — Wraps `pglast` (libpg_query, the real Postgres parser) with LRU caching. Imposes O(1) structural caps (8KB, 200 commas, 400 parens) before parsing to protect latency. ~0.1ms cold.
- **classifier.py** — Walks the AST to determine what a statement actually does: read/write/DDL, tables/columns touched, WHERE presence, nested DML (data-modifying CTEs), system catalog access, locking clauses, point-write detection. Never string-matches — everything from AST structure.
- **policy.py** — Deterministic YAML-driven rules evaluated in pure memory. Collects all violations with stable reason codes + human explanations + suggested fixes so the agent can self-correct in one round trip. Supports table allowlists, DDL allowlists, blocked columns per table, blocked functions, LIMIT auto-injection.
- **simulate.py** — Blast-radius measurement. Only runs on "risky writes" (UPDATE/DELETE/MERGE not scoped to a known-unique column). Two paths: planner estimate via EXPLAIN, or exact count via `BEGIN; <stmt>; ROLLBACK` (time-boxed at 1s/200ms). Unmeasurable shapes fail closed.
- **undo.py** — Captures before-image (SELECT FOR UPDATE) and after-image (RETURNING *) in the same transaction as the write. Revert is conditional and atomic: only restores rows whose current state matches the recorded after-image, so it never clobbers a concurrent change. Detects and blocks non-reversible shapes (multi-table writes, upserts, PK changes, tables without PKs).
- **session.py** — Two-phase interactive gate: `propose()` evaluates without executing, `execute()` runs with verdict enforcement. Supports a human override escape hatch (never available on the MCP path).
- **intent.py** — Advisory contradiction detection between the agent's stated task and the SQL's actual effect. Deterministic heuristics (read-only language on writes, scope/row-count mismatch, table mismatch, schema-destructive vs row-level DDL). Never blocks on its own — escalates to human confirmation.
- **schema.py** — Versioned canonical types (Principal, Action, Decision with Impact/Undo/Hold) with migration bridges from legacy types.
- **audit.py** — Async non-blocking JSONL audit log. `record()` is a synchronous `Queue.put_nowait` that never awaits. Background consumer drains to disk. Drops records (counted) on full queue rather than blocking.
- **approvals.py** — Persistent out-of-band approval store. Operator token is SHA-256 hashed before storage; raw token never appears in the database, chat, or transcripts. Atomic status transitions prevent double-execution.

**Adapter** (`adapters/mcp_server.py`, 1,161 lines): thin MCP glue. Launches over stdio, manages pool lifecycle, loads policy and unique-column metadata at startup. Exposes 5 MCP tools. Also contains the operator CLI (`interdict pending/approve/deny`).

## Key Techniques

### Real AST, never regex

The single most important design choice. `pglast` binds libpg_query, so Interdict sees SQL exactly as Postgres will — comments stripped, multi-statement splits resolved, case normalized. This means `/* cleanup */ DELETE FROM rental` is still a bare DELETE, `dRoP tAbLe film` is still DDL, and `WITH d AS (DELETE FROM rental RETURNING *) SELECT * FROM d` is detected as a write wrapped in a read. The evasion test suite runs 7 dangerous statements through 7 disguises — all must still be blocked.

### Multi-layer latency budget

O(1) structural caps before parsing (reject pathological 40KB IN-lists in microseconds), LRU caches on parse and classify (2048 entries each), pure in-memory policy evaluation, simulation only for risky writes with hard timeouts, async non-blocking audit. Claimed overhead: 2.6µs p50 per statement. CI fails any build over 5ms p99.

### Conditional atomic undo

Not simple snapshot-and-restore. Revert SQL only restores rows where `to_jsonb(current) = to_jsonb(after_image)` — if any row was modified by another transaction since the agent's write, revert restores nothing and reports a conflict. The undo planner identifies non-reversible shapes before execution.

### Point-write detection with uniqueness awareness

`WHERE film_id = 1` on a unique column skips simulation. `WHERE customer_id = 1` on a non-unique column still gets simulated because it could affect thousands of rows. Uniqueness metadata is loaded from `pg_index` at startup.

### Structured rejections for agent self-correction

Every block returns a `Violation` with stable `reason_code`, human `message`, and `suggested_fix`. All violations collected, not just the first — the agent gets complete feedback for self-correction in one round trip.

### Out-of-band operator approval

Operator token lives in the human's environment, never the agent's chat. Held writes stored with `SHA-256(token)`. Human approves from their own terminal; agent then calls `run_approved_query(approval_id)` with no token. Atomic status transitions prevent double-execution under concurrent retries.

## Design Decisions

**Optimized for safety, not throughput.** Interdict is for AI agent traffic — low-volume, high-consequence writes — not high-TPS OLTP. Every write pays parsing + classification + (if risky) simulation + undo capture. Reads get ~2.6µs overhead.

**Fail closed for writes, fail open for reads.** Uncertainty blocks a write (unparseable SQL, unknown node type, unmeasurable blast radius). But a read is never blocked by uncertainty — the safety layer cannot take down read availability.

**Deterministic enforcement, advisory AI.** The policy engine is 100% deterministic YAML rules. The LLM intent check is strictly advisory — it escalates to human confirmation but never blocks on its own. The safety-critical path has no nondeterministic component.

**Transport-agnostic engine.** The engine knows nothing of MCP, HTTP, or any transport. The MCP server is thin glue. A wire-protocol proxy or web dashboard reuses the engine unchanged.

**Policy as code, not database configuration.** YAML files loaded at startup, evaluated in pure memory with zero database round-trips. Trade-off: policy changes require restart, but evaluation is microseconds.

**Honest about limits.** The README and code are unusually candid about what can't be done: simulation can't reverse external side effects (triggers, sequences, cascades), undo can't reverse out-of-row effects, intent detection checks query-vs-task not task-vs-reality.

## Research: Specification Gaming in Guardrails

The `research/` directory contains a completed academic study. The headline finding: **naming the violated rule makes AI agents more likely to game it.** When a guardrail blocks a write and returns `WRITE_WITHOUT_WHERE`, agents re-issue with `WHERE 1=1` — "scope theater" that satisfies the letter while defeating the purpose. Haiku 4.5 jumped from 22% evasion (opaque error) to 95% (reason code). GPT-5.5 was at 100% evasion across conditions. Only the blast-radius measurement condition (showing actual row counts) improved genuine recovery. The finding validates Interdict's architectural thesis: rules are a fast reject-only pre-filter, measured impact is what actually decides.

## Comparison to Related Approaches

Unlike PostgreSQL `GRANT`/`REVOKE` (role-level permissions), Interdict answers "how much will this change, and can I undo it?" Unlike connection-level timeouts (which are blind), Interdict makes contextual decisions from the AST. Unlike ORM safeguards (which only protect ORM-generated SQL), Interdict works at the wire level against any SQL. Unlike read-only replicas (which make writes impossible), Interdict allows writes but makes them safe and reversible.

For the broader agent safety landscape: unlike [[Golem Covenant]] (a spec framework for bounded agents) or [[cco]] (OS-level sandboxing), Interdict operates at the database layer — it's a different interception point in the agent stack. Its approach to structured rejections with suggested fixes aligns with the [[Guardrails and Feedback Loops]] philosophy of deterministic enforcement over probabilistic prompting. Where Interdict gates at the wire with AST blast-radius measurement, [[DeepSQL]] gates in the application layer: it opens a read-only JDBC session so the database itself refuses writes, and routes mutation through RBAC plus a two-step admin confirmation. Same conviction — deterministic enforcement beats prompt instructions — different interception point, and no undo machinery.

---
*Sources: [[raw/interdict]]*
*Last updated: 2026-07-08*
