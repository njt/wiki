# Rebalancer — Meta's Assignment Solver Library

Rebalancer (facebook/rebalancer, Apache 2.0, OSDI 2024 paper) is the library Meta extracted from "dozens" of hyperscale resource-allocation problems — server allocation, ML training placement, traffic routing, load-balancing migrations. It is a generic *assignment solver*: you declare objects, containers, numeric dimensions, and a set of constraint/goal *specs*, and it finds an assignment of every object to exactly one container that respects the constraints and optimizes the goals, at up to ~1M objects. What makes it worth study is the split it draws between **problem definition and solving algorithm**: the same declarative model can be fed to a fast-but-heuristic local search or a slow-but-optimal mixed-integer solver, with zero changes to the model.

---

## Architecture

Single-process C++ core (~240K lines under `algopt/`) with multi-threaded parallelism, layered cleanly:

- **`entities/`** — the world model: `Universe.h` (~165 lines, the root), `Objects`/`Containers`, `Scope`/`Scopes` (hierarchical grouping — rack, cluster, region), static and dynamic object dimensions, plus exotic ones like `ObjectPartitionRoutingDimension` and `RoutingRing` (consistent-hash-style routing-aware assignment).
- **`interface/`** — the public API: `ProblemSolver.h` (893 lines) with a fluent builder (`setAssignment`, `addObjectDimension`, `addConstraint`, `addSolver`), `ProblemSolverFactory`, and a Thrift-typed spec layer shared across languages.
- **`materializer/`** — the compiler. ~60 `SpecBuilder`s (`BalanceSpecBuilder`, `CapacitySpecBuilder`, `ColocateGroupsSpecBuilder`, `DisasterRecoveryCapacitySpecBuilder`…) lower each declarative spec into an expression tree (`ExpressionBuilder`), running as folly coroutines (`Materializer.h` materializes goals and constraints concurrently). A `SplitConstraint` mechanism divides a constraint into hard, soft, and penalty components — hard for MIP feasibility, penalty-shaped for local search guidance.
- **`solver/`** — the engines. `solvers/LocalSearchSolver.cpp` drives `CoreLocalSearchSolve`, cycling over a `HotContainerSelector` and a library of ~30 **move types** (`moves/SingleGreedyMoveType`, `SwapMoveType`, `SingleChainMoveType`, `GroupMoveWithHintStrategiesMoveType`, `KLSearchMoveType`…). `expressions/` holds the evaluator the moves differentiate over (81 files: `LinearSum`, `NthLargest`, `Piecewise`, `MinimizeSquares`…). `OptimalSolver.cpp` instead encodes the same problem for the MIP backends in `algopt/lp/detail/{highs,gurobi,xpress}` behind a generic LP interface.
- **`python/Bindings.cpp`** (1,037 lines) — nanobind bindings that accept Thrift-shaped dicts via JSON round-tripping, so the OSS Python package mirrors the internal API without thrift-python dependencies.
- **`explorer/`** — a debugging UI: C++ Thrift backend serving run data, a thin JSON proxy (`POST /v2/<method>`) so a Next.js frontend needs no Thrift toolchain, wired up with docker-compose and seeded example bundles.

## Key techniques

- **Specs as a constraint DSL.** Users never write expressions; they pick from a finite, extensible catalogue of ~60 parameterized specs. Each spec builder knows how to emit *both* a local-search expression tree and (via `LpEvaluator`) MIP rows — one declarative surface, two compilation targets.
- **Move-type zoo instead of one neighborhood.** Rather than a single swap neighborhood, local search composes move types (single, swap, chains, group moves with hint strategies, triple loops, stratified random batches) and can run them in *stages* (`LocalSearchStageSolver`) with different objectives per stage. Benchmarks exist per move type (`benchmarks/move_types_benchmarks/`), i.e. the neighborhood design is treated as a measured engineering surface, not an algorithmic footnote.
- **Objective tuples with priorities.** `SolveState` carries `objectiveTupleStart/End` and a `higherPriorityObjConfig` — lexicographic multi-objective handling where higher-priority goals gate lower ones, plus `GlobalObjectiveValue` for comparisons across cycles.
- **Constraint softening.** `getSoftenedConstraint` converts hard constraints into penalty terms so local search can traverse slightly-infeasible regions on its way to feasible-and-better solutions, while MIP keeps them hard.
- **Engineering-for-two-worlds build.** The README openly documents the oddest decision: CMake globs the whole tree and auto-classifies files as library/tests/benchmarks because the same source must compile on Meta's internal buck/fbcode infra and in OSS CMake — adding a file requires re-running CMake manually.
- **The repo ships agent skills.** `.llms/skills/rebalancer/SKILL.md` (682 lines) documents `eval_move`, `list_server_moves`, and `query_mip` CLIs that talk to the Explorer backend to answer "why did/didn't the solver make this move?" — debugging tooling designed for an AI coding agent to drive.

## Design decisions

The central trade-off is **scale vs. optimality**, made explicit and user-selectable rather than hidden: local search for 1M-object problems with no guarantees, MIP for provable optimality on problems small enough. Everything else follows from making the two share one problem model — which is the hard engineering (the materializer's dual lowering, the hard/soft constraint split) and the genuine innovation here. The second trade-off is **expressiveness vs. usability**: a fixed spec catalogue is less flexible than a raw constraint language, but it makes the API "intuitive" for the dozens of non-expert teams Meta reports using it, and each new spec benefits every user. Third, the release engineering (PyPI wheels with pinned CMake 3.31, deb/rpm/Homebrew packages, a smoke-test `test_solve.cpp` in the README) shows a meta-lesson: a C++ library's usability is decided as much in its packaging as in its algorithms.

## Comparison notes

- Unlike [[Meadows — a System-Dynamics DSL in PureScript]], which compiles a small declarative language down to a simulator, Rebalancer compiles a declarative spec catalogue down to *two* interchangeable solvers — the same "model as data, engine as implementation detail" pattern applied to optimization rather than dynamics.
- Unlike [[Tork — A Distributed Workflow Engine]], which *is* the distributed control plane (queues, heartbeats, workers), Rebalancer deliberately is not distributed: a single multi-threaded process that decides placements for the systems that are. The two would compose — Tork-style engines run fleets; Rebalancer decides where the fleet's work lands.
- [[Queues Don't Fix Overload]] frames capacity as an immovable "red arrow" you must shed load against; Rebalancer is the tool for the step before that — *redistributing* load so the red arrow is hit later, and it can encode that trade-off directly (`ToFree`, `DrainCapacity`, `MinimizeMovement` specs).
- The bundled `.llms/skills/` directory makes this one of the few ingested repos that treats agent-driven debugging as a first-class deliverable, sitting alongside the Explorer UI as two audiences for the same introspection API — a concrete instance of the practices collected under [[Claude Code]].

---
*Sources: [[raw/rebalancer]], [[summary/rebalancer]]*
*Last updated: 2026-10-10*
