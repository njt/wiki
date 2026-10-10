# REA (morluto/rea) — MCP Server for Agent-Driven Reverse Engineering

The GitHub repository behind REA: an MIT-licensed TypeScript monorepo (~137K lines of non-test TypeScript, 284 test files, npm package `rea-agents`, ~40,000 GitHub stars) that packages reverse engineering as an MCP tool catalog any coding agent can drive. Where [[REA — Reverse Engineering with Your Coding Agent]] covers the landing page's pitch and demos, this note is about how the thing is actually built — and the build is where the interesting engineering lives.

---

REA's claim: point your agent at a shipped binary, app, or website and it will investigate, explain a feature with evidence, and reimplement it. The repo backs this with an unusually serious codebase — this is not a prompt wrapper around `objdump`. It is a provider-adapter system with a typed protocol, an evidence ledger, and per-engine bridge processes.

## Architecture

**One MCP server, many analysis engines, adapter bridges between them.**

- `src/main.ts` (117 lines) is the whole entry point: parse config from environment, `createBinarySession`, open an initial target, start the MCP stdio transport via `@modelcontextprotocol/server`. The MCP server owns one `BinarySession` for its process lifetime.
- `src/server/` registers the tool catalog in dozens of `register*Tools.ts` files — android, artifacts, browser, browser-scenario, electron, EVM, firmware, managed (.NET), process capture, web captures, sessions, evidence, prompts. The catalog is *static and complete*: `tools/list` always returns every tool including currently-unavailable ones, and never emits `notifications/tools/list_changed` (per `docs/mcp-contracts.md`) — availability is a runtime property reported by `binary_session`, not a catalog change.
- `src/contracts/` holds Zod schemas for every tool's input and output. A notable piece of contract engineering: root object unions are projected into a single object without root `anyOf`/`oneOf` "for model API compatibility", because model providers choke on top-level unions — runtime validation still enforces the exact union the advertisement softens.
- `src/bridge/` (Python/Java/Swift) are scripts injected into the analysis engines themselves: `hopper_bridge.py` (1,139 lines) runs *inside Hopper's own Python thread*; `ReaGhidraBridge.java` runs as a Ghidra extension; `rea_lldb_tracer.py` drives LLDB; pwntools modules handle ELF layout and recorded crashes; a mitmproxy addon handles captures.
- `src/composition/` wires providers per target type in tiny files (6–53 lines) — a hand-rolled dependency-injection layer over `src/domain/` types and `src/application/` services.

The bridge protocol (`src/hopper/protocol.ts`, 119 lines) is a small authenticated JSON-RPC: Zod-validated responses with typed remote errors (`remote`, `authorization`, `capability_unavailable`, `bridge_exception`) plus a discriminated-union event stream for `progress` and `diagnostic` events. Authentication is a random `REA_TOKEN` injected at bootstrap and checked with HMAC in the bridge — a local socket, but treated as hostile territory.

## Key techniques

- **In-process engine bridging, not subprocess scraping.** The Hopper bridge's docstring is an implementation war story: "Keep all Hopper API access on this thread: moving dispatch to a worker can deadlock Hopper." It lazily resolves Hopper's injected globals so the module can be imported and unit-tested without Hopper, via a `HopperApiFacade` Protocol boundary. That is why the repo has meaningful test coverage for a closed-source GUI disassembler.
- **Evidence as a first-class domain type.** `src/domain/evidence.ts` defines evidence records with SHA-256 digests over subjects (mach-o, ELF, ASAR, APK, …), provider identity, and canonical JSON digesting. `EvidenceMcpServer` wraps the MCP transport itself so that even *oversized tool-result errors* are bound into the evidence ledger before recovery — failures become auditable artifacts. Tool results come back with citations and stated limitations, which is what makes the agent's downstream claims checkable.
- **Canonical digests everywhere.** `canonicalDigest`, `digestCanonicalValue`, `digests.ts`, `ArtifactHash` — every artifact read goes through stable hashing so an investigation can be replayed and compared across versions. The CLI's `compare_web_captures` and session tools accept only complete `normalized_result` documents, rejecting partial captures at the schema level.
- **The skill as a contract.** `skill-src/reverse-engineer-anything/SKILL.md` (versioned, currently "34") teaches the agent *when not to use REA* ("Skip REA for ordinary source-repository architecture analysis"), how to distinguish a stale registration from a missing provider without changing files, and to present a read-only `setup --dry-run` plan and get approval before writing any configuration. The setup flow backs up existing agent configs before modifying them.
- **Budgeted decoding.** Files like `InterfaceBuilderDecodeBudget.ts` and `NativeUiOutputBudget.ts` cap how much parsed structure is emitted, keeping tool responses within model-usable size rather than dumping whole parse trees.

## Design decisions

- **Provider neutrality over engine lock-in.** Native analysis works over Hopper, Ghidra, or IDA — the user brings the engine (or approves an install); REA brings the protocol and contracts. The cost is an adapter layer per engine and a capability negotiation surface (`CapabilityInventory.ts`, `HopperProviderCapabilities.ts`); the benefit is that the tool catalog stays stable while engines come and go.
- **Static completeness over dynamic catalog mutation.** Advertising unavailable tools costs context tokens, but it sidesteps a whole class of agent confusion about tools appearing and vanishing mid-session, and it makes the catalog deterministic and testable.
- **Verification as the headline, not a footnote.** The showcases are chosen for checkability: the DX-Ball reconstruction passes 3,205 cases against the original x86 and reproduces all 63 compiled bytes; TH04 is diffed against period-compiler output. The project treats "the agent's explanation is right" as a claim requiring evidence — same instinct as [[Mole]]'s verbatim quote-checking, but applied to binaries instead of research sources.
- **Local-first with explicit trust boundaries.** Targets never leave the machine; the README is careful that the model provider's data policy is the only exfiltration path. Runtime capture runs under the user's own permissions, with each runtime guide documenting exactly what gets executed.

## Comparison notes

- [[REA — Reverse Engineering with Your Coding Agent]] is this repo's public face; its "agent as operator, human as approver" framing is implemented literally in the setup/doctor/dry-run flow documented in the skill.
- [[Kuna — Agent-First Decompiler]] attacks the same problem from the opposite direction: a decompiler built agent-first from scratch, where REA is an adapter over decompilers that already exist. Kuna owns the analysis; REA owns the interface.
- [[WinDbg MCP]] is Microsoft's version of the same pattern for crash debugging — a specialist tool exposed over MCP with a skill that treats model conclusions as hypotheses. REA generalises the pattern across a dozen target types, and shares its token-authenticated local bridge mechanics.
- [[Stateless MCP]] argued declared MCP tools beat arbitrary shell access for security; REA is a strong instance — agents get surgical reverse-engineering tools instead of driving Ghidra through shell commands, with every result carrying evidence rather than free-form output.

#tool #project #mcp #reverse-engineering #security

---
*Sources: [[raw/rea]], [[summary/rea]]*
*Last updated: 2026-10-10*
