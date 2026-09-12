---
url: https://github.com/microsoft/accordant
title: "Accordant: Executable behavioral specifications for .NET"
author: Microsoft
date_fetched: 2026-06-15
date_published: 2025
topics:
  - guardrails-and-feedback-loops
---

# Accordant: Executable behavioral specifications for .NET

Accordant is a model-based testing framework for .NET where you write an executable *spec* — code that captures the rules of your system. Given any state and any operation, the spec defines what the response should be and how the state should change. Accordant then generates test cases by exploring the state graph, runs them against your real implementation, and validates every response.

## Architecture

The codebase is organized into 7 C# projects:

| Project | Role |
|---------|------|
| `Accordant` | Core: `State`, `StateGraph`, `IStepFunction`, `SystemChecker`, `SharedStateAttribute`, `StateProfile` |
| `Accordant.Operations` | Test generation/execution: `Spec`, `Operation`, `Expect`, `ExpectedOutcome`, `TestCaseGenerator`, `TestCaseExecutor`, generation algorithms, persistence |
| `Accordant.Operations.Http` | HTTP transport layer: `HttpExecutable`, `HttpRequest`, `HttpResponse` |
| `Accordant.SourceGenerator` | Roslyn incremental source generator: generates `CloneInternal`, `AppendFieldHashes`, `StringRepresentationInternal`, `FreezeComponents` for `[State]`-annotated classes |
| `Accordant.Choose` | Combinatorial choice library (internal utility) |
| `Accordant.Invariant` | Runtime assertion helper |
| `Microsoft.Accordant.Cli` | `dotnet accordant new` CLI for project scaffolding |

### Core Pipeline

1. **Define state**: A POCO marked `[State] public partial class` — the source generator produces deep cloning, XxHash64 fingerprinting, deterministic string representation, and freeze-on-use immutability
2. **Define operations**: Subclass `Operation<TRequest, TResponse, TState>` implementing `Apply(request, state)` (the spec/oracle) and optionally `Execute(context, request)` (the runtime call)
3. **Provide inputs**: `InputSet` with sample request values for each operation
4. **Generate tests**: `spec.GenerateTests(initialState, inputs)` — builds a state graph, walks paths, outputs `SequentialTestCase` or `ConcurrentTestCase` lists
5. **Execute**: `spec.RunTests(context, initialState, testCases)` — runs the system, validates every response against the spec oracle

### State Graph Exploration

`StateGraph.ExploreStateGraph` (`Accordant/StateGraph.cs:422`) performs DFS from an initial state. Each node is fingerprinted with XxHash64(state hash + step function IDs) for deduplication. Step functions are *consumed* on application (they fire once) but can *produce* new step functions (for async work). The graph optionally returns edges/children or just traverses with hooks.

### Step Function System

`IStepFunction` (`Accordant/StepFunction/IStepFunction.cs:38`) is the core abstraction for state transitions:

- **`InputStepFunction`**: Applies an operation input — the primary mechanism for generating test sequences
- **`ContractStepFunction`**: Validates an observed (request, response) pair against a spec
- **`TerminatingStepFunction`** / **`AsyncOperation`**: Models background work that eventually completes; used for polling and unwinding during test generation

Step functions are consumed (removed from the state profile) after application, but can emit new step functions. This models async workflows: an operation triggers a `TerminatingStepFunction` that fires in subsequent steps until it reaches a terminal state.

### SystemChecker and Linearizability

`SystemChecker.Validate` (`Accordant/SystemChecker.cs:118`) determines whether a concurrent set of operations can be explained by *some* sequential ordering. It uses `StateGraph.ExploreStateGraph` on all possible interleavings of step functions (operations + background step functions). If any ordering produces a valid path where all `ContractStepFunction` instances are satisfied, the concurrent execution is linearizable.

This is the key algorithm for concurrency testing: run operations in parallel, feed the results to `spec.AllowsConcurrent()`, and let the SystemChecker search for an explanation.

### State Immutability Model

`State` (`Accordant/State/State.cs:364`) uses a freeze-on-use pattern:
- State objects are mutable when created (user modifies them imperatively)
- The framework freezes them (immutable after return from `ApplyInternal`)
- Frozen states cache their hash and string representation
- The only way to modify a frozen state is to clone it (deep copy, unfrozen)
- `ValidateNotMutated()` checks frozen-state integrity by comparing pre-freeze and post-use fingerprints

### Source Generator

`StateSourceGenerator` (`Accordant.SourceGenerator/StateSourceGenerator.cs:215`) is a Roslyn `IIncrementalGenerator` that processes classes marked `[State]`. Its pipeline:
1. Syntax filter finds candidate class/record/struct declarations with attributes
2. Semantic extraction validates the class and enumerates properties, handling `[SharedState]` references
3. Type analysis classifies each property into a `TypeKind` (Primitive, String, List, Dictionary, StateClass, etc.)
4. Validation propagates errors transitively (if A depends on B and B has errors, A is also flagged)
5. Code emission generates the partial class with `CloneInternal` (deep clone), `AppendFieldHashes` (incremental XxHash64), `StringRepresentationInternal` (deterministic serialization), and `FreezeComponents` (recursive freeze)

25 diagnostic codes (`STATE001`–`STATE025`) cover everything from missing `partial` to unsupported dictionary key types.

### Test Case Algorithms

Three sequential algorithms in `SequentialTestCaseAlgorithms` (`Accordant.Operations/GenerationAlgorithms/SequentialTestCaseAlgorithms.cs:208`):
- **StateCoverage**: DFS that ensures each reachable state is visited at least once
- **TransitionCoverage**: Generates all possible edge traversals up to `maxSequenceLength` (default 3)
- **RandomWalk**: N random walks with configurable max length and seed, with per-walk deduplication of operation names and cross-walk dedup of identical paths

Concurrent algorithms in `ConcurrentTestCaseAlgorithms` (`Accordant.Operations/GenerationAlgorithms/ConcurrentTestCaseAlgorithms.cs:166`): state coverage with concurrent execution at each node, combinational subsets when many edges exist.

### Fluent Expect API

`Expect` (`Accordant.Operations/Expect.cs:656`) provides a builder pattern for defining expected outcomes:

```csharp
Expect.That<TResponse>(predicate, explanation)
      .ThenState<TState>(modifier)          // auto-clone + mutate
      .ThenState<TState>((resp, s) => ..., mock: ...)  // response-dependent
      .SameState()                          // state unchanged
      .Triggers(stepFunction)               // background work
      .TriggersWhen(predicate, stepFunction) // conditional
Expect.OneOf(outcome1, outcome2)            // non-deterministic
Expect.Throws<TException>()                 // exception expected
Expect.Unit()                               // void operation
```

The `ThenState` overloads use automatic cloning (the framework clones the current state, passes it to the modifier action). Response-dependent state requires a `mock` function for state exploration (when the real response isn't available yet).

### Polling and Async

`PollingSetup` on an operation tells the framework to poll via another operation. When a `TerminatingStepFunction` is triggered, then during test execution the framework polls the specified operation until the step function's `IsTerminalState` returns true for all possible states.

### Persistence

Test cases can be serialized to/from JSON with `TestCaseFileRecord` classes and metadata wrappers. This supports saving generated tests to files and loading them back — useful for CI where generation and execution happen in different stages.

## Key Techniques

- **Spec as oracle**: The spec is reused for test generation (predicting state transitions), execution validation (checking observed responses), and concurrency analysis (explaining parallel results). One codebase, three uses.
- **State graph enumeration**: Exhaustive DFS with XxHash64 fingerprinting avoids re-exploring equivalent states. The graph naturally discovers all reachable states and transitions from a small input set.
- **Linearizability checking**: `SystemChecker` searches all interleavings of concurrent step functions to find a sequential ordering that explains observed responses. This is brute-force but correct for small concurrency groups.
- **Freeze-on-use immutability**: States are mutable during construction (simple to write), frozen by the framework (safe for hash caching and mutation detection). A pragmatic middle ground between full immutability and defensive copying.
- **Source generation for boilerplate**: The `[State]` attribute + Roslyn generator eliminates hand-written clone/hash/freeze code. The generator handles nested states, cycles, and 25 different diagnostic conditions.
- **Step function consumption/production**: Operations consume step functions (they fire once) but can produce new ones. This directly models async workflows where an operation starts background work that completes independently.
- **Mock response generators**: When state transitions depend on response values, the spec provides a `mock` function for state exploration. The mock is used during test generation; the real response drives the transition during execution.
- **Separation of Apply and Execute**: `Apply` defines the spec (pure logic, no side effects), `Execute` runs against the real system. This independence lets the spec serve as both generator and oracle.

## Design Decisions

- **Immutable but not immutable-by-default**: State is mutable until frozen, unlike FP approaches. Trade: simpler construction vs. potential mutation bugs caught by `ValidateNotMutated`.
- **State graph is the core abstraction**: Everything derives from graph exploration — test generation, concurrency checking, visualization. This is the right call; it eliminates duplicate logic.
- **Depth-limited exploration with `maxDepth`**: Prevents infinite graphs (e.g., a counter that increments without bound). Default algorithms also cap sequence length.
- **Deterministic hashing everywhere**: XxHash64 for performance, with collision-resistant fingerprints (state hash + step function IDs). Critical for deduplication during graph traversal.
- **Fluent API over configuration**: The `Expect.That(...).ThenState(...).Triggers(...)` pattern reads like a truth table. Better DX than config objects, though it means the framework has more API surface to maintain.
- **Built-in algorithms, replaceable**: Default test generation algorithms (StateCoverage, TransitionCoverage, RandomWalk) are provided but the API accepts custom `SequentialTestCaseAlgorithm` delegates.
- **No external service dependency**: All state exploration is in-process. Tests run against the real system but state simulation is purely local.
- **Hub-and-spoke dependency injection**: `TestingContext` acts as a service locator (`.Get<T>()`). Simple but lacks the compile-time safety of constructor injection.
- **Incremental source generator**: Uses Roslyn's `IIncrementalGenerator` for performance — critical for IDE responsiveness when state models have many properties.

## Comparison Notes

**vs. traditional unit tests**: Accordant inverts the relationship — the spec is the source of truth, tests are generated. Traditional tests scatter business rules across assertions; Accordant centralizes them, then verifies the system mechanically.

**vs. property-based testing (QuickCheck, FsCheck)**: Both generate test cases algorithmically. PBT generates *inputs* from type generators; Accordant generates *sequences* from a behavioral model. PBT checks invariants; Accordant checks exact response/state correctness. PBT is black-box; Accordant uses a white-box state model.

**vs. TLA+/Alloy**: Both model state space. TLA+/Alloy verify properties symbolically; Accordant generates concrete test cases and runs them against a real implementation. TLA+ finds design bugs; Accordant finds implementation bugs.

**vs. contract testing (Pact)**: Both define expectations about API behavior. Pact captures consumer-provider interactions; Accordant models the full state machine of the system. Pact is for integration boundaries; Accordant is for behavioral correctness.

**vs. [[A New Era for Software Testing]]**: antirez describes giving an LLM agent a markdown checklist and having it inspect commits to write targeted tests. Accordant has the same goal (systematic behavioral testing) but a fundamentally different mechanism: it generates tests from a formal spec rather than from commit analysis. The approaches could be complementary — Accordant for known behavioral contracts, LLM-based QA for regression coverage.

**vs. [[The Oracle Is the Asset]]**: Sam Ruby's thesis that the test suite is the durable asset aligns perfectly with Accordant's architecture. In Ruby's framing, Accordant is the tool that lets you write the oracle once and generate the verification from it. The spec IS the oracle; the generated tests are the verification.

**vs. [[Apache Burr]]**: Both use explicit state machines. Burr models agent workflows (actions as Python functions with typed state reads/writes); Accordant models system behavior for testing. Same mental model (state + transitions + graph) but different domains: Burr builds agents, Accordant tests them.

**vs. [[Maestro (UI Testing)]]**: Maestro provides a YAML DSL for UI test scenarios; Accordant generates test scenarios from a behavioral spec. Maestro is for writing tests; Accordant is for deriving them from first principles.

## Limitations

- Maximum depth constraints can miss bugs in long sequences (balanced by pragmatic defaults)
- Concurrent test generation is combinatorial — large operation sets may need `maxConcurrencyLevel` tuning
- Source generator requires `partial` class + `[State]` attribute — adds ceremony
- Hash collisions possible (64-bit XxHash64) though astronomically unlikely in testing contexts
- No support for probabilistic state transitions (e.g., "5% failure rate")
- .NET-only — the model-based testing concept is general but this implementation is C#-specific
- The `TestingContext` service locator pattern couples tests to a dependency container rather than explicit constructor injection

## Dependencies

- **Roslyn (Microsoft.CodeAnalysis.CSharp)**: For source generation
- **System.IO.Hashing**: XxHash64 for fast fingerprinting
- **System.Text.Json**: Test case serialization
- **NUnit**: Test framework (tests only)
- **System.CommandLine**: CLI scaffolding (CLI only)
- **System.Collections.Immutable**: Immutable collections for path tracking
