---
url: https://github.com/ruvnet/ruflo
title: Ruflo — Agent Meta-Harness for Claude Code and Codex
author: RuvNet (ruv@ruv.io)
date_fetched: 2026-07-18
date_published: 2025-08-10
---

# Ruflo — Agent Meta-Harness for Claude Code and Codex

## Overview

Ruflo (formerly Claude Flow) is an open-source TypeScript meta-harness that wraps Claude Code and Codex, adding 35+ plugins for multi-agent swarm coordination, self-learning vector memory, federated cross-machine agent communication, and enterprise security guardrails. One `npx ruflo init` installs hooks, agents, skills, an MCP server, and a daemon into a workspace.

The core claim: **Agent = Model + Harness.** The model writes; the harness provides tools, memory, loops, sandboxes, and controls. Ruflo is the harness layer.

Version analyzed: 3.32.2 (v3 monorepo with 10 ADRs guiding a "complete reimagining" of the architecture). ~5,679 lines in just the 5 core modules analyzed below; the full repo is substantially larger with 35+ plugins, 100+ scripts, docs, and tests.

## Architecture

**Monorepo structure with three layers:**

1. **Root** (`package.json`, `bin/cli.js`) — npm package entry point, `npx ruflo init` bootstraps everything
2. **v3/** — the "reimagined" domain-driven architecture: `@claude-flow/*` modules as separate packages
3. **plugins/**, **.claude/** — 35 installable Claude Code plugins (memory, swarm, federation, neural trader, etc.)

**Core modules (v3/@claude-flow/):**

| Module | File | Size | Purpose |
|--------|------|------|---------|
| `memory` | `memory/src/index.ts` | 895 lines | UnifiedMemoryService wrapping AgentDB + HNSW |
| `swarm` | `swarm/src/unified-coordinator.ts` | 1,844 lines | 15-agent hierarchical mesh coordinator |
| `neural` | `neural/src/sona-manager.ts` | 906 lines | SONA learning modes + RL algorithms |
| `hooks` | `hooks/src/reasoningbank/index.ts` | 1,090 lines | Vector-based pattern learning from sessions |
| `integration` | `multi-model-router.ts` | 1,079 lines | Cost-optimized multi-provider LLM routing |

**Plugin system** (per `v3/src/infrastructure/plugins/PluginManager.ts`): Microkernel pattern. Each plugin implements `Plugin` interface with `initialize()`, `shutdown()`, `getExtensionPoints()`. The PluginManager handles dependency resolution, version compatibility checks, config schema validation, and priority-based extension point ordering. Plugins can declare `dependencies` on each other and `minCoreVersion`/`maxCoreVersion` constraints.

**Two entry points:**
- **Plugin mode** (Path A): `/plugin install ruflo-core@ruflo` — adds slash commands and agent defs only; no MCP server
- **CLI mode** (Path B): `npx ruflo init` — full install: hooks, MCP server, daemon, 60+ commands, 30 skills, 98 agents

**Minimal Rust presence** (`Cargo.toml`): A workspace file with 2 members:
- `v3/crates/ruflo-federation-peer` — federation peer binary
- `v3/plugins/gastown-bridge` — bridge plugin

The Rust is for performance-critical networking and WASM-based components (embeddings via `@ruvector/rabitq-wasm`, vector indexing via `@ruvector/rvf-wasm`). The vast majority of logic is TypeScript.

## Key Subsystems

### Swarm Coordination (`@claude-flow/swarm`)

Two-tier coordinator design:
- **UnifiedSwarmCoordinator** (1,844 LOC): consolidates 4 legacy systems into one canonical coordinator. Supports mesh, hierarchical, centralized, and hybrid topologies. Default: 15-agent hierarchical mesh with Raft consensus at 0.66 threshold.
- **QueenCoordinator** (separate class): "hive-mind central orchestrator" that analyzes tasks, produces delegation plans with agent scoring, monitors domain health, and feeds results back into learning.

The swarm supports 4 consensus algorithms: Raft, Byzantine, Gossip, and Paxos — unusual for an agent orchestration framework (most just do leader election or round-robin). The use case is agents collectively deciding task priority and resource allocation.

**Agent pool** with min/max sizing (1–15 agents), idle timeout (5 min), and workload-aware balancing via `capability-match` strategy. Message bus with 10K queue size and batch delivery.

### Memory System (`@claude-flow/memory`)

**Hybrid backend** (ADR-009): SQLite for structured queries + AgentDB with HNSW indexing for semantic search. The `createHybridService()` factory wires both through `createDatabase({ provider: 'hybrid' })`.

Key components:
- **HNSWIndex** (1,461 LOC): M=16, efConstruction=200, efSearch configurable. Uses cosine similarity with optional quantization.
- **HybridBackend** (789 LOC): routes structured queries to SQLite, semantic queries to AgentDB/HNSW. Supports hybrid queries that fuse structured + semantic results.
- **TieredMemoryStore**: temporal validity with per-entry TTL (Zep/Graphiti-style), recency-weighted recall.
- **MemoryGraph**: builds a knowledge graph from stored entries with typed edges and graph-hop retrieval.
- **SmartRetrieval** (ADR-090): query expansion + diversity ranking, reduces the "similarity collapse" problem where all top-K results are semantically near-identical.
- **AutoMemoryBridge** (ADR-048): automatically synchronizes between Claude Code's memory and the AgentDB backend.
- **LearningBridge**: consolidates patterns from agent trajectories into durable memory.
- **RvfLearningStore** (ADR-057 Phase 6): persists learned patterns, LoRA weights, and EWC states across sessions via RVF format.

**Snapshot system** (ADR-125 Phase 3): every N stores (default 1000), the HNSW index + metadata is flushed to disk. Background consolidator (Phase 4) runs sweep + dedup + compact on a 6-hour timer.

### Hooks System (`@claude-flow/hooks`)

Event-driven lifecycle hooks with these components:
- **HookRegistry**: manages hook registrations by event type
- **HookExecutor**: runs hooks with priority ordering and timeout enforcement
- **ReasoningBank** (1,090 LOC): vector-based pattern learning from agent sessions; stores trajectories, retrieves similar past experiences to guide current decisions. A lightweight RAG system for agent reasoning rather than document retrieval.
- **DaemonManager**: background processes for metrics collection, swarm monitoring, and learning consolidation
- **OfficialHooksBridge**: maps Ruflo hook events to Claude Code's official hook events
- **WorkerManager**: 10 background workers (performance, health, swarm, git, learning, ADR, DDD, security, patterns, cache) with configurable thresholds and alerting
- **StatuslineGenerator**: generates terminal statusline output showing agent state, task progress

### Neural Learning (`@claude-flow/neural`)

Five SONA learning modes, each a different trade-off between compute and adaptiveness:
- **RealTime**: <0.05ms per step, for interactive use
- **Balanced**: default, balances latency and quality
- **Research**: exhaustive analysis, minutes per step
- **Edge**: minimal compute for resource-constrained environments
- **Batch**: offline processing of accumulated trajectories

Seven RL algorithms: PPO, DQN, A2C, Decision Transformer, Q-Learning, SARSA, Curiosity (intrinsic motivation for exploration). Factory function `createAlgorithm()` selects the right one based on task characteristics.

**NeuralLearningSystem**: integrates SONA manager + ReasoningBank + PatternLearner. Records trajectories (begin→step→complete), judges outcomes, distills successful patterns into retrievable memories.

Each learning mode implements `ModeImplementation` with:
- `adapt(behavior, context)`: adjust behavior based on recent outcomes
- `optimize(pattern)`: compress patterns for storage
- `learn(trajectory)`: update internal model

### Multi-Model Router (`@claude-flow/integration`)

Cost-optimized routing across 8 provider types: anthropic, openai, openrouter (100+ models), ollama (local), litellm (unified API), onnx (free local Phi-4), gemini, and custom. Five routing modes: manual, cost-optimized, performance-optimized, quality-optimized, rule-based.

Per-model cost tracking with budget alerts, circuit breaker for reliability, tool-calling translation between provider formats, response caching, and streaming support. The router dynamically selects models based on task type, budget constraints, and quality requirements.

### Federation

Enables agents on different machines to collaborate securely. Uses ephemeral agent IDs, swarm registration, and consensus-based message passing. The Rust `ruflo-federation-peer` binary handles the network layer. 

### Plugin Catalog

35 plugins organized into categories:
- **Core & Orchestration**: core, swarm, autopilot, loop-workers, workflows, federation
- **Memory & Knowledge**: agentdb, rag-memory, rvf, knowledge-graph
- **Security**: adr, security-audit, aidefence, daa
- **Agent Extensions**: agent, goals, intelligence, graph-intelligence, neural-trader, sparc, testgen, plugin-creator
- **Integrations**: browser, business-pods, market-data, iot-cognitum, ruvllm, ruvector, jujutsu, migrations
- **Platform**: arena, docs, observability, cost-tracker, ddd, bbs-federation

Each plugin can contribute agents, MCP tools, slash commands, hooks, and skills.

## Key Techniques

1. **Two-tier swarm with consensus**: Unlike frameworks that use simple leader-worker or round-robin, Ruflo's swarm supports Raft/Byzantine/Gossip consensus for multi-agent decisions. The QueenCoordinator layer analyzes tasks (pattern matching against past successes, resource estimation) before delegating; the UnifiedSwarmCoordinator handles execution.

2. **Hybrid memory with query routing**: SQLite handles `getByKey` and structured queries; AgentDB/HNSW handles `semanticSearch`. The unified API (`UnifiedMemoryService`) routes queries transparently. This is the same pattern as Zep and Graphiti but with snapshot-based persistence and background consolidation.

3. **SONA as behavioral adaptation, not fine-tuning**: The 5 SONA learning modes adjust agent behavior in-process (changing prompt strategy, tool selection, parallelism) rather than updating model weights. The RL algorithms (PPO, DQN, etc.) are used for meta-learning — learning which strategies work for which task types — not for training the LLM.

4. **PatternLearner + ReasoningBank**: Two complementary memory systems. ReasoningBank is a vector store of past trajectories for similarity retrieval (episodic memory). PatternLearner extracts reusable patterns from successful trajectories and stores them as generalized templates (semantic memory). Both feed into the swarm's task delegation decisions.

5. **Plugin dependency resolution**: The PluginManager validates config schemas, version compatibility, and dependency ordering before loading — similar to OSGi or VS Code extensions but for agent plugins. Shutdown order is reverse of load order.

6. **Event sourcing for state changes**: The EventCoordinator (`v3/core/interfaces/event.interface.js`) records all agent lifecycle events, task state transitions, and memory mutations as immutably timestamped events. Enables deterministic replay and debugging.

7. **Smart retrieval with graph hops**: The RAG memory plugin goes beyond vector similarity — it follows typed edges in a knowledge graph (`MemoryGraph`) to find related entries even when vector distance is high. Query expansion and diversity ranking prevent the "all results look the same" collapse.

## Design Decisions

**Optimized for: ambitious scope over ship safety.** The project claims 35 plugins, 98 agents, 60 commands, 30 skills, 7 RL algorithms, 5 SONA modes, 8 attention mechanisms, and 4 consensus algorithms. The README cites 8.1M+ ecosystem downloads. Some components are clearly battle-tested (the plugin system, memory backends, hooks); others appear aspirational or partially wired (the attention coordinator's own code notes the Flash Attention speedup is "UNMEASURED — sentinel [0, 0] means 'no verified benchmark'").

**Optimized for: plugin extensibility over monolithic coherence.** The microkernel pattern means any plugin can add agents, tools, hooks, and commands. This is powerful for customization but means system behavior depends on which plugins are loaded and in what order.

**Optimized for: TypeScript/Node.js ecosystem.** Despite the "Rust" branding in the README ("supercharged Rust-based AI engine"), the Rust presence is minimal — two crates for networking and WASM bridges. The core logic is all TypeScript with dynamic imports for optional native modules.

**Sacrificed: documentation coverage.** The code has rich JSDoc for public APIs but variable coverage for internal components. The plugin READMEs are marketing-focused; architectural docs live in ADRs scattered across the repo.

**Sacrificed: launch-time simplicity for runtime capability.** `npx ruflo init` installs a lot of files — hooks, settings, agents, skills, mcp config, workspace config. This is the "give Claude Code a nervous system" approach: powerful but not lightweight.

## Architectural Observations

**The attention coordinator is an honest piece of engineering.** The code explicitly marks unverified claims: `// Speedup is UNMEASURED — sentinel [0, 0] means "no verified benchmark". // Do NOT restore a fabricated 2.49x-7.47x range.` This is unusual self-awareness in open-source agent tooling.

**The project has undergone significant architectural churn.** V3 is described as a "complete reimagining" with 10 ADRs — dropping Deno support, consolidating 4 coordination systems into 1, adopting event sourcing, switching to Vitest. This suggests a project that iterates aggressively, sometimes at the expense of backward compatibility.

**TypeScript as glue language for native modules.** The project uses dynamic imports (`safeImport<T>()`) to optionally load native modules (agentic-flow, WASM embedding generators, Rust-based HNSW). If the native module is available, it gets used; otherwise, a pure-JS fallback runs. This is a pragmatic pattern for hybrid JS/native projects.

**The plugin system is genuinely reusable.** It's not just a list of directories — it has dependency resolution, config validation, version compatibility checking, and extension point priority ordering. A plugin can declare that it extends another plugin's extension point, and the framework handles the lifecycle.

## Tests

The test suite (Vitest, found under `v3/__tests__/` and `tests/`) covers:
- Integration tests for swarm, memory, plugins, workflows, and MCP
- Appliance tests for GGUF engine, RVFA format, and signing
- Feature gap tests and tool honesty tests
- Unit tests for memory backends (RVF, SQLite, embeddings, HNSW)

Scripts under `scripts/` are smoke tests (prefix `smoke-`) and audit scripts (prefix `audit-`) that verify invariants across the codebase.
