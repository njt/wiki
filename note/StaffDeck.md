# StaffDeck

StaffDeck is OpenBMB's open-source (AGPL-3.0) platform for building and governing **enterprise "digital employees"** — persistent agent personas with job descriptions, capability profiles, knowledge bases, SOP skills and tool access, served through a web workbench, WeChat/Enterprise-WeChat channels, an open REST API and a CLI. Released July 2026, at v0.4.0 by August, it is the most complete Chinese-ecosystem answer to the "AI as enterprise workforce" thesis: not a coding agent but an agent *HR department* — hiring, provisioning, routing, observing, and retiring digital staff. Its ~97k-line FastAPI backend is a dense catalog of production patterns for long-running, stateful, multi-agent enterprise systems.

---

## Architecture

A Python/FastAPI monolith (`backend/app/`, ~97k LOC) plus a React/TypeScript workbench (`frontend-enterprise/`), served from a single port (`scripts/dev.py` lifecycle, `single_port_app.py`), with SQLModel/SQLite-class storage (`backend/app/db/`). Key layers:

- **Agent runtime** (`backend/app/core/`): `agent_loop.py` (4.2k-line chat turn pipeline) and `turn_coordinator.py` (2k lines) own the turn: routing decisions, capability manifests, session locks and leases (`harness_session_lease.py`), cancellation, human handoff, and streaming events.
- **SOP skills as state machines** (`backend/app/skills/`, `backend/src/staffdeck_harness/sop/`): skills are typed graphs — `SkillGraphNode` carries `instruction`, `condition`, `expected_user_info`, `allowed_actions`, optional flags; execution walks nodes, enforces required *slots* (`enforce_required_slots`), and supports `handoff` nodes found by BFS from the active step (`GraphRules.find_handoff_node_id`).
- **Knowledge as an OKF wiki** (`backend/app/knowledge/`): a staged ingest pipeline (parse → normalize → document → bucket → chunk → summarize → discover SOPs/tools) converts documents into typed *concepts* (`Source Document`, `Source Section`, `Topic`, `Playbook`, `Business Rule`, `Query Analysis`). Retrieval routes a query down the hierarchy (document → bucket → chunk → evidence) with minimum score thresholds and citation excerpts, rather than flat vector search.
- **Harness** (`backend/app/harness/` + vendored `harness-engine/`): a registry/executor validating every tool call against a pydantic argument model (`HarnessExecutor`), a Landlock-based Linux sandbox (`harness-engine/native/landlock-run/`), and — strikingly — DeepSeek's own TypeScript harness ("dsh", everything-is-a-plugin on Cordis) embedded wholesale as the Harness v3 runtime.
- **Channels** (`backend/app/channels/`): channel-agnostic adapter registry; `service_intake.py` does idempotent dedupe and durable inbox, `service_outbox.py` does retry/backoff delivery, `service_autoroute.py` classifies intent and routes to the right employee with stickiness during in-flight SOPs and handoff windows.
- **Teams** (`backend/app/teams/`): the multi-agent layer (below). Also an A2A adapter exposing Codex CLI as a durable external agent (`backend/app/a2a/codex_adapter.py`), scheduled-task workers, feedback capture, and memory jobs.

## Key techniques

- **Bid-based task allocation with HP and rebuttals** (`backend/app/teams/wakeup.py`, `service.py`): a task-pool market where members bid with a structured JSON block (`plan`, `estimated_cost`, ...); the team-leader LLM scores every bid (`bid_scores`) and scores *deduct HP* — an agent whose HP hits zero is eliminated — followed by optional rebuttal rounds and an LLM-judged award. Literal game-show mechanics encoded in prompts, with JSON-repair instructions when the judge misbehaves.
- **Shared blackboard memory**: team members may propose blackboard entries with their reports; the TL reviews each suggestion (`blackboard_writes`) before it becomes team working memory — memory writes are *governed*, not automatic.
- **Distillation of SOPs with correctness rules baked into prompts** (`skill_distiller.py`): generated flows must "form a closed loop" (no "please wait" as a final reply — must call tools or hand off), must be *adaptive* (don't re-ask what slots/history already answer), and must *confirm* before irreversible writes. Two model-repair attempts on malformed output.
- **Hierarchical retrieval with cheap lexical-feeling thresholds** instead of pure embeddings: score floors (`SEARCH_MIN_DOCUMENT_SCORE = 2.0` etc.) at each tier, query-noise phrase stripping (Chinese politeness padding like "我想知道" removed before routing), evidence excerpts capped at 900 chars, related-chunk expansion capped at 6.
- **Memory kept deliberately small** (`backend/app/memory/service.py`): only three kinds (`profile`, `preference`, `fact`) captured by an LLM extractor per turn; `recall()` ignores the query entirely and returns the user's whole memory list — a flat, cheap, auditable design that sidesteps retrieval complexity.
- **Operational hygiene rare in agent repos**: Fernet-encrypted channel credentials, idempotent inbound dedupe, crash recovery, session lease fencing (`HarnessSessionLeaseLost`), per-day paginated delivery logs, and a production checklist that warns that mounting an employee on a channel exposes it to *all* that channel's users.

## Design decisions

- **Determinism first, model second.** The SOP state machine, slot enforcement, handoff BFS, and tool validation are all deterministic spines around LLM steps — the same "deterministic spine, agentic leaves" stance the wiki keeps encountering, applied to business process rather than code.
- **Governance as a first-class product surface.** Access scopes, capability subsets per SOP node (`required_*_ids` must be a subset of allowed ones, validated in the schema), citable evidence, full execution traces, human takeover, and a risks section that explicitly disclaims regulated-domain use. The product is as much an audit harness as an agent.
- **Vendor, don't reinvent, the coding-agent layer.** Embedding DeepSeek's dsh harness for sandboxed execution trades coherence for maturity — StaffDeck's own Python harness handles *dialogue-shaped* SOP execution while the vendored TS harness handles *code-shaped* work.
- **Trade-off:** the monolith is huge and single-port, favoring deployability (a desktop .dmg!) over modularity; `agent_loop.py` at 4.2k lines shows the cost. Memory is flat and query-blind — fine for profile facts, weak for episodic history. And the bid market's "HP" semantics are prompt-encoded, i.e. only as reliable as the judge model.

## Comparison notes

- Versus [[The Harness Is the Company]], StaffDeck is the literal artifact of that thesis: the outer loop (routing, staffing, audit, memory governance) is the product, and the model is a swappable part behind `DEMO_MODEL_BASE_URL`.
- Versus [[Munder Difflin — Clones of You, Not a Shared Bot]], which clones *you* on your own laptop, StaffDeck institutionalizes agents as *org-chart* entities with IDs, permissions and knowledge bases — the platform-first pole of the same question.
- Versus [[AI as an Enterprise Operating System]] (adoption as mechanism design), StaffDeck ships the OS itself: a gallery/广场 of reusable skills and knowledge as templates, copy-not-edit semantics, and channel stickiness rules as organizational guardrails.
- Versus [[Multi-Agent AI Systems Are Organizations]]: StaffDeck's teams are a concrete implementation — a leader, a blackboard, and a bidding market — and its HP-elimination bidding is a notable answer to "how do LLM teams pick workers": price discovery via simulated stakes rather than orchestrator intuition.

---
*Sources: [[raw/staffdeck]], [[summary/staffdeck]]*
*Last updated: 2026-10-24*
