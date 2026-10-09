# Docker Agent

Docker's entry into the agent-framework wars: a ~630K-line Go monorepo behind the `docker agent` CLI plugin that lets you declare agents, teams, tools, RAG, hooks, and budgets entirely in YAML, then run them locally, sandbox them in Docker, and distribute them through OCI registries. Its thesis is that agents are container-shaped artifacts — build once, push, run anywhere — with the multi-agent delegation machinery handled by the runtime, not the config author.

---

## Architecture

- **Entry and packaging**: `main.go` is a thin signal-handling shim into `cmd/root`, built as a Docker CLI plugin (`docker agent`) or standalone binary. The `docker-agent` repo even builds itself: `docker agent run ./golang_developer.yaml` is the documented contributing path.
- **Core abstractions** (`pkg/agent/agent.go`): the `Agent` struct is the whole product surface in one type — instruction, toolsets, provider list with fallback models and cooldown policy, sub-agents, handoffs, routing, compaction thresholds, safety mode, hooks, evaluators, structured output, and budget knobs (`maxIterations`, `maxConsecutiveToolCalls`, `maxToolResultTokens`). A `team.Team` groups agents; an `agentRouter` (`pkg/runtime/agent_router.go`) tracks "which agent is currently driving" with an atomic pointer.
- **The runtime** (`pkg/runtime/runtime.go`, ~2,200 lines): `LocalRuntime` is the control loop — toolset lifecycle, tool-change events, MCP prompt discovery, session store, title generation, steering/follow-up message queues, progressive tool emission to the TUI. Delegation logic lives in `agent_delegation.go`; context management in `compaction/`, `context_breakdown.go`, and `budget.go`.
- **Tools** (`pkg/tools/`): a `StartableToolSet` lifecycle (with backoff in `startable_backoff.go`), MCP transport client (`pkg/tools/mcp/` with OAuth token stores, reconnect, session clients), and ~30 builtin toolsets under `pkg/tools/builtin/` — shell, filesystem, fetch, think, todo, memory, plan, tasks, handoff, transfertask, skills, LSP, openapi, webhook, scheduler, structured output. A `Catalog` interface supports provider-native tool search with `SearchOnly` tools surfaced lazily.
- **Protocols**: the same runtime is exposed as an MCP server, an A2A server (`pkg/a2a/`, Google's agent-to-agent protocol), and an ACP agent (`pkg/acp/`), plus a WASM build (`cmd/wasm/`) that runs the whole runtime in a browser with cloud providers proxied server-side.
- **Supporting layers**: `pkg/rag/` (BM25, embedding, hybrid fusion via RRF/weighted/max, reranking, tree-sitter chunking), `pkg/hooks/` (pre/post tool-use and LLM-call hook pipelines with evaluators), `pkg/sandbox/` (Docker Sandboxes lifecycle), `pkg/skills/` (skill files with frontmatter, local/remote/GitHub sources), `pkg/permissions/`, `pkg/oci/` (push/pull agent packages), `pkg/compaction/`.

## Key techniques

- **Delegation as a validated graph, not free-form calling**: `validateDelegation` in `pkg/runtime/agent_delegation.go` maintains a per-session delegation lineage; it detects cycles by checking the target against the caller-appended lineage and enforces a hard-coded `maxDelegationDepth` of 10 against runaway recursion — a fixed runtime guard, explicitly "not user configuration." Background fan-out allocates fresh lineage slices so concurrent sub-agents never share backing arrays.
- **Fallback models with stickiness**: the `Agent` carries fallback providers, per-fallback retries with exponential backoff, and a `fallbackCooldown` — after a non-retryable failure the runtime sticks with the fallback for a cooldown period rather than flapping back.
- **Proactive compaction**: `pkg/compaction/compaction.go`'s `ShouldCompact` computes input+output+added tokens against the model's context limit with a configurable threshold fraction, and agents can name a dedicated cheaper `compaction_model` for summary generation. Budget config extends to `max_cost`, `max_tokens`, and `max_time` per agent.
- **RAG as strategy composition**: `pkg/rag/strategy/` ships separate BM25 and embedding strategies, `pkg/rag/fusion/` composes them with RRF, weighted, or max fusion, and a rerank stage plugs in behind it — hybrid search assembled from swappable parts rather than one monolithic retriever.
- **Secret redaction wired at three layers**: opting into `redact_secrets` injects a pre-tool-use builtin that scrubs tool arguments, a `before_llm_call` message transform that scrubs outgoing content, and a dispatcher tool-output scrub so secrets never reach event consumers or the persisted session file.
- **WASM runtime with a cloud firewall**: the browser build (`cmd/wasm/`) stubs cloud providers into `cloud_off.go` and proxies real calls through `bridge.go`, so API keys stay server-side — agents as portable browser artifacts.

## Design decisions

- **YAML over code**: the config schema (`pkg/config/latest/types.go`, plus a published `agent-schema.json`) puts models, providers, toolsets, commands, skills, hooks, permissions, budgets, and RAG in one declarative file. The trade-off is real: sophisticated behavior (handoffs, force-handoff, routing) is reachable without programming, but complex logic like `Routing` and `HarnessConfig` leaks into YAML as the framework grows.
- **Distribution via OCI**: agents are pushed to any OCI registry and run by reference (`docker agent run myorg/agent:tag`). That bets on the Docker ecosystem for sharing rather than a marketplace — portable but with no discovery or curation layer.
- **Everything through the same loop**: TUI, headless CLI, eval harness, MCP/A2A/ACP servers, and WASM all sit on `LocalRuntime`, which keeps behavior consistent but concentrates a lot of responsibility in one 2,200-line file.
- **Safety as configuration plus sandbox**: author-declared `safety` modes, network allowlists, and the permissions system live alongside opt-in Docker Sandboxes execution — layered containment rather than one mechanism.

## Comparison notes

- Versus [[Mecatl]], Stacklok's Go harness, the differences are striking: Mecatl inverts Docker Agent's structure — a small explicit engine behind ports with conformance-tested adapters and persistence in Redis/K8s leases — while Docker Agent concentrates capability in one fat `LocalRuntime` and bets on the Docker platform (CLI plugin, Sandboxes, OCI, Model Runner) for persistence and containment. Docker Agent wins on batteries-included; Mecatl wins on architectural legibility.
- Like [[Introducing Omnigent]], it treats agents as composable artifacts, but where Omnigent is a meta-harness wrapping *existing* agents (Claude Code, Codex, Pi) behind a uniform API, Docker Agent is a from-scratch runtime with its own toolsets, delegation, and RAG — including a `pkg/codingharness/` that shells out to Claude Code CLI and Codex as harnesses, an admission that wrapping beats rebuilding for coding work.
- Its lineage-validated delegation with depth caps is a concrete answer to the coordination failure modes catalogued in [[Multi-Agent Systems Have a Distributed Systems Problem]]: cycle detection and a hard depth limit on `transfer_task` chains are exactly the causal-ordering guards that essay says most frameworks omit.
- Its layered containment (safety modes, redact_secrets across three layers, network allowlists, Docker Sandboxes) sits in the same family as [[A Deep Dive on Agent Sandboxes]], but buys its isolation from Docker Sandboxes rather than building a microVM like NVX or Firecracker-based platforms.

---
*Sources: [[raw/docker-agent]], [[summary/docker-agent]]*
*Last updated: 2026-10-09*
