# Ruflo

Ruflo is an open-source agent meta-harness that transforms Claude Code from a single-agent coding tool into a coordinated multi-agent platform with swarm orchestration, self-learning memory, cross-machine federation, and enterprise security. It's the most architecturally ambitious open-source harness for Claude Code — implementing plugin-based microkernel architecture, hybrid vector memory, four consensus algorithms, and five learning modes. Its core thesis is that **the harness matters more than the model**: the model writes code, but the harness provides the tools, memory, loops, and controls that make agents actually productive.

---

## Architecture

Ruflo is organized as a **TypeScript monorepo with 35 plugins around a microkernel core**. The v3 directory (`@claude-flow/*` packages) implements domain-driven design with clear separation: domain types, application services, and infrastructure adapters.

**Core modules:**
- **Swarm** (`@claude-flow/swarm`, 1,844 LOC unified coordinator): 15-agent hierarchical mesh with two-tier decision-making — a QueenCoordinator analyzes tasks and produces delegation plans, a UnifiedSwarmCoordinator handles execution with configurable topology (mesh, hierarchical, centralized, hybrid)
- **Memory** (`@claude-flow/memory`, 895 LOC service): hybrid SQLite + AgentDB/HNSW backend that routes structured queries to SQLite and semantic queries to HNSW vector search
- **Hooks** (`@claude-flow/hooks`): event-driven lifecycle with ReasoningBank for vector-based session pattern learning and 10 configurable background workers
- **Neural** (`@claude-flow/neural`): 5 SONA learning modes + 7 RL algorithms for in-process behavioral adaptation, not model fine-tuning
- **Integration** (`@claude-flow/integration`): multi-model router across 8 providers with cost-optimized routing, plus attention coordinator, worker patterns, and provider adapters

**Plugin system** (`v3/src/infrastructure/plugins/PluginManager.ts`): Microkernel pattern with proper dependency resolution, config schema validation, version compatibility checking, and priority-based extension point ordering. More like OSGi than typical CLI plugin systems.

The Rust presence is minimal — two crates for federation networking and WASM embeddings. The "Rust-based AI engine" branding refers to optional native modules accessed via dynamic imports.

## Key Techniques

### Two-tier swarm with consensus

Unlike frameworks that use simple leader-worker dispatch, Ruflo's swarm supports 4 consensus algorithms (Raft, Byzantine, Gossip, Paxos) for collective agent decisions — which task to prioritize, how to allocate resources, when to escalate. The QueenCoordinator layer does task analysis (pattern matching against past successes, resource estimation, delegation planning) before handing off to execution. This is an unusually sophisticated coordination model for a coding-agent harness; most comparable tools use simple round-robin or priority queues.

### Hybrid memory with query routing

The `HybridBackend` (ADR-009) routes queries based on type: structured lookups go to SQLite, semantic search goes to AgentDB with HNSW indexing. A `SmartRetrieval` module (ADR-090) adds query expansion and diversity ranking to prevent the "all top-K results are the same" collapse that plagues pure vector search. A knowledge graph overlay (`MemoryGraph`) provides typed-edge graph hops for finding related entries that vector distance alone would miss.

### SONA as behavioral meta-learning

The 5 SONA learning modes (real-time, balanced, research, edge, batch) adapt agent *behavior* — prompt strategy, tool selection, parallelism decisions — rather than model weights. The 7 RL algorithms (PPO, DQN, A2C, Decision Transformer, Q-Learning, SARSA, Curiosity) are used for meta-learning: learning which strategies work for which task types, not training the LLM itself. This is a clever separation: the LLM stays frozen, the harness learns.

### Dual episodic/semantic memory

Two complementary memory systems: **ReasoningBank** stores raw trajectories for similarity retrieval (episodic — "what happened last time we tried this?"), while **PatternLearner** extracts generalizable patterns from successful trajectories (semantic — "here's the strategy that works for auth refactors"). Both feed into the QueenCoordinator's task delegation decisions, giving the swarm a form of experience-based planning.

### Honest architectural self-awareness

The attention coordinator's code contains an unusual meta-comment: `// Speedup is UNMEASURED — sentinel [0, 0] means "no verified benchmark". // Do NOT restore a fabricated 2.49x-7.47x range.` This is rare honesty in open-source AI tooling, where performance claims often float untethered from measurement.

## Design Decisions

**Ambitious scope over ship safety.** The project claims 35 plugins, 98 agents, 60 commands, 30 skills, 7 RL algorithms, 5 SONA modes, 8 attention mechanisms, and 4 consensus algorithms. Some subsystems are clearly production-grade (plugin system, memory backends, hooks); others appear aspirational or partially wired to native modules that may or may not be available.

**Plugin architecture over monolith.** Everything extends via plugins — even core features like federation and neural trading. This gives users fine-grained control over what's loaded but means system behavior depends on plugin set and load order.

**TypeScript with dynamic native module loading.** The `safeImport<T>()` pattern tries to load native modules (Rust-based HNSW, agentic-flow, WASM embedding generators) and falls back to pure-JS implementations. This is pragmatic engineering — the system works without native deps but gets faster when they're available.

**Significant architectural iteration.** V3 is described as a "complete reimagining" guided by 10 Architecture Decision Records — dropping Deno support, consolidating 4 coordination systems into 1, adopting event sourcing, switching to Vitest. This suggests a project that iterates aggressively, sometimes at the expense of backward compatibility.

**The "nervous system" approach.** `npx ruflo init` installs hooks, settings, agents, skills, MCP config, and a daemon into the workspace — giving Claude Code extensive capabilities but also adding complexity. This contrasts with lighter-weight approaches like single-purpose MCP servers or Claude Code skills.

## Comparison Notes

**Vs. Omnigent (Databricks):** Both are meta-harnesses that wrap existing coding agents. Omnigent focuses on a uniform API for composition and contextual security; Ruflo focuses on swarm coordination and self-learning memory. Omnigent is Apache 2.0 and designed for enterprise composition; Ruflo is MIT and designed as a personal/team augmentation for Claude Code specifically.

**Vs. Paca:** Paca is a self-hosted project management tool where agents are Scrum teammates. Ruflo is lower-level — it provides the orchestration infrastructure (swarms, memory, hooks) but doesn't prescribe a project management methodology. Paca builds on similar ideas (WASM sandboxing, MCP, agent execution) but targets a specific use case.

**Vs. Fleet Supervisor (sermakarevich):** Fleet Supervisor is a Python supervisor for parallel coding agents with a pluggable backend model. Ruflo provides a much broader surface area (memory, learning, federation, plugins) but is tightly coupled to Claude Code's hooks and MCP infrastructure, while Fleet Supervisor is agent-agnostic.

**Vs. Bram (Jon Udell):** Bram is a lightweight Tauri shell providing hash-verified worklist lifecycle and PreToolUse hook enforcement. Ruflo is the opposite approach — maximum capability over minimal ceremony. Both use the Claude Code hooks system, but Bram adds a governance layer while Ruflo adds an entire agent platform.

**Vs. single-purpose MCP servers:** The ecosystem now has many focused MCP servers (memory, browser automation, database access). Ruflo's value proposition is bundling these into a coherent harness with learning loops and swarm coordination — but the trade-off is complexity and surface area, which can introduce hard-to-debug interactions.

## Tags

#tool #project #agents #memory #orchestration

---
*Sources: [[raw/ruflo]]*
*Last updated: 2026-07-18*
