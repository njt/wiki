# Tork — A Distributed Workflow Engine

Tork (github.com/runabol/tork) is a ~15K-line Go workflow engine by Arik Cohen: a job is an ordered list of tasks, each task runs in its own container, and one binary plays all three roles — standalone, coordinator, or worker. It matters here because it is a complete, readable implementation of the orchestration layer this wiki keeps describing abstractly: queues, schedulers, heartbeats, retries, and a leaderless control plane, all in code you can read in an afternoon. It is the concrete workflow-engine layer that an agent fleet's task queue would sit on, and its answers to crash recovery, concurrency throttling, and secret handling are transferable to agent orchestration directly.

---

## Architecture

**One binary, three modes** (`engine/engine.go`). `ModeStandalone` wires a worker and a coordinator into one process over an in-memory broker; `ModeCoordinator` and `ModeWorker` split them for scale-out. The whole distributed system is embedded in the standalone process, which is also how the test suite exercises cluster behavior.

**Leaderless coordinators** (`internal/coordinator/coordinator.go`). There is no leader election and no scheduler loop. Every coordinator instance subscribes to the same fixed set of queues from `broker/queues.go`: `pending`, `started`, `completed`, `error`, `heartbeat`, `jobs`, `logs`, `progress`, `redeliveries`. RabbitMQ distributes messages across competing consumers; whoever gets the message processes it. "No single point of failure" is achieved by having no dispatcher at all.

**State lives in the datastore** (`datastore/datastore.go`, `datastore/postgres/`). The `Datastore` interface's signature is the design: `UpdateTask(ctx, id, mutate func(*Task) error)` — functional updates run inside `WithTx` transactions with state guards ("can't complete task X because it's FAILED"), which is how competing coordinators avoid clobbering each other without locks.

**Workers are thin** (`internal/worker/worker.go`). A worker subscribes N concurrent consumers per configured work queue, wraps task execution in the task-middleware chain, publishes `started`/`completed`/`error` events, and exposes a private health API on ports 8001–8100. Cancellation arrives out-of-band on an exclusive `x-<workerID>` queue and cancels the task's context.

**Handlers per event** (`internal/coordinator/handlers/`). One small handler per state transition — `pending.go` schedules or skips, `completed.go` advances the job, `error.go` retries or fails, `redelivered.go` recovers — each wrapped by `task.ApplyMiddleware` and an error-handler that turns handler errors into task failures.

## Key techniques

- **Progressive task materialization** (`handlers/completed.go`). A job definition is a template; execution is task rows. When task N completes, the handler increments `Job.Position` in one transaction, then creates *only* task N+1, evaluating its `{{ expr }}` templates against the accumulated job context. A 1,000-task job stores one task row until it runs. Job progress is literally `Position/TaskCount`.
- **Composites as parent IDs** (`coordinator/scheduler/scheduler.go`). `parallel`, `each`, and `subjob` are just fields on one `Task` struct (`task.go`); the scheduler dispatches on which is non-nil. Children get `ParentID` rows; the parent completes when its completion counter reaches its size.
- **A database-backed concurrency semaphore** (`handlers/completed.go` + `postgres.go:928`). `each` with `concurrency: 3` creates all child tasks in `CREATED` state but only fires the first 3. Each child completion transaction bumps `Each.Completions`, calls `GetNextTask` (`SELECT * FROM tasks WHERE parent_id = $1 AND state = 'CREATED' LIMIT 1`), and promotes it to `PENDING`. Slot release is a side effect of the completion commit — no separate rate limiter, no race window.
- **The `/tork` volume trick** (`runtime/docker/tcontainer.go`). The task's `run` script is written into a hidden `/tork` volume as an executable `entrypoint` (mode 0555, copied in as a tar archive), and that file is the container's default CMD. `$TORK_OUTPUT=/tork/stdout` and `$TORK_PROGRESS=/tork/progress` are files in the same volume; on exit the worker reads them back via `CopyFromContainer`. So the structured result travels through the filesystem while stdout is streamed to logs in parallel (`dockerLogsReader` strips Docker's 8-byte stream headers by hand).
- **A serial image puller with TTL pruning** (`runtime/docker/docker.go`). All pulls funnel through a one-slot channel processed by a single goroutine — no concurrent pull races. A last-used map plus an hourly pruner removes images idle past 72 hours, but only when zero tasks are running. An optional verification step creates a throwaway container running `true` to prove the image works, deleting it if not.
- **Middleware as the extension seam** (`engine/engine.go`, `middleware/`). Five chains — web, job, task, node, log — plus registrable brokers, datastores, runtimes, and mounters. Redaction is job/log middleware: default matchers are substrings (`SECRET`, `PASSWORD`, `ACCESS_KEY`), and a value appearing more than once in a log part redacts the whole line rather than mangling it.
- **Recovery by broker redelivery** (`handlers/redelivered.go`). A crashed worker's unacked task comes back redelivered; the handler increments a counter, re-publishes to the task's own queue, and fails the task after 5 redeliveries. Retries are different: `error.go` clones the task into a *new* task row with `Retry.Attempts++` — history is preserved, nothing is mutated in place.
- **Cron with an anti-drift lock** (`handlers/schedule.go`). Scheduled jobs run through gocron on every coordinator, guarded by a distributed `locker.Locker` (in-memory or Postgres). The lock wrapper enforces a minimum 10-second hold on release so a clock-drifting coordinator can't instantly re-acquire and double-fire.
- **Secrets encrypted at rest** (`internal/encrypt/encrypt.go`). AES-GCM with a SHA-256-derived key from a configured passphrase — functional, though the key derivation is weak by modern standards.

## Design decisions

**At-least-once over exactly-once, with container idempotency as the backstop.** A task completed-but-unacked will re-execute; Tork accepts this because each task is a fresh container doing side-effecting work the user chose. The fresh-container-per-task rule is the idempotency story, stated in the README and honored in the code.

**No dispatcher, so no dispatcher to crash.** The cost of leaderlessness is that every state transition is a Postgres transaction on the coordinator path — correctness is bought with database round-trips rather than in-memory scheduling. For Tork's task sizes (containers, seconds-to-minutes each) that's the right trade.

**Strict sequence by default.** A job is a linear list; task N+1 blocks on N. Parallelism is opt-in via `parallel`/`each` composites rather than a DAG engine. This keeps the mental model trivial (a YAML list) at the cost of Airflow-style dependency graphs.

**The coordinator is in the trusted core; containers are the arbitrary-code boundary.** Expression evaluation (`internal/eval/eval.go`, the expr language) runs on the coordinator at scheduling time. Job authors get a real expression language without it ever executing inside the control plane as shell — arbitrary code only ever runs in the task container, with CPU/memory limits, `privileged: false` by default, and bind mounts disabled unless explicitly allowed (`mounts.bind.allowed = false`).

**The shell runtime is a documented footgun** (`runtime/shell/`), included for convenience but warned against, with optional uid/gid pinning via `setid`.

## Comparison notes

- Unlike [[Paperclip]], which is a control plane for *agent CLIs* with org-chart governance and budgets, Tork is the layer underneath: the same shape (heartbeats, queues, a web UI over fleet state) applied to plain containerized work. An agent fleet that outgrew a shell-script scheduler would rebuild something Tork-shaped.
- Unlike [[Fleet Supervisor (sermakarevich)]], whose beads queue lives inside one Python process, Tork externalizes the queue into a real broker (RabbitMQ with `x-max-priority` honored per task), which is what lets workers scale horizontally and survive coordinator restarts.
- Read next to [[Durable Execution Without History Replay]], Tork's recovery story is the classic position the essay argues against: broker redelivery plus task rows in Postgres, no event-sourced replay and no durable continuations — a completed-but-unacked task simply re-runs, and container idempotency papers over the gap.
- And next to [[We Built a Scalable Agent Sandbox]], Tork shows the same containment instincts — one container per unit of work, resource limits, no privileged mode, bind-mount allowlists — implemented as a workflow primitive a decade before agent harnesses needed them for sandboxing.

---

*Sources: [[raw/tork]], [[summary/tork]]*
*Last updated: 2026-09-15*
