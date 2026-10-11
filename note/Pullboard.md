# Pullboard

Pullboard is an MIT-licensed npm CLI (v0.8.6, ~21k lines of dependency-free Node 22 ESM) that turns a git repo into a coordination queue for a hierarchy of coding agents: the human rules from a spec, one coordinator agent plans and merges, builders work in isolated worktree lanes, and a verifier that built none of the work must accept the exact submitted commit before it counts. It is deliberately local-first — the whole board is one SQLite file inside `.git`, nothing is hosted, and nothing ever calls a model.

---

## Architecture

- **Board in the git common dir** (`src/board.js`, 2600 lines): a single SQLite database (Node's built-in `node:sqlite`, zero dependencies) in `.git`, shared by every worktree and never committed. Every move is one immediate transaction, so two agents can never claim the same item; every move lands in an append-only event log.
- **The lifecycle is declared once** (`src/machine.js`): states (open → claimed → submitted → verified/withdrawn), moves, roles (agent / coordinator / clock), and ~25 named guards each with a typed refusal code and a repair step. Final states carry exit guards *every* path into them must name — the verified state has an SQL proof condition requiring an ACCEPT verdict at the submitted commit, on the frozen criterion digest, by an agent who did not build it. `machineProblems` checks the declaration's properties, and the tests demonstrate each property failing on a deliberately broken copy.
- **Lanes as worktrees** (`src/lanes.js`, `src/hooks.js`): each lane owns paths and spec sections; pre-commit/pre-merge hooks refuse commits outside a lane's folders. `pullboard.json` names the gate (`npm test`, `pytest`, …) and the lanes.
- **Gate as a shared resource** (`src/gate.js`): the check command runs over a committed tree with nothing untracked and nothing changed mid-run; a green run leaves a stamp in the git dir so the same tree is never checked twice. Agents see a 3000-char *digest* of failures, not the log — passing output "says nothing" and costs tokens.
- **Hooks** (`src/hooks.js`): commit messages must cite the spec row (`feat(web): pay by card [G1]`); secret-scanning blocks ~12 key patterns in pre-commit; pre-push re-checks the gate so `--no-verify` locally still can't push unproven code.
- **The coordinator/verifier split**: `pullboard worktree` prints a subagent's opening instructions; skills ship for Claude Code (spec-review, verify, decompose, plan, run, signoff) and `pullboard prompt <role>` for Codex and others. `pullboard view` renders a local board of every project on the machine with a Needs-you row, timeline and shouts; a "relay" layer (multiple `src/relay-*.js` files) with signed/sealed records is planned to push the board to a phone.

## Key techniques

- **Frozen criterion digests.** When an agent claims an item, the spec rows it cites are hashed (`item_frozen_digest`); any later edit to `SPEC.md` invalidates the claim until the coordinator re-freezes. This kills the whole class of "the spec moved under me" failures.
- **Refusals as UX.** Every blocked move returns a typed code plus the exact next step ("cd <lane worktree> && pullboard claim <id>") — the error message *is* the instruction, tuned for agents rather than humans.
- **Model-tier routing.** Items route light/mid/strong; an agent's route set at join time means a small local model can take light items (`pullboard run` for unattended builds) but a frontier item only lands on a capable model. Cost allocation is a first-class scheduling input.
- **Token-aware output shaping.** Gate digests, brief limits (8k chars), and shout channels all cap what agents read; the system treats context budget as a resource to be rationed by the harness, not the agent.
- **Sealed, replayable records.** An `ENGINE_VERSION` rides every sealed move; verdicts and events are append-only; `view --export` and `pullboard export` write replayable snapshots. Receipts (reject reason → resubmit → accept with evidence) are the durable artifact of a review.

## Design decisions

- **SQLite in `.git` instead of a service.** No daemon, no account, no sync — but the board is per-machine, and the README is candid that worktree identity "is not a security boundary." They chose ubiquity and zero-ops over fleet features.
- **Enforce what a machine can check; reserve judgement for the person.** Doctrine rules each name an enforcer (hook, gate, or review); spec rows stay drafts until the human approves. The hierarchy is structural, not aspirational: only coordinator-flagged decisions reach the human at all.
- **Verification by a *different* agent at the *same* commit**, not by the builder or later in CI. The verifier must checkout the pinned commit and explain the accept or reject; the SQL proof in the machine makes skipping structurally impossible rather than a policy suggestion.
- **Complexity cost is real**: 21k lines, a formal machine declaration, machineProblems property tests, and typed refusal codes — a lot of machinery for a queue. The bet is that the failure modes it prevents ("it said done, it wasn't"; two agents, one file) cost more than the machinery.

## Comparison notes

- [[vibe-kanban]] is the closest cousin: both are local work queues that fan coding agents out across git worktrees. Pullboard goes further on proof — frozen criteria, a different-agent verifier, receipts, and a machine-declared lifecycle — while vibe-kanban stays a lighter dispatcher.
- [[The Harness Is the Company]] argues the durable asset is the harness around the model; Pullboard is a concrete instance: the moat here is the state machine, the refusals and the receipts, not any model choice.
- [[When Spec-Driven Development Pays Off]] describes spec-first as planning discipline; Pullboard operationalises it — approved rows freeze into per-item criteria, and the gate field on each row ties requirement to proof.
- [[llm-as-a-verifier]] collects the case for a second model checking the first; Pullboard makes that second check structural (builder ≠ verifier, same commit, evidence note) instead of advisory.
- Compared to [[Queues Don't Fix Overload]], Pullboard's queue is about *correctness of handoff*, not throughput — its lanes and held lanes are more back-pressure than scheduling optimization.

---
*Sources: [[raw/pullboard]], [[summary/pullboard]]*
*Last updated: 2026-10-11*
