---
url: https://trigora.dev/blog/durable-execution-without-history-replay/
title: "Durable execution without history replay"
author: Unattributed (Trigora)
date_fetched: 2026-09-13
topics:
  - distributed-systems
  - agent-architecture
---

A short position piece from Trigora proposing **Transparent Continuation Checkpointing (TCC)**: instead of recovering a failed durable program by replaying its retained execution history, the compiler and runtime capture the program's *live continuation* — the control state needed to resume from the current position — at every durable boundary, commit it, and restore it directly after a failure. The contrast: history replay does *load history → re-execute the prefix → reconstruct the position*; TCC does *load committed continuation → restore live state → resume*.

The claimed change is not constant-time recovery but a change in what recovery *depends on*: with replay, recovery cost tracks the retained history; with TCC, it tracks the state the program still needs. A program that has run ten thousand operations but holds a small live continuation should not get harder to recover just because its past is long. The author is careful about this — recovery stays sensitive to the size and structure of the continuation.

The preliminary evaluation holds live state at ~4 KB while durable-boundary depth grows from 10 to 1,000: TCC recovery stays around 0.6–0.9 ms while the evaluated Temporal baseline grows from ~61 ms to ~1.7 s. Worker creation is excluded, and the author explicitly disclaims any general production-speedup claim — it demonstrates a difference in recovery scaling under the tested conditions, nothing more. Execution semantics were exercised across 50,000 generated cases with no observed semantic failures in the evaluated subset.

External effects remain explicit durable operations (completed work is not re-run after recovery), unsupported language constructs fail at compile time rather than producing ambiguous runtime behavior, and the prototype supports durable effects, external waits and events, child executions, cancellation, structured concurrency, and crash recovery. Turning it into production infrastructure still requires: portable continuation representation, program/checkpoint versioning, handling larger live states, durable storage and commit protocols, observability, cross-language compatibility, framework integrations, and long-running correctness testing — plus open design questions on checkpoint retention, branching from earlier continuations, and runtime-version migration. Trigora is being built around this model, initially for long-running AI agents.
