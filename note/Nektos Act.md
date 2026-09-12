# Nektos Act

`act` is the definitive tool for running GitHub Actions workflows locally. Written in Go (~23K lines), it parses `.github/workflows/*.yml`, builds a dependency-graph execution plan, and runs each job inside Docker containers — letting you test CI/CD pipelines on your laptop without pushing to GitHub. It's the most complete local emulation of the GitHub Actions runtime available, primarily because it implements a full expression language evaluator with type coercion, YAML-level mutation, and the same workflow command protocol as GitHub's hosted runners.

---

## Architecture

`act` follows a four-stage pipeline:

1. **CLI** (`cmd/root.go`) — Cobra-based command with 90+ flags (platforms, secrets, env vars, matrix filtering, watch mode, etc.)
2. **Planner** (`pkg/model/planner.go`) — Parses workflow YAML (validated against a JSON schema), builds a `Plan` of `Stage`s (serial) containing `Run`s (parallel jobs), topologically sorted by `needs` dependencies
3. **Runner** (`pkg/runner/runner.go`) — Expands matrix builds into individual job instances, each wrapped in a `RunContext` — the central state object that holds env vars, step results, container references, and expression evaluators for a single job execution
4. **Job Executor** (`pkg/runner/job_executor.go`) — Orchestrates the job lifecycle: start container → pre-steps → main steps → post-steps (in reverse) → cleanup

The core abstraction is the **Executor pattern** (`pkg/common/executor.go`): `type Executor func(ctx context.Context) error`. This is a composable functional middleware — `.Then()`, `.Finally()`, `.OnError()`, `.If()` chain executors declaratively. `NewPipelineExecutor` composes serial chains; `NewParallelExecutor` fans out with a worker pool. The entire pipeline (container lifecycle, step execution, cleanup, error handling) is built by composing these primitives. It's an effects system built entirely on closures — no stateful pipeline objects, no channels between stages.

## Key Techniques

### Expression Language Evaluator

`pkg/exprparser/` wraps `actionlint`'s AST parser but provides its own evaluator (`interpreter.go`, 670 lines). It handles all GitHub Actions contexts (`github`, `env`, `job`, `steps`, `secrets`, `vars`, `strategy`, `matrix`, `needs`, `inputs`) via reflection-based property access using `json` struct tags.

The type coercion system is particularly notable: strings are coerced to numbers for comparisons (`""` → 0, `"true"` → 1, `"false"` → 0), and failed coercion returns `NaN`. Default status checks wrap user expressions — `if: someCondition` at job level implicitly becomes `success() && someCondition`.

Expression rewriting (`expression.go`) converts templates like `"Hello ${{ env.NAME }}!"` to `format('Hello {0}!', env.NAME)` — a clean approach that reuses the existing `format()` function rather than building a separate string-concatenation evaluator.

### YAML-Level Evaluation

Unlike tools that do string substitution on YAML, `act` walks the `yaml.Node` AST and evaluates expressions within each node, preserving structure. This correctly handles GitHub's undocumented features: the `${{\ insert\ }}` directive (merging nested maps) and sequence flattening (when a scalar evaluates to a sequence, items merge into the parent).

### Container-as-Execution-Environment

Rather than running actions directly, `act` starts a long-running container with `tail -f /dev/null` as entrypoint, then uses `docker exec` for each step. This provides full filesystem isolation, consistent environments, service container support with automatic networking, and file-based workflow commands (`GITHUB_OUTPUT`, `GITHUB_ENV`, `GITHUB_PATH`) via tar archive reads from container paths.

### Workflow Command Protocol

`pkg/runner/command.go` parses both GitHub Actions (`::command::`) and Azure DevOps (`##[command]`) command formats from step stdout. File-based commands (`GITHUB_OUTPUT`, `GITHUB_ENV`, `GITHUB_PATH`, `GITHUB_STATE`, `GITHUB_STEP_SUMMARY`) are read from container paths after each step executes.

### Step Lifecycle

The `step` interface defines three phases: `pre()`, `main()`, `post()`. Four step types implement this: inline shell scripts (`stepRun`), local actions (`stepActionLocal`), remote actions fetched from GitHub (`stepActionRemote`), and Docker image steps (`stepDocker`). Post-steps run in reverse order so cleanup cascades correctly.

## Design Decisions

**Fidelity over performance**: The expression evaluator handles edge cases (type coercion, insert directives, sequence flattening) to match real GitHub Actions exactly, not just common cases. This correctness-first approach is what distinguishes `act` from simpler YAML linters.

**Composability over simplicity**: The Executor pattern requires understanding functional composition, but enables declarative pipeline construction. A for-loop over steps would be more approachable but couldn't express the conditional/finally/parallel composition that workflows need.

**Docker-as-platform**: `act` requires Docker. The `-self-hosted` mode (direct host execution) exists but is secondary. Docker provides the isolation that makes `act`'s emulation trustworthy — environments match GitHub's runner images.

**Adapting Docker CLI code**: Rather than using Docker's SDK, `act` ports option-parsing from `docker/cli` itself (`pkg/container/docker_cli.go`). This means `act` understands Docker's full container creation flags natively, including volume mounts, capabilities, device rules, and CDI device injection.

## Comparison Notes

Unlike related tools in the CI/CD and local-development space:

- **vs. `actionlint`** (which `act` depends on): `actionlint` parses and lints workflow YAML but doesn't execute. `act` uses `actionlint`'s expression parser as a library and builds execution on top of it.
- **vs. simple shell wrappers**: Most "run actions locally" tools skip expression evaluation or type coercion, making them unreliable for real workflows. `act`'s evaluator is the differentiator.
- **vs. hosted runners**: GitHub's own runners push results to a central API; `act` instead displays results in the terminal. The fundamental architecture (job container, docker exec per step, file-based commands) is the same.

`act` is part of the broader "shift left" movement — moving verification earlier in the development loop. In the context of [[Agent Coding Workflow]], tools like `act` that enable fast local feedback loops directly improve the quality of AI-generated CI/CD changes. An agent that can test workflow changes locally before pushing them will produce fewer broken CI runs.

---

*Sources: [[raw/nektos-act]]*
*Last updated: 2026-07-08*
