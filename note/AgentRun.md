# AgentRun

AgentRun is a TypeScript workflow DSL from Parcha Labs (v0.1.0-beta.4) that turns agent behavior into inspectable, rerunnable documents embedded in your own application. Its thesis: a fixed sequence of steps doesn't need a free-running agent — define the steps in a validated workflow document, spend Jev typed decisions on cheap judgments, and call the agent only where investigation is genuinely needed. The host keeps its tools, model access, permissions, and budgets.

---

## Architecture

Three npm packages in a workspace monorepo:

- **`@parcha/agentrun-dsl`** (`packages/dsl/src/`) — the engine. `workflow.ts` (2,254 lines) holds the node types, validator, and interpreter; `system-one.ts` compiles decision schemas into typed question sets; `vocabulary.ts` is the closed language vocabulary; `author.ts` is the Zod-based TypeScript builder.
- **`@parcha/agentrun-jev`** (`packages/jev/src/index.ts`) — the adapter to TypeSafe's Jev System One API, with an injectable `JevClient` boundary, retry with backoff, and a `JevError` class that deliberately excludes provider bodies and input state from diagnostics.
- **`@parcha/agentrun-pi`** (`packages/pi/src/`) — a Pi extension (~2,900 lines) with `/agentrun` commands (demo, run, status, graph, save/load), a workflow inspector, and a packaged skill so a coding agent can build workflows inside the editor.

The core abstraction is the **workflow document**: JSON with 17 node kinds (`chain`, `code`, `agent`, `decide`, `extract`, `report`, `artifact`, `map`, `parallel`, `loop`, `escalate`, `call`, `workflow`, `judge`, `pick`, `sift`, `route`). Nodes split into two families: *generative* kinds (one adapter session each, answering against a schema) and *judgment* kinds (one judge request each, no tools, no session). The engine is dependency-injected through `WorkflowDeps`: `runNode` (your agent), `runEffect` (your tools), `runJudge` (Jev), plus checkpoint/recovery/memo hooks.

## Key techniques

- **Typed decisions as compiled questions.** A `judge` node's `out` schema must be a flat object of string enums (choice), booleans (yes/no), and integers with level descriptions (score, 2–10 levels). `compileQuestions()` in `system-one.ts` rejects anything else — "prose and lists belong to a decide node" — so each judgment is one request returning per-answer confidences, stored at `<as>$answers` alongside the schema-valid value. A `sift` node asks the same question set of every list item in *one* request (item i's questions prefixed `i.`) and filters by a confidence threshold.
- **Host-owned policy as a capability boundary.** `HostPolicy` hooks (`systemBlocks`, `submissionSchema`, `decodeSubmission`, `afterNode`) are host code handed in through deps; "nothing in a workflow document can name or reach it, so a candidate workflow cannot change its host's channels." Host state lives under a reserved `$host` key no workflow may write, is checkpointed with the rest of state, and parallel branches merge `$host` deltas key-by-key with array-append semantics and write-conflict detection.
- **Verification as a submit-time reviewer.** Every generative node takes a `verify` clause compiled into a reviewer handed to the session's submit tool; a rejected verdict continues the *same* session with the message as the next tool result, bounded by `maxDrives`. The engine re-verifies returned submissions even if the adapter ignores the hook.
- **Escalation as a terminal state.** `escalate` nodes with predicates (mechanical ones like `gte`/`count_gte`/`no_new_items`, plus `ask` judge questions) end the run with `status: "escalated"` — the support demo exits with code 2. Human review is a workflow outcome, not an error path.
- **Recovery plumbing.** Idempotency-keyed effect memos (polled calls deliberately never memoized — "their result is a moment in time"), checkpoint hooks with a `required` failure mode, and retry classes (`timeout`, `http_5xx`, `http_429`, `connection`, `exit`) where `exit` retries only if the author declared it.
- **A Lean formal model.** `spec/lean/` models JSON values, syntax, semantics, validation and desugaring in core Lean, with theorems that quantify over *oracle functions* for everything delegated (adapters, schema checks, code execution). A shared conformance corpus in `spec/lean/conformance/` runs against both the Lean model and the TypeScript interpreter. Notably honest framing: "the implementation is not formally verified by these proofs."

## Design decisions

The big bet is **documents over live agents**: ordinary functions could handle a fixed sequence, but a workflow document can be inspected, hashed, rerun, and invoked as a tool by an agent. The cost is a closed vocabulary — the validator admits exactly the 17 kinds and their fields, and any other key is an authoring error. That rigidity buys the conformance story and cross-host portability.

Second, **model-tier routing as a first-class concern**: every generative node takes a `tier` (`fast`/`default`/`strong`) and `effort`/`thinking` dials — with the comment that "there is deliberately no off: thinking is never off." The README's framing: a router that sends the few thin cases to `strong` buys depth only where the decision is on the edge.

Third, honest limits, stated in the README itself: `code` nodes execute JavaScript with process privileges (untrusted authors need a host sandbox), and typed decisions with validated shapes "do not prove that an answer is factually correct."

## Comparison notes

- [[Apache Burr]] also models agents as explicit, inspectable structures (Python decorators as state machines with replay), but AgentRun goes further: a whole validated document language with a formal model, and judgment vs. generative as separate node families rather than a single loop.
- [[System One Models and Jev]] describes the model AgentRun builds on; this repo shows the consuming side — how Jev decisions are embedded in workflows with confidence thresholds, `sift` keep-filters, and `unsure` branches on `route` nodes. [[Jev Can't Be Calibrated]]'s caution applies directly: AgentRun's thresholds (e.g. `gte: 0.8`) are host-chosen gates on scores whose calibration is distribution-dependent.
- [[Building Reliable Agentic AI Systems]] argues production agents need structured context engineering and reflection loops; AgentRun is the same philosophy as an embeddable product — the `verify` clause is a bounded reflection loop, and the review escalation is the human gate that PRINCE's three loops imply.
- [[The Agentic Product Standard v2.0]] prescribes an autonomy ladder and eval pyramid; AgentRun operationalizes a slice of it — the escalation path and scripted fixtures/evals (`npm run eval:research` over labeled cases) are exactly the review-gate and eval discipline that standard calls for.

---
*Sources: [[raw/agentrun]], [[summary/agentrun]]*
*Last updated: 2026-09-25*
