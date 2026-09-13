---
url: https://trigora.dev/blog/durable-execution-without-history-replay/
date_fetched: 2026-09-13
---

# Durable execution without history replay

Most durable execution systems recover by replaying retained execution history. After a worker fails, a fresh worker loads the history and re-executes the program until it reconstructs the current position.

This is a useful model. It provides durable progress while allowing workers to remain ephemeral. But it also makes accumulated history part of the recovery path.

That tradeoff becomes more noticeable for programs that operate for hours or days, call many tools, wait for external events, create child executions, and change direction dynamically. Long-running agents increasingly have this shape.

I’ve built and evaluated a different recovery primitive: checkpointing the program continuation instead of reconstructing it from history.

## Transparent Continuation Checkpointing

I call the approach Transparent Continuation Checkpointing, or TCC.

At durable boundaries, the compiler and runtime capture the live continuation: the control state required for the program to continue from its current position. When execution resumes after a failure, the runtime loads the committed continuation and restores the program directly.

The distinction is:

**History replay**

Load retained history → re-execute the prefix → reconstruct the current position

**TCC**

Load committed continuation → restore live execution state → resume

External effects remain explicit durable operations. Completed durable work is not repeated after recovery, and unsupported language constructs fail during compilation rather than producing ambiguous runtime behaviour.

The current prototype supports durable effects, external waits and events, child executions, cancellation, structured concurrency, and crash recovery.

## What changes

TCC does not make recovery constant-time. Recovery remains sensitive to the size and structure of the live continuation.

The intended change is in what recovery depends on.

With replay, recovery is influenced by the execution history retained to reconstruct the current position. With TCC, recovery is influenced primarily by the state the program still needs.

A program that has performed ten thousand operations but retains a small live continuation should not necessarily become harder to recover simply because its past is long.

## Preliminary evaluation

I ran a controlled comparison in which live continuation state remained approximately fixed while durable-boundary depth increased from 10 to 1,000.

In that evaluation, TCC recovery remained between approximately 0.6 and 0.9 milliseconds. Fresh-worker replay reconstruction in the evaluated Temporal baseline increased from approximately 61 milliseconds to 1.7 seconds.

Worker creation was excluded, the live state was approximately 4 KB, and these results should not be interpreted as a general production-speedup claim. They demonstrate a difference in recovery scaling under the tested conditions, not that every TCC workload will outperform every replay-based system.

I have also exercised the execution semantics across 50,000 generated cases, with no observed semantic failures in the evaluated subset.

## What remains difficult

Turning the prototype into production infrastructure still involves substantial work:

- Portable continuation representation
- Program and checkpoint versioning
- Efficient handling of larger live states
- Durable storage and commit protocols
- Operational observability
- Compatibility across language frontends
- Framework integrations
- Long-running correctness and failure testing

There are also design questions around checkpoint retention, branching from previous continuations, migration between runtime versions, and how much of the execution representation should remain stable across languages.

I’m building Trigora around this model, initially for long-running AI agents. The broader question is whether continuation-based recovery can provide a better execution substrate for dynamic, long-lived software.

The architecture, semantics, benchmark setup, and current limitations are described in more detail in the technical paper. You can also see continuation recovery in action in a controlled demonstration of the current TCC compiler/runtime.

I’d be particularly interested in criticism from people who have worked on workflow engines, compilers, checkpointing systems, or distributed runtimes.
