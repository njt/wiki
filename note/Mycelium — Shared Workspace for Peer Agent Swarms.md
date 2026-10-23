# Mycelium — Shared Workspace for Peer Agent Swarms

Mycelium (mycelium-io, Apache-2.0, Python 3.12) is coordination infrastructure for autonomous agents operating as peers with no orchestrator and no hierarchy: one room per mission holds a shared memory store, an encrypted group channel, and a kanban-style **board** whose every row is a markdown task with its own conversation thread. Its bet is that the missing piece in multi-agent work isn't another orchestrator but *mediated negotiation* — an aligner engine that runs a real game-theoretic negotiation when agents disagree, so a swarm converges on one shared answer instead of producing parallel outputs a human reconciles.

---

## Architecture

**Hub-holds-everything, thin clients everywhere else.** The repo splits into five components: `mycelium-cli/` (a Python CLI that is also the agents' hands), `fastapi-backend/` (an always-on FastAPI "moderator" hosting ~98 service modules), `mycelium-frontend/` (Next.js UI), `mycelium-desktop/` (a Tauri app that runs the hub locally), and `contracts/` — frozen JSON schemas (`board-vocabulary.json`, `slim-wire.json`, `mentions.json`) that pin the vocabulary shared across components.

- **Storage is markdown, not a database.** Rooms are folders at `~/.mycelium/rooms/{room}/` with namespaces `work/`, `decisions/`, `status/`, `context/`, `procedures/`, `log/`. A task row is a markdown file with YAML frontmatter; the frontend renders the board as a *projection* over that bag of fields (`mycelium-cli/src/mycelium/board/model.py`) — "every view is a projection over that bag, so a coordination surface and a typed view over a namespace are one mechanism."
- **The channel is AGNTCY SLIM**, an MLS-encrypted group-messaging transport where the node forwards only ciphertext. An "epistemic envelope" (Layer 9) rides on top; every task's thread is just a tagged, membership-scoped slice of the room's one channel, minted with the row.
- **Participation is two stateless CLI calls.** `mycelium await` long-polls until a message addresses the agent's handle (the backend holds membership via a presence lease plus a durable transcript cursor, so no tick is missed between turns); `mycelium respond` posts the reply. An agent stays a *resident* Claude Code session via `await --loop --exec`, which loops await → reason → respond — the loop *is* the wake, no daemon, no cold-spawn, so context survives between turns.
- **The aligner** (`fastapi-backend/app/services/aligner.py`, ~1,100 lines) runs in-process in the backend, dormant until `@`-summoned; the mediator's own LLM is a Pi agent with a per-episode `--session` file (`pi_session.py`) so it remembers the whole haggle across SAO rounds.

## Key techniques

- **Real mechanism-design over agent chatter.** `mediator.py` maps natural-language positions onto a NEGMAS Stacked Alternating Offers mechanism. NEGMAS is synchronous Python, so `mech.run()` executes on a worker thread while each negotiator bridges back to the async loop with `run_coroutine_threadsafe` to publish an `@`-addressed prompt and collect the agent's actual reply. NEGMAS owns proposer rotation and termination — the run stops *the instant* everyone accepts the standing offer.
- **Interpretation, not protocol.** Agents never emit structured negotiation markers; an LLM mediator reads their prose, restates its reading into the transcript ("recording @growth as counter → 35% tech") so a misread is visible and correctable in-band, and folds the reading into deterministic MPC/GAR/SCR consensus metrics. JSON extraction is deliberately robust (scans every `{` with `raw_decode` rather than a greedy regex).
- **Term-mismatch check as negotiation stage 0** — before any offer, the mediator looks for words two agents use in *different senses* ("done", "priority"), because an agreement built on ambiguous vocabulary settles nothing. Exactly one clarifying round, never a loop.
- **Custody is a clock-draining lease, separate from assignment.** `status` says what stage a row is at; `custody` is a lease renewed by a live agent, and a background sweep (`lease_sweep.py`) announces expiry *once per lease identity* `(room, key, claimed_at)` — and only for crossings observed while the hub was up. A board full of dead agents reads empty, not busy.
- **Memory search without a vector DB**: one JSONL file per room, brute-force cosine over fastembed BAAI/bge-small-en-v1.5 384-dim ONNX embeddings, deterministic UUIDs minted via `uuid5(room, key)`. Explicitly documented as "fine at personal scale; a real ANN index is a later problem."
- **Anti-theater bookkeeping**: an "already agreed" fast path settles immediately when all stated positions match, round-0 clarifying ticks are distinguished from NEGMAS's own rounds, and stop reasons ("same offer refused three times") are reported as human sentences.

## Design decisions

- **Coordination decided separately from execution.** Converging on a negotiation does *not* resolve the task and failing does not strip its custody: "the negotiation decides *what*; the rows are *how the team carries it out*." The `TASK_FIELDS` list marks which axes a thread fold must never write.
- **Cost is a first-class constraint.** The aligner is dormant by default (zero idle cost), engines are summoned explicitly, and the cheap backend does all the listening.
- **Auth is a ladder that starts off.** A shared-secret channel key works out of the box for a laptop; SignerJwt per-member credentials and an HTTP JWT gate are opt-in rungs. Honest about what's missing: revocation is the open piece before multi-user deployment.
- **Verdict broadcast has empty wire parents** — a subtle distributed-systems fix: since a group broadcast isn't echoed to its own sender, real causal parents would strand some receiver's CausalOrderBuffer holding the commit forever; the full chain stays in the episode record instead.

The honest weakness is that this is opinionated, young, single-hub software: the scale ceiling of brute-force search, markdown-as-database, and one in-process engine per backend are all fine for a laptop team and all questionable at fleet scale — which the project openly frames as out of scope.

## Comparison notes

- Unlike orchestrator-worker systems such as [[Orchestrator - Worker Skill]] or [[Multi-Agent Orchestration Across Repositories]], Mycelium refuses the supervisor role entirely — its own README says if you have an orchestrator you don't need it. Where those systems route tasks top-down, this one gives peers a board, a channel, and a referee for when they disagree.
- Against [[Multi-Agent Systems Have a Distributed Systems Problem]]'s warnings, Mycelium is a rare example of taking them seriously in the artifact itself: durable transcript cursors, lease-expiry sweeps with exactly-once semantics, and causal-parent handling in the commit path are all responses to real distributed-systems traps in agent chat.
- Its "custody drains by the clock" board differs from [[vibe-kanban]] and [[Managing Agents via Kanban Boards]], where assignment is a static field; deriving liveness from a renewing lease means staleness is visible without a heartbeat service.
- Its memory model — one hub-held store, agents write prose that later agents inherit — leans toward the "shared external memory" camp of [[How AI Agent Memory Works]] and [[Three Tier Memory]], but with the index treated as a disposable derived artifact rather than the system of record; closer in spirit to [[Zero-Mem]]'s minimalism than to persistent-recall designs like [[Devin Memory and Dreaming]].

#tool #project #agents #memory #multi-agent #negotiation

---
*Sources: [[raw/mycelium]], [[summary/mycelium]]*
*Last updated: 2026-10-24*
