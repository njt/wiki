# Process-Based Concurrency: Why BEAM and OTP Keep Being Right

Erlang's BEAM runtime has embedded the actor model at the VM level since 1986, and modern AI frameworks keep independently reinventing the same patterns. The article argues this isn't coincidence -- BEAM's approach to concurrency is fundamentally correct, and other platforms can only approximate it in userspace.

---

## Key Quotes

> "On BEAM, the blast radius of a failure is exactly one process. Always."

> "'Let it crash' does not mean 'ignore errors.' It means: separate the code that does work from the code that handles failure."

> "An AI agent is long-lived, stateful, failure-prone, and concurrent. This maps directly to BEAM processes."

## Key Themes

#erlang #agent-architecture #orchestration

The core argument is simple: there are two approaches to concurrency (shared state with locks vs. isolated state with message passing), and BEAM chose correctly in 1986. The properties that make BEAM processes different from threads or goroutines:

- **~2KB per process** -- millions per machine
- **Own heap, stack, and GC** -- per-process garbage collection means no system-wide pauses
- **Preemptive scheduling** -- ~4,000 reductions per timeslice, no process can starve the system
- **Complete isolation** -- memory access between processes is impossible, not merely discouraged
- **Hot code swapping** -- deploy without disconnecting active sessions

The "let it crash" philosophy separates business logic from error recovery. Supervisors manage failure at a higher level with strategies (`:one_for_one`, `:one_for_all`, `:rest_for_one`), keeping business logic clean.

The AI agent connection is the strongest part: agents are long-lived, stateful, failure-prone, and concurrent -- exactly the problem BEAM was built to solve. AutoGen, LangGraph, and CrewAI are all independently building OTP-like patterns in Python. The pattern of reinvention (Java -> Akka in 2009, Node.js -> worker threads in 2010s, AI frameworks -> OTP patterns in 2020s) is compelling evidence.

Connects to [[NornicDB]] (graph database for agent memory -- agents need both the concurrency model and the persistence layer), [[Navaris]] and [[OpenSandbox]] (process isolation at the sandbox level rather than the VM level).

## Critical Analysis

This is the best single-article introduction to why BEAM matters for the AI agent era. The argument from reinvention is strong: when multiple independent teams arrive at the same architecture, it signals convergence on a correct solution. The honest tradeoff discussion (raw throughput, ecosystem size, learning curve) prevents this from being a Erlang fanboy piece. The weakest point is the practical one: even if BEAM is the right architecture, Python dominates the AI ecosystem, and "rewrite it in Elixir" is not actionable advice for most teams. The real value of this article is as a design reference -- build your Python/Go/Rust agent framework with BEAM's principles even if you can't use BEAM's runtime.

---
*Sources: [[summary/process-based-concurrency-beam-otp]]*
*Last updated: 2026-05-14*
