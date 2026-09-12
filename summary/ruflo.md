---
topics:
  - agent-architecture
---
# Ruflo — Summary

Ruflo (formerly Claude Flow, v3.32.2) is an open-source TypeScript meta-harness that wraps Claude Code and Codex with 35 plugins providing multi-agent swarm coordination, self-learning vector memory, federated cross-machine agent communication, and enterprise security controls. Install via `npx ruflo init`.

## Architecture

**Domain-driven monorepo** with three layers: root npm package, v3/ module packages (`@claude-flow/swarm`, `memory`, `neural`, `hooks`, `integration`, etc.), and 35 installable plugins. Minimal Rust for federation networking and WASM embeddings.

## Core Subsystems

- **Swarm**: Two-tier coordination (UnifiedSwarmCoordinator + QueenCoordinator). 15-agent hierarchical mesh. Four consensus algorithms: Raft, Byzantine, Gossip, Paxos — unusual for agent orchestration.
- **Memory**: Hybrid SQLite + AgentDB/HNSW backend. Query routing delegates structured to SQLite, semantic to HNSW. Smart retrieval with graph hops and diversity ranking to avoid similarity collapse.
- **Hooks**: Event-driven lifecycle with ReasoningBank (vector-based session pattern learning), 10 background workers, and Official Claude Code hooks bridge.
- **Neural**: 5 SONA learning modes (real-time, balanced, research, edge, batch) adapting agent behavior in-process. 7 RL algorithms (PPO, DQN, A2C, Decision Transformer, Q-Learning, SARSA, Curiosity) for meta-learning strategy selection.
- **Routing**: Multi-model router across 8 providers with cost-optimized, quality-optimized, and rule-based modes.
- **Plugins**: Microkernel pattern with dependency resolution, version checking, and extension point priority ordering — more like OSGi than loose extensions.

## Key Design Choices

Optimized for ambitious scope and plugin extensibility over ship safety and simplicity. Some components are battle-tested; others are aspirational (the attention coordinator honestly marks its speedup claims as "UNMEASURED"). TypeScript with optional native module loading via dynamic imports. Significant architectural churn (v3 is a "complete reimagining" with 10 ADRs).

## When This Matters

Ruflo is relevant when you need a single harness that provides swarm coordination, cross-session memory, multi-provider routing, and event-driven hooks for Claude Code — rather than assembling these from separate tools like separate MCP servers, memory plugins, and workflow frameworks.
