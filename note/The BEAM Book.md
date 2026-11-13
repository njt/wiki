# The BEAM Book

The BEAM Book is Erik Stenman's open-source, CC BY 4.0 manuscript (repo `happi/theBeamBook`) explaining the Erlang RunTime System from compiler to scheduler to distribution. It matters because it is one of the few serious, code-backed treatments of a production virtual machine that was *designed for fault-tolerant distributed systems from day one* — the same supervision, isolation and message-passing ideas the agent world keeps reinventing, with forty years of production mileage.

---

The repo is a ~15,000-line AsciiDoc book (`book.asciidoc` includes ~30 chapters under `chapters/`) plus per-chapter runnable code under `code/`, built with Asciidoctor into PDF/EPUB/HTML (`make pdf-a4 html`). A second edition lives in `second-edition/` with a companion "Implementation Deep Dives" PDF including a loader source walkthrough. Contributors include Richard Carlsson, Roberto Aloi and Dmytro Lytovchenko; first edition printed 2025 (ISBN 978-91-531-4253-9).

## Architecture

Not a software system but a *documented* one — the book dissects ERTS (Ericsson's Erlang/OTP runtime, written in C) in two parts:

- **Part I, Understanding ERTS**: `chapters/compiler.asciidoc` (compiler options, intermediate formats, parse transforms, and how to compile your own language to BEAM), `chapters/processes.asciidoc` (the lightweight process as the central abstraction), `chapters/beam.asciidoc` (the VM), `chapters/beam_modules.asciidoc` (the `.beam` chunk format), `chapters/beam_instructions.asciidoc` plus a 587-line instruction appendix, `chapters/scheduling.asciidoc`, `chapters/memory.asciidoc` (1,557 lines — the longest chapter), `chapters/gc.asciidoc`, `chapters/io.asciidoc`, `chapters/distribution.asciidoc`, `chapters/c.asciidoc` (NIFs and port drivers).
- **Part II, Running ERTS**: ops, debugging, tracing, testing, profiling — the observability story of a live node.

The chapter code is itself instructive: `code/beam_modules_chapter/src/beamfile.erl` is a ~30-line Erlang binary-pattern-matching parser for the `.beam` chunk format (`FOR1`/`BEAM` header, 4-byte-aligned `Atom`/`ExpT`/`ImpT` chunks), and `code/beam_chapter/src/vsm.c` is a minimal virtual stack machine used pedagogically before showing why BEAM is *not* one.

## Key techniques

- **Register machine over WAM, not a stack machine.** Unlike BEAM's predecessor JAM (Joe's Abstract Machine), BEAM is a register machine "loosely based on" the Warren Abstract Machine. The book builds the naive stack-machine compiler/interpreter in a few lines of Erlang, then shows why real implementations pay for register allocation: compact code generation trades against execution speed.
- **Reductions as scheduler currency.** The scheduler is preemptive via *reduction counting* — a term from lambda-calculus beta-reduction, but deliberately loosely defined: roughly one per function call, checked against a quota at each call boundary. BIFs bump the counter proportionally to their work. This gives soft preemption without instrumenting every instruction.
- **Process = OS-process analogy taken literally.** Each Erlang process has its own address space (stack + heap in one memory block), communicates only via signals/messages, and is scheduled preemptively — so faults and GC are isolated per process, not per runtime.
- **Distribution as a runtime primitive.** `chapters/distribution.asciidoc` treats named nodes (`name@host`, short vs long names, which cannot interoperate) connecting into a cluster as core runtime behaviour, not a library.

## Design decisions

The introduction's thesis is the book's stance: writing a bytecode interpreter or even a JIT is "just the tip of the iceberg" — the runtime system (memory management, scheduling, I/O, distribution, signals) is the actual product. That explains the page budget: memory and GC together outweigh the instruction-set chapters. The trade-off is honesty over polish: the book repeatedly admits ERTS-specific pragmatism (no official definition of an Erlang Runtime System exists; timeouts are only guaranteed to *not* fire early, not to be soft real-time). The second edition's split of implementation deep-dives into a separate paid companion PDF while keeping examples CC BY 4.0 is a sustainability trade-off.

## Comparison notes

- The pedagogy — build a toy VM, then explain why the real one differs — parallels [[Ship Your Interpreter (Basis)]], which pushes the same move further: formal refinement proofs that the shipped interpreter *is* the semantics. The BEAM Book predates that world and reaches understanding by source walkthrough instead of proof.
- Dan Luu's [[We Should Be More Ambitious (Dan Luu)]] celebrates agents building interpreters in hours; this book is the accumulated opposite asset — decades of distilled runtime engineering that such agents would need as context to build anything comparable on purpose.
- The BEAM's process isolation and message-passing topology is exactly the substrate [[Multi-Agent Systems Have a Distributed Systems Problem]] says multi-agent LLM systems lack: Meiklejohn's lost updates and missing causal ordering are problems Erlang solved at the runtime layer by making every actor a scheduler-managed process with per-process heap. Agent frameworks reinvent supervision where ERTS ships it.
- The node-as-unit-of-deployment model with built-in clustering contrasts with [[Dippin (Language)]], which keeps language and runtime separate — BEAM's lesson is that distribution guarantees are cheapest when the runtime owns them.
- [[Process-Based Concurrency BEAM OTP]] covers the same ground from the programmer's side (processes, OTP supervision as the application pattern); the BEAM Book is its counterpart below the API — explaining *why* those primitives are cheap, in scheduler, heap and allocator terms.

#tool #project #runtime #erlang #distributed-systems #concurrency

---
*Sources: [[raw/thebeambook]], [[summary/thebeambook]]*
*Last updated: 2026-11-13*
