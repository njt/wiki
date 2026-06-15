# Accordant

Executable behavioral specifications for .NET — Microsoft's framework for model-based testing. You write a *spec* (a single source of truth defining expected responses and state transitions for every operation), provide sample inputs, and Accordant generates, executes, and validates hundreds of tests against your real implementation — including sequential, concurrent, and async workflow tests.

---

## Architecture

Accordant is a C# .NET library (7 projects, ~4,100 LOC core logic), delivered as the `Microsoft.Accordant` NuGet package (MIT license).

**Core pipeline**: [[State]] definition → [[Operation]] spec → [[InputSet]] → State graph exploration → Test case generation → Execution + validation. The same spec serves three roles: test oracle (during generation), response validator (during execution), and linearizability checker (for concurrency).

**Key modules** (file-level refs from the repo):

| File | Role |
|------|------|
| `Accordant/StateGraph.cs:422` | State graph DFS exploration with XxHash64 fingerprint dedup |
| `Accordant/State/State.cs:364` | Abstract state: clone, freeze-on-use, XxHash64 hashing, deterministic string representation |
| `Accordant.Operations/Operation/Operation.cs:423` | `Operation<TRequest, TResponse, TState>` — abstract base with `Apply` (spec) and `Execute` (runtime) |
| `Accordant.Operations/Operation/Spec.cs:703` | `Spec<TState>` — operation registry, test generation, execution, conformance checking |
| `Accordant.Operations/ExpectedOutcomes.cs:331` | `ExpectedOutcome` / `ExpectedOutcomes` — response validation + state transition + step function triggering |
| `Accordant.Operations/Expect.cs:656` | Fluent builder API: `Expect.That(...)`, `.ThenState(...)`, `.Triggers(...)`, `Expect.OneOf(...)` |
| `Accordant/SystemChecker.cs:118` | Linearizability checking: searches all interleavings of concurrent operations for a valid sequential ordering |
| `Accordant/StepFunction/IStepFunction.cs:270` | Step function abstraction: `IStepFunction`, `BaseStepFunction`, `TerminatingStepFunction`, `AsyncOperation` |
| `Accordant.SourceGenerator/StateSourceGenerator.cs:215` | Roslyn incremental generator producing clone/hash/freeze code for `[State]` classes |
| `Accordant.Operations/TestCaseGenerator.cs:801` | Test generation orchestration: graph construction → algorithm → serialization |
| `Accordant.Operations/TestCaseExecutor.cs:1400` | Test execution with lifecycle hooks, polling, and verification |

**Data flow**: User writes `[State] public partial class MyState` → source generator produces partial class with `CloneInternal`, `AppendFieldHashes`, `StringRepresentationInternal`, `FreezeComponents` → operations defined via `Operation<TReq, TRes, TState>` or inline lambdas → `InputSet` provides sample requests → `StateGraph.ExploreStateGraph` builds the state graph via DFS with fingerprint dedup → generation algorithm walks the graph collecting operation call sequences → `TestCaseExecutor` runs the system, validates responses via `Spec.Allows()`, handles polling and lifecycle hooks.

## Key Techniques

**Spec as triple-use oracle**: The same `Apply(request, state)` method is used for (1) state prediction during test generation, (2) response validation during test execution via `Spec.Allows()`, and (3) linearizability checking via `SystemChecker.Validate`. One code path, three purposes — eliminates divergence.

**State graph enumeration with fingerprinting**: `StateGraph.ExploreStateGraph` (`Accordant/StateGraph.cs`) performs DFS from an initial state, applying step functions at each node. Each node gets a fingerprint via `XxHash64(state hash + sorted step function IDs)`, avoiding re-exploration of equivalent states. The graph naturally discovers all reachable states — operations that change state create new nodes, operations that don't loop back.

**Freeze-on-use immutability**: States are mutable when created (imperative construction), then frozen by the framework before use (immutable with cached hash/string representation). Modification of a frozen state is detected via fingerprint comparison and throws `StateFrozenException`. Cloning is the only way to derive new state. This hits the ergonomic sweet spot between full immutability and defensive copying.

**Step function consumption/production**: Step functions are *consumed* when applied (removed from the state profile) but can *produce* new step functions. This directly models async workflows: an operation triggers a `TerminatingStepFunction` that fires in subsequent steps until `IsTerminalState` returns true. The framework polls during execution and unwinds during test generation.

**Linearizability checking for concurrency**: `SystemChecker.Validate` (`Accordant/SystemChecker.cs`) brute-forces all interleavings of concurrent step functions, searching for a sequential ordering that satisfies the spec. If found, the concurrent execution is linearizable. This catches race conditions, double bookings, and lost updates without requiring explicit concurrency annotations in the spec.

**Roslyn incremental source generation**: The `StateSourceGenerator` (`Accordant.SourceGenerator/`) uses Roslyn's `IIncrementalGenerator` pipeline: syntax filter → semantic transform → type classification (25 `TypeKind` values) → error propagation → code emission. Generates deep clone, incremental hash, deterministic serialization, and recursive freeze. 25 diagnostic codes cover everything from missing `partial` to unsupported dictionary key types.

**Response-dependent state with mock generators**: When `ThenState` depends on response values (e.g., server-generated IDs), the spec provides a `mock` function for state exploration. The mock is used during test generation; the real response drives the transition during execution. The framework requires both paths to prevent silent divergence.

**Fluent truth-table API**: `Expect.That(r => predicate, explanation).ThenState(s => s.Property = value)` reads as: "I expect the response to satisfy this predicate, and after that the state should have this property set." Combined with `Expect.OneOf(...)` for non-determinism and `.Triggers(...)` for async work, the spec code reads like a truth table.

## Design Decisions

- **State as simplified black-box model**: The state intentionally ignores implementation details (SQL vs Redis vs flat file). Only the information needed to define *correct behavior* is modeled. → Simpler specs, but may miss storage-specific bugs.
- **Graph-first architecture**: Everything derives from state graph exploration — test generation, concurrency checking, visualization. No duplicate traversal logic. → Consistent but means the graph must be tractable (depth limits, state constraints).
- **Inline operations via lambdas**: `spec.Operation<TReq, TRes>(name, (req, state) => { ... })` for simple cases; subclassing `Operation<,,,>` for complex ones with custom `Execute` and `DerivedFrom`. → Low ceremony for simple specs, full power when needed.
- **Deterministic XxHash64 over SHA256**: XxHash64 for node fingerprinting (performance), length-prefixed hashing for collision resistance. 64-bit is fine for test-sized state spaces.
- **Built-in algorithms, replaceable delegates**: Default algorithms (StateCoverage, TransitionCoverage, RandomWalk) can be swapped via `TestGenerationOptions`. → Sensible defaults, extensible for fuzzers or custom exploration.
- **No external service dependency**: All state exploration is in-process. Tests run against the real system (via `Execute`/`ExecuteAsync`) but simulation is purely local.
- **CLI scaffolding for AI agents**: The `agent/INSTALL.md` and `agent/starter/` template let AI coding agents bootstrap an Accordant project. A bet that AI-written specs will outnumber hand-written ones.

## Comparison Notes

**vs. [[A New Era for Software Testing]]**: antirez's approach (LLM agent inspects commits, writes targeted tests from a markdown checklist) and Accordant share the goal of systematic behavioral testing but differ in mechanism. Accordant generates tests from a formal spec (deterministic, reproducible); antirez's approach is LLM-driven (flexible, regression-focused). They're complementary — Accordant for known behavioral contracts, LLM-based QA for commit-specific regression coverage.

**vs. [[The Oracle Is the Asset]]**: Sam Ruby's thesis that the test suite is the durable artifact finds its purest expression in Accordant. The spec IS the oracle; the generated tests and the implementation that passes them are derivatives. Ruby: "frameworks will become transpilers, and you'll own the spec." Accordant is the framework that makes the spec the durable asset today.

**vs. [[Apache Burr]]**: Both use explicit state machines with state + transitions + graph. Burr models agent workflows (Python decorators on functions); Accordant models system behavior for testing (C# operations on state). Same mental model, different domains — and they compose: you could spec a Burr agent with Accordant.

**vs. property-based testing (QuickCheck/FsCheck)**: PBT generates *inputs* from type generators and checks invariants. Accordant generates *operation sequences* from a behavioral model and checks exact response/state correctness. PBT is input-focused ("for all inputs, property holds"); Accordant is sequence-focused ("for all reachable states, every operation produces the right response").

**vs. pact/contract testing**: Pact captures consumer-provider interactions (HTTP request → response). Accordant models the full state machine (request → response + state transition). Pact is integration-boundary correctness; Accordant is behavioral correctness. They operate at different abstraction levels.

**vs. TLA+/Alloy**: Both model state spaces. TLA+/Alloy verify properties symbolically; Accordant generates concrete test cases and runs them. TLA+ finds *design* bugs; Accordant finds *implementation* bugs. A project could use TLA+ for the high-level design and Accordant to verify the implementation matches.

## AI Integration

Accordant explicitly targets AI coding agents as users. The `agent/` directory contains:
- `agent/INSTALL.md` — instructions for AI agents to bootstrap a project
- `agent/AGENTS.md` — context file consumed by coding agents
- `agent/starter/` — complete example spec (key-value store) with state, operations, tests, and project file
- `agent/skills/README.md` — skill definitions for AI agents

The README states: "AI can help write the spec, and the spec provides strong guardrails when AI generates the implementation." This is a two-way bet: LLMs write specs (capturing intent), and specs mechanically verify LLM-generated code (enforcing correctness).

---
*Sources: [[raw/microsoft-accordant]], https://microsoft.github.io/accordant/docs/index.html*
*Last updated: 2026-06-15*
