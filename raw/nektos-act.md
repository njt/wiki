---
url: https://github.com/nektos/act
title: nektos/act — Run GitHub Actions Locally
author: Casey Lee (cplee) and contributors
date_fetched: 2026-07-08
date_published: 2019-02-27
---

# nektos/act

Run GitHub Actions workflows locally using Docker. A Go CLI tool (~23K lines of Go) that parses `.github/workflows/*.yml`, builds a dependency-graph execution plan, and runs each job in Docker containers — letting you test CI/CD pipelines without pushing to GitHub.

## Repository Structure

```
main.go              — entry point, embeds VERSION, calls cmd.Execute()
cmd/
  root.go            — Cobra CLI: 90+ flags (platforms, secrets, envs, matrix, etc.)
  execute_test.go    — integration tests
  platforms.go        — platform image mapping
  list.go, graph.go   — workflow listing/graphing commands
pkg/
  model/
    workflow.go       — YAML parsing of .github/workflows/*.yml (Workflow, Job, Step)
    planner.go        — Plan/Stage/Run: dependency graph from `needs` declarations
    action.go         — Action model (action.yml parsing)
    github_context.go — GitHub context struct (event, repo, ref, sha, etc.)
  runner/
    runner.go         — Converts Plan to Executor chains; matrix expansion
    run_context.go    — ~1200 lines: job state container, Docker orchestration, env setup
    step.go           — step interface (pre/main/post lifecycle); file commands
    step_factory.go   — Factory: stepRun, stepActionLocal, stepActionRemote, stepDocker
    step_run.go       — Inline shell script execution
    step_action_local.go  — Local composite/custom action execution
    step_action_remote.go — Remote action (GitHub/dockerhub) fetch & run
    step_docker.go    — Docker image step execution
    action.go         — Action reading: action.yml, Dockerfile detection, trampoline
    expression.go     — ExpressionEvaluator: ${{ }} interpolation, YAML node evaluation
    command.go        — Workflow command parser (::set-env::, ::set-output::, etc.)
    job_executor.go   — Job lifecycle: setup → steps → post → cleanup
    action_cache.go   — Action content caching (bolthold/bbolt)
  exprparser/
    interpreter.go    — GitHub Actions expression language evaluator (670 lines)
    functions.go      — Functions: contains, format, join, toJSON, fromJSON, hashFiles
  container/
    container_types.go — Container interface (pull/create/start/exec/remove/close)
    docker_cli.go     — Docker CLI integration (~1160 lines): container, mount, network options
    docker_run.go     — Container run orchestration (960 lines)
    host_environment.go — Host (non-Docker) execution environment
  common/
    executor.go       — Core Executor = func(ctx) error; .Then/.Finally/.If/.IfNot
    logger.go         — Logrus-based logging utilities
    context.go        — Context helpers (cancellation, job errors)
```

## Architecture

### Execution Pipeline (4 stages)

1. **CLI** (`cmd/root.go`): Cobra-based CLI parses 90+ flags into an `Input` struct (`runner.Config`). Supports event names, job IDs, matrix filtering, secret injection, platform mapping, watch mode.

2. **Planner** (`pkg/model/planner.go`): Reads `.github/workflows/*.yml`, parses each into a `Workflow` struct (validated against a JSON schema at `pkg/schema/workflow_schema.json`), then builds a `Plan` containing `Stage`s (serial) with `Run`s (parallel jobs). Uses topological sort on `needs` dependencies — iteratively peels layers where all deps are satisfied.

3. **Runner** (`pkg/runner/runner.go`): Converts `Plan` into an `Executor` chain. Expands matrix builds (`strategy.matrix`) into individual job instances. Each run creates a `RunContext` — the central state object for a job execution.

4. **Job Executor** (`pkg/runner/job_executor.go`): Orchestrates job lifecycle: `startContainer → [step.pre()] → [step.main()] → postExecutor → [step.post()] → stopContainer → interpolateOutputs → closeContainer`. Post steps run in reverse order.

### Core Abstraction: The Executor Pattern

Defined in `pkg/common/executor.go`:
```go
type Executor func(ctx context.Context) error
```

This is a composable functional middleware. Methods on Executor provide chaining:
- `.Then(executor)` — run after success
- `.Finally(executor)` — run after always (cleanup)
- `.OnError(executor)` — run on error
- `.If(conditional)` / `.IfNot(conditional)` — conditional execution
- `NewPipelineExecutor(executors...)` — serial composition
- `NewParallelExecutor(parallel, executors...)` — concurrent execution with worker pool

The entire pipeline is built by composing these primitives. A job's steps become:
```go
NewPipelineExecutor(
    startContainer,
    InitializeNodeTool,
    NewPipelineExecutor(steps...),
    postExecutor,
    stopContainer,
    interpolateOutputs,
    closeContainer,
)
```

This is an effects system built entirely on closures — no stateful pipeline objects, no channels between stages. The `ctx context.Context` carries loggers, job errors, and cancellation signals.

### Expression Evaluator Architecture

`pkg/exprparser/` wraps the `rhysd/actionlint` library's AST parser but provides its own evaluator (actionlint only parses/lints). The `Interpreter` interface:
```go
type Interpreter interface {
    Evaluate(input string, defaultStatusCheck DefaultStatusCheck) (interface{}, error)
}
```

Key design decisions:

**Context Resolution** (`evaluateVariable`): `github`, `env`, `job`, `steps`, `runner`, `secrets`, `vars`, `strategy`, `matrix`, `needs`, `inputs` map to Go structs/maps. Property access uses reflection with `json` struct tags for field name matching.

**Type Coercion** (`coerceToNumber`, `coerceToString`): GitHub Actions has loose typing. Strings are coerced to numbers for comparisons; `""` → 0, `"true"` → 1, `"false"` → 0. If coercion fails, returns `NaN`.

**Default Status Checks**: When an `if:` expression doesn't include a status check function (`success()`, `failure()`, etc.), the interpreter wraps the user's expression with `<defaultStatus>.&& <expr>`. For `if:` at the job level, the default is `success()`; for `if:` at post steps, it's `always()`.

**Expression Rewriting** (`rewriteSubExpression`): Templates like `"Hello ${{ env.NAME }}!"` are rewritten to `format('Hello {0}!', env.NAME)`. This is a string-level transformation that converts mixed literal+expression text into a `format()` function call. The `{` and `}` escaping (`{{`, `}}`) is handled so literal braces aren't confused with format placeholders.

**YAML-Level Evaluation** (`EvaluateYamlNode`): Unlike string-substitution approaches, `act` walks the YAML AST and evaluates expressions within each node, preserving structure. It handles GitHub's undocumented features:
- **Insert directive**: `${{\ insert\ }}` in mapping keys merges nested maps
- **Sequence flattening**: If a scalar evaluates to a sequence, the items are merged into the parent sequence

### Container Subsystem

`pkg/container/Container` interface provides:
- `Pull(forcePull)`, `Create(capAdd, capDrop)`, `Start(attach)`, `Exec(command, env, user, workdir)`
- `Copy(destPath, files...)`, `CopyDir(destPath, srcPath, useGitIgnore)` — via tar archives
- `GetContainerArchive(ctx, srcPath)` — read files from container
- `Remove()`, `Close()`, `GetHealth(ctx)`, `ReplaceLogWriter(out, err)`

Two implementations:
- **Docker containers** (`pkg/container/docker_cli.go`): Full Docker lifecycle via `docker/cli` library. Adapted from Docker's own CLI code (see `DOCKER_LICENSE`). Containers are started with `tail -f /dev/null` as entrypoint to keep them alive for subsequent `docker exec` calls.
- **Host environment** (`pkg/container/host_environment.go`): When `runs-on: self-hosted`, executes directly on the host machine without Docker.

### Workflow Command Protocol

`pkg/runner/command.go` implements the GitHub Actions workflow command protocol. Commands are parsed from stdout via two regex patterns:
- GitHub Actions format: `::command key=value::message`
- Azure DevOps format: `##[command key=value]message`

Supported commands: `set-env`, `set-output`, `add-path`, `debug`, `warning`, `error`, `add-mask`, `stop-commands`, `resume-command`, `save-state`, `add-matcher`. Output variables are captured and stored in `StepResults`.

Environmental file commands (`GITHUB_OUTPUT`, `GITHUB_ENV`, `GITHUB_PATH`, `GITHUB_STATE`, `GITHUB_STEP_SUMMARY`) are handled by reading tar archives from container paths after step execution.

### Step Lifecycle

The `step` interface (`pkg/runner/step.go`):
```go
type step interface {
    pre() common.Executor
    main() common.Executor
    post() common.Executor
    getRunContext() *RunContext
    getGithubContext(ctx) *model.GithubContext
    getStepModel() *model.Step
    getEnv() *map[string]string
    getIfExpression(context, stepStage) string
}
```

Four step types:
- `stepRun`: Inline `run:` scripts — executes bash/pwsh via container exec
- `stepActionLocal`: Local `./` actions — composite actions, JavaScript actions, Docker actions
- `stepActionRemote`: Remote `owner/repo@ref` actions — cloned/fetched, then executed like local
- `stepDocker`: `docker://image` steps — pulls image and runs with entrypoint

### Action Caching

`pkg/runner/action_cache.go` uses `go.etcd.io/bbolt` (BoltDB) and `timshannon/bolthold` for content-addressed caching of remote actions. Actions are fetched once, stored by hash, and reused across runs. Offline mode (`--action-offline-mode`) skips fetch entirely.

## Key Techniques

### 1. Functional Executor Composition

Instead of an OOP step pipeline with stateful stage objects, everything composes from `func(ctx) error`. This has several advantages:
- Executors are pure middleware — no mutable pipeline state
- `.Then()`, `.Finally()`, `.If()`, `.OnError()` read like declarative specs
- Parallel execution is a one-line wrapper
- The `ctx` carries all cross-cutting concerns (logging, cancellation, job errors)

### 2. YAML-Aware Expression Evaluation

Most GitHub Actions emulators do string substitution on YAML. `act` actually walks the YAML AST (`yaml.Node` tree) and evaluates expressions within nodes, mutating them in place. This correctly handles GitHub's undocumented features (insert directives, sequence flattening) that string-substitution approaches get wrong.

### 3. Container-as-Execution-Environment

Rather than executing actions directly, `act` starts a long-running container (entrypoint: `tail -f /dev/null`), then uses `docker exec` for each step. This provides:
- Full filesystem isolation per job
- Consistent environment (matching GitHub's runner images)
- Service container support with automatic networking
- `GITHUB_OUTPUT`/`GITHUB_ENV` via filesystem (not IPC)

### 4. Expression Rewriting for Mixed Content

`rewriteSubExpression` converts `"prefix ${{ expr }} suffix"` into `format('prefix {0} suffix', expr)` by scanning for `${{ ... }}` blocks, extracting them, and building a format string. This is clever because the existing `format()` function handles the interpolation, avoiding a separate string-concatenation evaluator.

### 5. Two-Tier Evaluator Scoping

Job-level expressions and step-level expressions use different evaluators with different contexts:
- Job evaluator: has access to `needs` (outputs of dependent jobs)
- Step evaluator: has access to `steps` (current job's step results) and `inputs` (action inputs)

### 6. Matrix Build as Cartesian Product

`GetMatrixes()` computes the Cartesian product of matrix dimensions. `selectMatrixes()` filters by user-specified `--matrix` inclusions. Each matrix combination gets its own `RunContext` with a numbered suffix (e.g., `test-1`, `test-2`).

## Design Decisions

### What was optimized for

1. **Fidelity over performance**: The expression evaluator handles type coercion, YAML-level mutations, insert directives, and sequence flattening — all to match real GitHub Actions behavior exactly, not just the common cases.

2. **Composability over simplicity**: The Executor pattern is elegant but requires understanding functional composition. A simpler approach (sequential for-loop over steps) would be more approachable but less flexible.

3. **Docker-as-platform over portability**: `act` requires Docker (or a Docker-compatible runtime). The `-self-hosted` mode exists but is clearly secondary. This is a reasonable trade-off — Docker provides the isolation and environment fidelity that makes `act` trustworthy.

### What was sacrificed

1. **Startup latency**: Each job starts a full container. There's no warm pool or caching of container state between runs (beyond `--reuse`).

2. **Cross-platform parity**: Windows support exists but is secondary. The container subsystem has Linux-specific extensions and assumes Unix paths in many places.

3. **Plugin extensibility**: The expression language and step types are hard-coded. There's no hook for adding custom functions or step types without modifying the source.

### Non-obvious design choices

1. **BoltDB for action caching**: Using an embedded key-value store (bbolt) rather than the filesystem for caching action metadata is unusual. It provides content-addressed lookup and transactional consistency, but adds a dependency and complexity.

2. **Adapting Docker CLI code**: Rather than using the Docker SDK directly, `act` adapts code from `docker/cli` itself. The `docker_cli.go` file includes ported option-parsing from the Docker CLI, which means `act` understands Docker's full container creation flags natively.

3. **`tail -f /dev/null` as container keep-alive**: Instead of managing a process lifecycle, `act` starts a container with a no-op entrypoint, then `docker exec`s into it. This is simpler than orchestrating process startup/shutdown but means the container process is always present.

## Comparison Notes

Unlike `nektos/act` which runs full workflows in Docker containers with expression evaluation, most CI testing tools either:
- Parse/lint YAML without execution (e.g., `actionlint`, which `act` actually depends on for parsing)
- Execute but don't emulate the GitHub Actions runtime (e.g., simple shell scripts that wrap `docker run`)
- Run on GitHub's infrastructure rather than locally

`act` is the most complete local GitHub Actions emulator, primarily because of its expression language evaluator — type coercion, YAML-level mutation, and context resolution match GitHub's actual behavior rather than a simplified subset.

The architecture closely mirrors how GitHub Actions runners work internally: a job container, step exec within the container, file-based command protocol for output/state/path, and post-step cleanup. The main gap is `act` doesn't implement the full runner protocol (job registration, result reporting to GitHub's API).

In the broader CI/CD space, this is a **localism** approach (run CI on your laptop) vs. the dominant **remotism** approach (push to a CI server). The trend toward local-first developer tools (local LLM inference, local databases) makes `act` more relevant, not less — it's part of the "shift left" movement, moving verification earlier in the development loop.

## Key Stats

- ~23K lines of Go (excluding tests)
- 90+ CLI flags
- 14 expression functions
- 4 step types
- 2 container backends (Docker, host)
- Depends on `rhysd/actionlint` for expression parsing, `moby/moby` for Docker API, `go-git/go-git` for repository metadata
