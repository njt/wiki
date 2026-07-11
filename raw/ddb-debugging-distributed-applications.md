---
url: https://arxiv.org/abs/2607.06107
title: "DDB: Source-Level Interactive Debugging for Distributed Applications"
author: Yibo Yan, Junzhou He, Seo Jin Park
date_fetched: 2026-07-11
date_published: 2026-07-07
---

# DDB: Source-Level Interactive Debugging for Distributed Applications

**Authors:** Yibo Yan, Junzhou He, Seo Jin Park — University of Southern California

**License:** CC BY 4.0 | **arXiv:** 2607.06107v1 [cs.DC] 07 Jul 2026

## Abstract

Interactive debugging helps developers understand program behavior at the source level by pausing execution, navigating call stacks, and inspecting runtime state. But traditional interactive debuggers only work for single-process execution, and have been considered impractical for distributed systems. Call stacks stop at process boundaries, debugging state doesn't survive infrastructure churn, and debugger-induced pauses trigger timeout cascades that destroy the debugging flow. Developers resort to iterative log-and-redeploy cycles instead of live hypothesis testing.

DDB extends interactive debugging to distributed applications through three solutions. Distributed Backtrace (DBT) embeds causality metadata in every RPC and reconstructs unified call stacks across RPC boundaries. An intent-preserving control plane coordinates breakpoints across dynamic process sets. Pause-Erased Time (PET) virtualizes each process's clock, decoupling logical time from physical pauses. DDB integrates with RPC frameworks in 10–60 lines of code. Evaluated on gRPC, ServiceWeaver, Nu, and Quicksand across up to 122 processes, DDB achieves "30 ms median cross-RPC backtrace latency, sub-5 ms time jump under repeated execution pauses, and adds 1–5% throughput overhead." In a user study, DDB achieved 100% fault localization compared to 38.5% for baseline tools, with median localization time around 8 minutes.

## 1. Introduction

Modern software decomposition — spanning serverless platforms, microservices, actor frameworks, distributed application runtimes, and modular programming frameworks — means code now executes across multiple services and machines. Application developers who may lack deep distributed systems expertise routinely write code spanning dozens of processes. The barriers to adopting distributed programming have dropped substantially while the debugging challenges have grown.

For single-process debugging, developers attach a debugger like GDB or LLDB, set breakpoints, and inspect call stacks and variables — a tight hypothesis-inspect-refine loop. Once an application becomes distributed, this workflow collapses. A breakpoint in one process reveals nothing about remote callers. Attaching separate debuggers to every upstream process doesn't scale, and pausing any process triggers leader re-elections, transaction aborts, or cluster-wide crash recovery because distributed systems rely on failure-detection timeouts as short as 50–100 ms.

Traditional large-scale infrastructure teams mitigate this with heavy investment in structured logging, distributed tracing (Jaeger, OpenZipkin, OpenTelemetry), and custom monitoring. Application developers lack access to such resources and fall back to iterative log-and-redeploy cycles that can consume days to weeks per fault. In the user study, participants using baseline tools (GDB, distributed tracing) "failed to localize cross-service faults in 61.5% of cases and often exceeded the 20-minute time limit."

Each existing tool addresses a different aspect of distributed diagnosis, but none provides the interactive cross-RPC, source-level fault localization workflow. DDB enables developers to pause running distributed execution, navigate cross-RPC call chains, and examine live runtime state across service boundaries.

The paper makes four contributions:

1. **Distributed Backtrace (DBT):** Cross-RPC stack reconstruction transparent to user-level threads, enabling live inspection of caller state across RPC boundaries.
2. **Intent-Preserving Control Plane:** Debug-intent propagation across replica scaling, node churn, and computation migrations.
3. **Pause-Erased Time (PET):** Time virtualization via Virtual Deadline Enforcement that decouples physical and logical time.
4. **Implementation and Evaluation:** Integration with four RPC frameworks in two languages (20–60 LoC each), evaluation at 122-process scale.

## 2. Background and Motivation

### 2.1 The Developer Workflow and the Missing Capability

The paper presents a detailed scenario: a developer debugging a distributed key-value store where read requests return stale values after writes. The developer can reproduce the issue and use distributed tracing to identify the request path, but understanding the root cause requires inspecting runtime state no log captured — version identifiers in RPCs, comparator evaluations, cache slot targeting. In single-process development, GDB answers these questions directly. For distributed applications, no equivalent exists. The developer falls back to iterative logging: instrument handlers, redeploy, re-run. Each round reveals only specifically logged variables. After several cycles over extended time, the developer might discover the root cause — something a single interactive cross-service inspection could have reached in minutes.

### 2.2 Entrypoints for Interactive Debugging

Interactive debugging begins at entrypoints like manual breakpoints, watchpoints, assertion failures, or memory faults. In distributed settings, catching the event isn't sufficient because the root cause often resides in an upstream remote caller or manifests across auto-scaling replicas. Every entrypoint requires reconstructing the full cross-process execution context — the chain of remote callers, their arguments, and their runtime state.

### 2.3 Why Interactive Debugging Breaks Across Process Boundaries

Three specific challenges prevent interactive debugging across process boundaries:

**① Call stacks stop at the process boundary.** If a request traverses services A → B → C and a breakpoint fires in C, GDB reveals only C's local stack. Recovering context from B or A requires manually attaching separate debuggers and identifying the correct thread among potentially hundreds. User-level threading (e.g., goroutines) compounds the difficulty because the goroutine that sent an RPC may have been descheduled.

**② Debugging operations require per-process manual effort.** In a 180-process deployment, developers must manually identify relevant replicas and insert breakpoints individually. The process set changes constantly due to auto-scaling, rolling restarts, and computation migrations. Unless breakpoints automatically follow these changes, debugging coverage is silently lost.

**③ Execution pauses cause timeout cascades.** The paper surveyed timeout thresholds in LogCabin (a Raft implementation), RAMCloud, and Nu. "Thresholds range from 100 ms (RPC retry) to 500 ms (election timer)." Pausing at a breakpoint for even a few seconds triggers every timeout in this range, causing consequences from extra heartbeats to aborted transactions and cluster-wide crash recovery. This breaks the intended debug flow.

## 3. Overview: A DDB Debugging Session

A live debugging session on a 3-node Raft cluster illustrates DDB in action. The developer investigates why followers improperly reject the leader's heartbeats.

**Setting a breakpoint across replicas:** The developer sets a single breakpoint at the AppendEntries RPC handler. DDB's intent-preserving control plane automatically applies it to all follower replicas without requiring per-process attachment.

**Pausing the cluster safely:** When both followers hit the breakpoint, DDB pauses the entire cluster. Without DDB, Raft's election timeouts would trigger a spurious leader election. PET virtualizes each process's clock so failure detectors remain unaware of the pause.

**Inspecting the cross-RPC call chain:** The developer sees a Distributed Backtrace extending from the follower's handler back to the leader's send_heartbeat caller frame. Clicking the upstream leader frame reveals the exact arguments sent across the network.

**Granular per-process control:** The developer single-steps one follower while the other stays suspended, evaluating expressions in the Watch pane and comparing state across replicas to find the root cause.

## 4. DDB Debugging Model and Design

DDB is organized around three pillars: Distributed Backtrace (DBT), an intent-preserving unified control plane, and Pause-Erased Time (PET).

### 4.1 Distributed Backtrace (DBT)

**Abstraction:** When execution pauses, the developer issues a `dbt` command. DDB returns a unified call stack from the current frame back across each RPC boundary to the request root. For a request traversing A → B → C, pausing in C and issuing `dbt` presents frames from C, then B, then A. The developer can navigate any frame (including those in remote processes) to inspect local variables, read heap objects, and modify values.

**Mechanism:** Two problems must be solved: locating the correct caller context in a remote process and presenting a coherent unified stack.

**Caller-context capture design:** A strawman approach would record the caller's OS thread ID at RPC send time. This fails with user-level threads (goroutines, CILK workers, Folly fibers, Shenango, Caladan) because the calling uthread may be rescheduled off its original OS thread between send and callee pause. DDB instead captures thread context (register values identifying the stack) at the call site and embeds it in the RPC payload as compact caller-context metadata. This is the only RPC protocol change and requires approximately 20 LoC per framework. At the callee process, the RPC handler extracts this metadata and stores it in an injected extraction frame on the stack.

**Cross-process stack reconstruction:** When `dbt` is issued at process C, DDB locates the extraction frame, reads metadata identifying B by IP and PID, then contacts B's debugger agent out-of-band to fetch stack memory segments. The agent temporarily restores B's thread context and performs standard DWARF stack unwinding using debug symbols. This repeats until no further caller-context metadata exists. All frames are concatenated into a single stack.

**Guarantees:** Completeness (cross-RPC caller chain reconstructed to root), Inspectability (every frame exposes locals, arguments, and heap objects), and Order preservation (frames appear in reverse request-path order).

**Edge cases:** If a caller has terminated, DBT reports a truncated backtrace with annotation. If an upstream framework lacks DDB integration, DBT stops at that boundary.

### 4.2 Intent-Preserving Unified Control Plane

**Abstraction:** A debug-intent is defined as a triple: a source-code location, a scope identifying target processes, and a debugger command. The control plane resolves each intent against the current application topology and propagates it to every matching process. By default, the control plane enforces a pause-the-world policy — when a breakpoint triggers a pause in one process, the control plane broadcasts a pause command to all other attached processes.

**Logical Groups and Scope Resolution:** DDB organizes processes into logical groups (by default, same binary = same group). In a 180-process SocialNet deployment with 36 services at 5 replicas, the developer sees 36 groups. DDB performs automatic scope resolution by identifying which logical groups map a given source file. An internal two-level source-map maps source-code paths to logical groups and logical groups to running processes.

**Intent Preservation Under Dynamics:** The control plane maintains a continuous invariant: an intent is active on a process if and only if the process matches the intent's logical scope and its loaded binary maps the specified source-code location. When a new process joins, the control plane evaluates this invariant and applies relevant intents immediately. When a process terminates, its debugging state is garbage-collected. For computation migration frameworks (Nu, Quicksand, ServiceWeaver), breakpoints follow the computation automatically because the control plane targets logical scopes rather than physical processes. Heap state handling uses framework-specific plugins.

**Guarantees:** Invariant enforcement (no matching process misses an intent, no out-of-scope process receives one), minimal developer expression (source-file and logical-group granularity), and composability with other DDB primitives.

### 4.3 Pause-Erased Time (PET)

**Problem:** Every debugger pause advances the system clock while the application is stopped. From the application's perspective, time jumps by the pause duration. Distributed systems rely on strict physical timing invariants — missed heartbeats trigger elections, late RPCs trigger retry storms, expired leases trigger disconnections. A ten-second pause triggers every timeout below that threshold.

**Abstraction:** PET gives each debugged process a pause-erased view of time where all debugger pauses are invisible. PET covers three time semantics: reading the current timestamp (get_time), sleeping until an absolute deadline (sleep_until), and sleeping for a relative duration (sleep). These map to POSIX APIs like clock_gettime, gettimeofday, pthread_cond_timedwait, sem_timedwait, sleep, and nanosleep.

**Pause-offset Accumulation:** DDB records a timestamp when a process is paused and another when resumed. The difference (measured pause duration) is accumulated into a running cumulative pause offset. Before resuming, DDB publishes this offset to a local shim layer (interposed via LD_PRELOAD) that intercepts POSIX time API calls and subtracts the cumulative offset from return values.

**Virtual Deadline Enforcement:** A subtler problem arises for processes sleeping when paused. On resume, the kernel may wake the thread immediately because the real-time deadline has passed — before the pause-adjusted deadline. PET's shim layer tracks the pause-adjusted absolute deadline for every active timer wait. On every return from sleep, the shim reads current PET. If PET is earlier than the adjusted deadline, the shim re-arms the wait for the remaining duration. Application code doesn't execute until the PET deadline is genuinely reached.

"This is a fundamental requirement for interactive distributed debugging" — without it, dynamically erasing pauses would fix get_time but fail to prevent timeout cascades from in-flight sleeps.

**Guarantees:** Two invariants: Monotonicity (PET never decreases) and Timer correctness (time-blocking operations return only when PET reaches their target). These ensure application logic correct with respect to real time remains correct with respect to PET.

**Limitations:** PET maintains virtual time only within the attached cluster. External systems relying on physical time (uninstrumented cloud storage, third-party API gateways) will perceive pauses as genuine timeouts. Developers must mock time-sensitive external dependencies or attach the load-generating client to the control plane.

## 5. Implementation

DDB is implemented on x86_64 Linux with experimental aarch64 support.

**System Architecture:** DDB comprises a centralized coordinator, a graphical frontend (VS Code Extension), and per-process distributed components. The VS Code Extension communicates with the Centralized Coordinator, which uses ServiceDiscover to track dynamic process topologies and reports them to DCore (the central engine). DCore orchestrates debugging sessions and distributes coordination messages to Distributed Processes. Each target process is paired with a DKnot agent (an extended GDB) that attaches to the process, relays debug-intents, and implements Distributed Backtrace. DKnot sends offset adjustments to a locally interposed shim layer that enforces PET via LD_PRELOAD. A lightweight DConnector integration library embeds caller-context metadata within RPC payloads.

**Interaction Between Components:** During normal execution, DCore maintains coordination with all DKnot agents. When a new process spins up, ServiceDiscover detects it and updates the global topology. DCore initializes a new DKnot instance that attaches to the new process. When a process hits a breakpoint, DCore pauses the world, DKnot agents calculate pause timestamps, and on resume each DKnot publishes the cumulative offset to its shim layer.

**Codebase:** DDB supports gRPC (~20 LoC C++), Nu (~30 LoC C++), Quicksand (~60 LoC C++), and ServiceWeaver (~10 LoC Go). Total DDB code is approximately 22,196 LoC (DCore: 14,867 Rust, DKnot: 1,401 Python, DConnector: 1,954 C/C++/Go, Adapter: ~700 TypeScript). The socialnet benchmark was ported to ServiceWeaver (3,154 LoC Go).

### 5.1 Limitations

**Programming Language Support:** The prototype assumes GDB as the underlying debugger, so it doesn't work for binaries GDB cannot interpret.

**PET:** DDB only provides PET on top of POSIX time APIs, not for raw rdtsc or NIC hardware timestamps.

**In-flight Packets During Pauses:** DDB relies on host OS TCP receive buffers to queue in-flight packets. Under pause-the-world behavior, clients and upstream services are also paused, so TCP buffers absorb in-flight traffic without exhausting capacity, assuming no unmanaged external traffic.

## 6. Evaluation

The evaluation answers five key questions about integration effort, diagnostic reliability, responsiveness, PET effectiveness, and runtime overhead.

**Setup:** Responsiveness was evaluated on a 24-server Chameleon cluster (Intel Xeon Gold 6240R, 24 cores, 2.4GHz, 187GB RAM). PET effectiveness on a local server (Intel Xeon Gold 5420+, 28 cores, 2.00 GHz, 256GB RAM). Metadata overhead on 12 CloudLab servers (16-core AMD 7302P at 3.00GHz, 128GB RAM, Mellanox ConnectX-5 NIC).

**Applications:** SocialNet microservice benchmark on ServiceWeaver, Nu, and Quicksand; a C++ Raft consensus implementation for gRPC; and a synthetic application for PET isolation.

### 6.1 User Study

A two-part controlled user study was conducted.

**Study 1: Integration Effort** — 5 graduate researchers integrated DDB and OpenTelemetry into a C++ codebase. Two goals: tracing execution backward from callee to caller with local state visible (G1), and observing concurrent fan-out (G2). All 5 participants completed both goals with DDB (median 400 seconds). Only 1 completed G1 with OpenTelemetry and none completed G2 within the 20-minute limit.

**Study 2: Diagnostic Efficacy** — A within-subjects design with 9 participants comparing DDB against GDB and OpenTelemetry across three bug cases, partially counterbalanced via Latin Square. All participants had access to structured logging and traffic replay.

The three test cases were: (1) a non-deterministic segmentation fault across two of three Raft processes, (2) a logic error where a stale leader fails to update its term after network partition recovery, and (3) a distributed deadlock from concurrent leader elections. All required reasoning across multiple Raft processes (3–5 nodes) with timing-sensitive protocol behavior.

DDB enabled 100% localization across all trials and 89% fix rates within the 20-minute limit. GDB achieved 44% localization and 22% fix rates. OTel achieved 33% and 11%, respectively. DDB's median localization time was approximately 500 seconds across all bug cases.

**Qualitative analysis:** Participants using OTel spent disproportionate time on instrumentation and navigation rather than diagnostic reasoning. GDB worked for single-process faults but didn't scale — at five processes participants could no longer efficiently manage threads. In timing-sensitive cases, pausing for inspection triggered RPC timeouts that invalidated cluster state, forcing session restarts.

### 6.2 Responsiveness

The paper reports end-to-end latency for `dbt` and `continue` commands across cluster sizes of 38, 62, and 122 processes.

The `continue` command finishes under 5 ms because debug-intent distribution is heavily parallelized. The `dbt` command latency remains under 200 ms at the tail even at 122 processes, where over 14,600 concurrent requests are triggered (one for every thread across every paused process).

**Call Depth:** DBT latency is bounded by call depth. Reconstruction takes a median of 48 ms at two hops, 121 ms at three hops, 173 ms at four hops, 217 ms at five hops, 259 ms at six hops, and 435 ms at ten hops. A recent study reports typical cloud application call depths average around 3 to 4, keeping sub-second latency within interactive thresholds.

### 6.3 PET Effectiveness

**Overhead:** The PET shim layer introduces nanosecond-scale overhead per intercepted time API call (95 ns for gettimeofday, 88 ns for clock_gettime(REALTIME), 89 ns for clock_gettime(MONOTONIC) with PET vs. 29, 28, and 28 ns without). The cumulative offset calculation takes 1.265 μs on average.

**Masking Time Jumps:** A synthetic application running a tight loop (~50 μs per iteration) was repeatedly paused and resumed with 100 ms intervals. The application consistently perceived ≤5 ms time jumps — well below the 100 ms interval.

**Real-World Safety Margin:** The paper surveyed timeout thresholds in LogCabin, RAMCloud, and Nu. The most aggressive thresholds range from 50 ms to 250 ms. PET's ≤5 ms maximum drift provides an order-of-magnitude safety envelope.

The survey found thresholds including LogCabin's heartbeat timeout at 250 ms (low severity — excessive heartbeats) and election timeout at 500 ms (moderate severity — disrupted leadership). RAMCloud's RPC timeout at 100–200 ms (low), crash recovery trigger at 250 ms (severe — initiates large-scale recovery), and transaction timeout at N×50 ms (severe). Nu's full shard probing at 400 ms (moderate), connection pool timeout at 10 s (severe), and service keepalive at 10 s (severe).

### 6.4 DDB Runtime Overhead on Applications

DDB introduces minimal throughput degradation. SocialNet on ServiceWeaver showed 0.6% throughput drop; on Nu, 3.8%. Attaching GDB to gRPC Raft yielded 18.4% degradation; attaching DDB to the same gRPC Raft yielded 20.1% — demonstrating that "DDB achieves distributed debugging capabilities with overhead comparable to a traditional single-process debugger." The caller-context metadata adds only 52 bytes per RPC payload.

## 7. Related Work

**Formal methods and model checking:** Formal verification (Verdi, IronFleet) and model checking provide strong correctness guarantees but operate on mathematical models, not running code. Runtime verification checks specifications against live executions but doesn't enable interactive inspection.

**Record and Replay:** Tools like Friday and Replay Debugging capture non-determinism for post-hoc analysis. Record-and-replay is orthogonal to DDB — it targets non-deterministic bugs via replayed execution, while DDB operates on live, real-time execution.

**Distributed logging, tracing, and monitoring:** Frameworks like Azure Monitor, Cloud Logging, LogDevice, Jaeger, OpenZipkin, OpenTelemetry, Dapper, Pivot Tracing, and Canopy enhance observability by aggregating logs, establishing causality, and enabling tracing-based analysis. They're well-suited for identifying which service is behaving anomalously but don't provide source-level interactive debugging. DDB addresses a different need: once a developer knows where a problem occurs, they need to understand why by pausing execution and inspecting live state. Unlike tracing, DDB's metadata isn't sampled and produces no persistent records.

**HPC/Distributed debuggers:** TotalView and Linaro DDT support multi-process debugging for HPC's homogeneous execution model but don't reconstruct call stacks across RPC boundaries or protect against timeout interference. Ray's built-in debugger provides a single-point view of distributed tasks but lacks distributed backtrace and requires manual cross-referencing.

**Classic distributed debugging:** Early research focused on consistent global snapshots (Chandy and Lamport), distributed halting (Miller and Choi), and causal ordering (Fowler and Zwaenepoel). Modern cloud-native architectures shift the problem space to deep RPC chains, dynamically scaling replica sets, and strict timeout-driven availability. DDB addresses practical developer workflow bottlenecks with an intent-preserving control plane that manages debug state across ephemeral infrastructure.

## 8. Conclusion

Interactive debugging has been widely treated as impractical for distributed systems due to call stacks stopping at process boundaries, debugging state failing to survive infrastructure dynamics, and physical debugger pauses triggering timeout cascades. DDB restores the tight hypothesis-testing loop through three user-space mechanisms — Distributed Backtrace, an intent-preserving control plane, and Pause-Erased Time — without requiring kernel modifications. In the user study, DDB achieved 100% fault localization (vs. 38.5% for baseline tools) with 30 ms median backtrace latency and 1–5% throughput overhead, requiring only 20–60 lines of per-framework integration.

## Appendices (Referenced)

The paper includes several appendices covering:

- **A: PET Mechanism** — Formal description and invariants correctness, PET under-measurement slack, wall clock as clock-source, and vDSO handling.
- **B: Distributed Backtrace** — Distributed stack stitching algorithm details and stack stitching latency at scale.
- **C: Implementation Details** — Additional implementation-specific aspects.
- **D: Extended Measurement of Command Handling Latencies** — Full command latency measurements.
- **E: Migration Continuity (Nu and Quicksand)** — How DDB handles computation migration for caller heap state across process boundaries.
