# Mole

**Mole is a deep-research agent in Go that enforces its own cost ceiling in the database schema, verifies every extracted claim against a verbatim quote from its source before it can reach the answer, and gates analysis of local CSV/folder data behind a k-anonymity aggregation boundary.** Where most "deep research" tools wrap a tool-calling loop in Python and trust the model to stop when asked, mole replaces trust with structure: a reserve-before-dispatch ledger, deterministic quote checks, graph-derived confidence, and a SQL-parser-plus-floor gate. It runs as two static binaries (`mole`, `mole-mcp`) on your own machine with your own keys, and speaks MCP so a coding agent can drive it — either by handing it a question, or in **toolkit mode** where the agent's model does the reasoning and mole contributes the deterministic half that doesn't care whose model is on the other side.

---

## Architecture

**A planner → executor → verifier → output pipeline, not a tool-calling loop.** The README's diagram is literal: a question is decomposed by the planner, leads run one-per-worker under the executor, actors search→fetch→extract→mine claims, the verifier builds a claim graph and adjudicates contradictions, and output synthesizes prose from the surviving claims. The design is a *pipeline of role-separated stages* where each stage's spend is attributed to a `Role` enum (`planner`/`executor`/`verifier`/`output`) so cost breakdowns are a `GROUP BY`, not archaeology (`internal/core/core.go`, `internal/store/sqlite/migrations/0001_init.sql`).

The code is ~30 packages under `internal/`, each with a single job: `budget/` (the ledger), `planner/`, `executor/`, `actors/` (web/academic/local), `verifier/`, `output/`, `compute/` (the local-data boundary: `connector` → `hypothesis` → `sqlguard` → `gate`), `store/` + `store/sqlite` (pure-Go SQLite), `llm/` (the provider boundary), `daemon/`, `mcpserver/`, `session/`.

**Two callers share one runner.** `internal/session/session.go` splits the lifecycle into `Recover`/`Create`/`Run` so a CLI and a long-lived daemon reuse the same loop: the daemon creates a session row and returns its id before work finishes (a stdio MCP subprocess dies when the coding agent's session closes, so a unix-socket daemon holds the state and a disposable `mole-mcp` shim forwards to it). `internal/daemon/daemon.go` listens on a mode-0600 unix socket and `peer_linux.go` reads `SO_PEERCRED` to refuse connections from other users — credentials stay daemon-side, never in `.mcp.json`.

**The web actor is a straight-line pipeline with conditional fetch.** `internal/actors/web.go` runs search → (fetch → extract)? → chunk → mine → verify quotes → reduce. The fetch is *skipped* when the search provider already returned usable page content — an efficiency that also collapses a whole class of "js_required" failures. The actor is the only component that touches raw source material; everything raw dies when `Run` returns. A per-session *copy* of the actor is taken (`actor := *r.Actor` in `session.go`), so concurrent sessions can't cross-attribute claims while connection pools and rate limiters stay shared.

## Key Techniques

**Budget is reserved before dispatch and settled after, in integer micro-dollars.** The central idea of `internal/budget/ledger.go`: post-hoc charging lets a pool of N workers overshoot by N lead-costs, because each passes the "can I afford one more?" check before any has paid. Reserving first bounds overshoot to estimate error on a single lead. Money is `int64` micro-dollars (`core.USDMicros`) because float accumulation drifts over thousands of operations. The availability check and the hold happen in one transaction, and the SQLite schema enforces `CHECK (spent >= 0)`, `CHECK (held >= 0)`, `CHECK (escrow >= 0)` — the last line of defense is the database refusing to write a negative balance, not code (`migrations/0001_init.sql`). An append-only `tool_calls` ledger is the source of truth; `Session.Spent` is a materialized sum written in the same transaction, and `Ledger.Verify` recomputes it to prove consistency.

**Escrow holds back the report.** 15% of the budget is reserved at session creation for output generation and the final verification pass — otherwise a session spends 100% on research and has nothing left to write the answer (`Config.EscrowFraction`, released only in `session.finish`).

**A claim is only accepted if its quote appears verbatim in the source.** `internal/actors/quote.go`'s `FindQuote` is "the single most valuable check relative to its cost": deterministic, zero model calls, and it catches the dominant failure mode — a fabricated citation attached to a plausible sentence. A quote shorter than 24 collapsed-space characters is rejected (a three-word quote matches almost any document by chance). `Mine` (`internal/actors/mine.go`) discards a claim whose quote doesn't match *before it reaches the store* — it's not fixed downstream, it never exists.

**Confidence is derived from graph structure, never asked of the model.** `internal/verifier/confidence.go` computes confidence from independent publishers (counted by eTLD+1 via the public suffix list, and by *paper* for scholarly sources — DOI registrant, pmid, pmc, arxiv), a hand-written source-class table (peer-reviewed > primary > secondary > unknown, defaulting *weak* so unrecognized sources understate rather than invent confidence), contradicting edges, supersession, and grounding. Self-reported confidence is rejected as uncalibrated ("mostly encoding fluency"). Union-find clusters duplicate claims; corroboration saturates (a scraper farm can't drive confidence to certainty).

**Contradiction adjudication is confirmed twice.** A single model judgement gets "contradicts" right 51% of the time; a second agreeing judgement raises precision to 70% — at the cost of keeping 53 edges where one pass kept 105. Mole takes that trade deliberately (`verifier.confirm`), because a false contradiction spends money researching a disagreement that doesn't exist.

**The planner sees counts, never page text.** The digest carries sub-question wording, claim counts, dead-end causes (a fixed enum) and budget-remaining — never page-derived text. The planner comment calls this "stronger than fencing it: there is nothing to fence," and it closes the prompt-injection path completely, at the cost that a content-aware replan can't suggest a new angle.

**The local-data boundary is layered.** `compute/sqlguard` parses the statement (only a single aggregate SELECT passes), `compute/hypothesis` lets the model pick a template + column names that are *looked up* in the connector profile rather than escaped into SQL (lookup makes it impossible to interpolate anything not already a column), and `compute/gate` applies a k-anonymity floor (buckets under 5 records are folded into an unnamed "other"), withholds free-text columns, and runs pairwise statistical tests with Holm correction. `exfil.go`'s `verifyNoLeak` is an *invariant*, not a metric: before an envelope is returned it's checked against the rows it came from, and one carrying a value it shouldn't is withheld. `mole crossings` shows exactly what left.

**Citations are assigned mechanically, not written by the model.** `output/report.go` numbers sources before the prompt is built, so "the model cannot invent a citation: every [n] it can legitimately write already maps to a real source." `validateBody` rejects a synthesized body that carries a forged citation, and `stripControls` strips terminal-control and bidi-override characters at render time so a hostile page can't rewrite what the reader sees.

**Chunking exists to keep quotes checkable.** `llm/chunk.go` splits into contiguous byte-offset slices, preferring paragraph then sentence then whitespace boundaries (with CJK terminators handled), because a chunk that paraphrased or reordered text would make grounded claims unverifiable.

## Design Decisions

- **Two model tiers.** `cheap` (Haiku) for per-chunk extraction and adjudication; `strong` (Opus) for planning and synthesis — because chunk mining runs once per chunk while planning runs once per lead, and "using one model for both either overpays on the many calls or underperforms on the few." A reasoning model that spends its whole output allowance on chain-of-thought and returns empty content is a named error (`llm.ErrEmptyOutput`), not a mystery parse failure.
- **Honesty over polish.** `mole eval <session-id>` grades mole's own runs and prints a scorecard where any metric it can't compute *says so* instead of reading zero — including 100% claim-integrity/citation-accuracy and an honest 70% contradiction precision with the confirm pass. The confidence model explicitly refuses a fifth "recency" term because nothing can measure it yet.
- **"Falsify your own fix."** The contributing rule: after a change, revert the mechanism and confirm the test fails — several of the project's own tests were caught passing with the fix removed.
- **Crash-recoverable by design.** Leads are *leased* (not marked running), heartbeated at a third of the TTL, and swept at boot; reservations expire; abandoned sessions are marked. The `Recover` path runs per-process at daemon startup.
- **The toolkit-mode trade.** In toolkit mode the prompt-injection fence "stops being a guarantee and becomes a convention" — the fetched text lands in a prompt mole doesn't assemble. That's the explicit cost of using the subscription you already pay for.

## Comparison Notes

- **vs. [[Local Deep Research]]** — the closest sibling. Both are local deep-research agents, but they optimize opposite things. LDR is a Flask monolith with ~25 pluggable search engines, per-user encrypted SQLite, and a DLP egress guardrail; mole is a Go pipeline whose "guardrail" is a budget *ledger* and whose privacy boundary is a *deterministic* aggregation gate. LDR's egress module calls itself "an in-process correctness guardrail, NOT a hard security boundary"; mole's equivalent (`compute/gate` + `sqlguard`) refuses malformed input structurally and its exfil check is an invariant run before anything crosses. LDR chases benchmark coverage (95% SimpleQA); mole chases a 0% budget-overshoot number it can verify.
- **vs. [[Agent Orchestration]]** — mole is another confirmation of the planner/worker/judge finding, but the judge is not a peer agent: verification is a *stage* with its own budget share, and its confidence is *computed*, not judged. The planner/executor/verifier separation exists so each role's spend is attributable and each stage can be bounded independently.
- **vs. [[Guardrails and Feedback Loops]]** — mole is the thesis applied to research: the quote check and the ledger are *structural* gates (deterministic, no model), where a "be careful with citations" prompt would be the behavioural one. The claim "every stored claim carries a source and a verbatim quote" is enforced by `FindQuote` and the SQL CHECK constraints, not by instruction.
- **vs. [[Citations for Accurate Long Form Content]]** — Ian's insight was citations as breadcrumbs *for subagents to verify*. Mole mechanizes that verification: the quote is checked by string matching before the claim exists, so the verifier never re-reads the page unless grounding explicitly pays to. The breadcrumb is checked, not just left for a later agent.

## Tags

#tool #project #agents #research #verification #budget #security #mcp #go

---
*Sources: [[raw/mole]], [[summary/mole]]*
*Last updated: 2026-08-22*
