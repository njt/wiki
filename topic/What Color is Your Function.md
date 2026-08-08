# What Color is Your Function

Bob Nystrom's 2015 allegory of "red" and "blue" functions is the most durable explanation of why async/await feels like a hack even when it works. The article names the structural problem — function coloring — that every language with async IO inherits, traces it to the callstack-unwinding requirement, and identifies threads (green or OS-level) as the only genuine solution. A decade later, JavaScript still has red and blue functions, and every new language that ships with async/await is opting into the coloring problem.

---

## Key Quotes

> "Every function has a color. Each function—anonymous callback or regular named one—is either red or blue."

The central metaphor. Red functions are async, blue are sync. The genius of the allegory is that it makes the problem feel absurd before revealing that this *is* how JavaScript, Dart, C#, and Python actually work. The absurdity is the point.

> "You can only call a red function from within another red function."

The infectiousness rule. This is why async propagates upward through a codebase like a virus. One low-level IO call turns your function red, which turns every caller red, which turns their callers red — all the way to `main()`. The compiler or runtime can't help you; the coloring is structural.

> "Red functions are more painful to call."

The ergonomic tax. In callback-era JavaScript, this meant nesting. In async/await languages, it means wrapping return types in `Task<T>` or `Future<T>`, needing `await` at every call site, and losing the ability to use red functions in synchronous contexts. The tax has been reduced but never eliminated.

> "Some core library functions are red."

The trap. You can try to keep everything blue, but if the platform's IO primitives are red — as they are in Node.js, the browser, and most async-first runtimes — you're forced into the red world. This is why Node.js developers couldn't just "avoid callbacks": the filesystem, the network, and the database all spoke red.

> "No one ever for a second thought that a programmer would write actual code like that."

On continuation-passing style (CPS). The transformation that turns nested callbacks into something the event loop can resume was invented as a *compiler intermediate representation* in the 1970s. Node.js made programmers hand-write CPS, which is like asking drivers to hand-crank their engines because the starter motor broke. Async/await automates the cranking, but the engine still works the same way.

> "Go is the language that does this most beautifully in my opinion. As soon as you do any IO operation, it just parks that goroutine and resumes any other ones that aren't blocked on IO."

The solution. Goroutines (and Lua coroutines, Ruby fibers, BEAM processes) let you suspend the *entire callstack* without unwinding it. The stack stays intact; the scheduler just switches to another goroutine. This eliminates the distinction between sync and async code — IO operations look synchronous but don't block other work. Nystrom's praise for Go is not about the language overall but about this one design decision.

> "Concurrency in Go is a facet of how *you* choose to model your program, and not a color seared into each function in the standard library."

The key distinction. In Go, concurrency is an architectural choice. In JavaScript, it's a property of every function signature. One is a tool you pick up when needed; the other is a tax you pay on every line of code, whether you're doing concurrent work or not.

---

## Key Themes

**#concept Function Coloring** — The core contribution. Nystrom didn't invent the async/sync problem, but he gave it the name that stuck. "Function coloring" entered the programming lexicon because the metaphor is precise: it's not that async functions are *different* from sync ones; it's that the language forces every function to declare a color, and the colors don't compose freely. The metaphor works because it makes the problem feel like the arbitrary restriction it is.

**#pattern Async Propagation (the infectiousness rule)** — Async infects upward. If `f()` calls a red function, `f()` must be red. If `g()` calls `f()`, `g()` must be red. The entire call chain to `main()` or the event handler turns red. This is not an implementation detail; it's a logical consequence of needing to unwind the callstack to yield to the event loop. Async/await makes the syntax cleaner but doesn't change the propagation rule.

**#concept CPS as Compiler IR, Not Programmer API** — Continuation-passing style transforms direct-style code into a chain of closures, moving local variables from the stack to the heap so they survive stack unwinding. Compilers have done this internally since the 1970s. Callbacks forced programmers to do it manually. Async/await automates it at the compiler level. The insight: every async abstraction is just a different skin on the same CPS transformation.

**#tool Green Threads as the Escape Hatch** — Languages with lightweight userspace threads (goroutines, coroutines, fibers, BEAM processes) avoid function coloring entirely because they can suspend the callstack in place rather than unwinding it. This is not a library-level fix — it requires runtime support. Nystrom's implicit argument is that any language shipping async/await without green threads chose a local maximum over a global one.

**#comparison Languages That Got It Right vs. Wrong** — The scorecard: JavaScript (colored), Dart (colored), C# (opted into coloring), Python (colored) vs. Go (colorless), Lua (colorless), Ruby (colorless), pre-async Java (colorless). The dividing line is threads. C# is the interesting case: it *could* avoid coloring by using OS threads, but chose to add async/await and the whole `Task<T>` machinery anyway, making it an opt-in failure.

---

## Critical Analysis

**The metaphor is the contribution, not the diagnosis.** The async/sync split was well-known before 2015. What Nystrom did was give it a name and an explanation that lets programmers *feel* why it's wrong, not just know it. The allegorical language construction — building up a strawman language that sounds absurd, then revealing it's exactly JavaScript — is a rhetorical masterclass. The article has outlasted most programming blog posts because the metaphor is more durable than any specific technology it critiques.

**Async/await fixed rule #4 but left rules #1–3 and #5 intact.** Nystrom's update acknowledges that async/await makes red functions less painful to call. But the other problems remain: functions still have colors, the infectiousness rule still applies, and core library functions are still red. Every async/await codebase eventually hits the "what color is my higher-order function?" problem — you can't write a generic `map()` or `filter()` that works on both sync and async callbacks without either duplicating the function or adding an effect-system layer the language doesn't support. [[Process-Based Concurrency BEAM OTP]] describes the actor model that avoids this entirely: BEAM processes are the logical endpoint of the green-thread approach Nystrom advocates, with per-process heaps, preemptive scheduling, and failure isolation that Go goroutines don't provide.

**The Go-vs-Rust tension in 2026.** Nystrom's article predates Rust's async story. Rust chose the coloring path — `async fn` returns `impl Future`, `.await` is required at call sites, and the ecosystem has struggled with the same higher-order-function problems Nystrom describes. The irony: Rust has the systems-programming chops to implement green threads (and did, in early versions), but removed them in favor of zero-cost async abstractions. Go, by contrast, baked goroutines into the runtime and the culture. The [[The GUS Stack — Go, Unix, SQLite]] argument for Go as the agent-friendly language is strengthened by Nystrom's analysis: goroutines mean agents writing Go code don't have to think about function color, which is one less thing for them to get wrong.

**The continuation connection the article doesn't fully explore.** Nystrom mentions that callbacks are CPS and that async/await automates the CPS transform, but doesn't connect this to the broader PL theory of delimited continuations. Languages with first-class continuations (Scheme, and increasingly effect-system languages like Koka and OCaml 5) can implement async IO as a library rather than a language feature — `await` becomes a user-defined control operator that captures and resumes the continuation. This is the real escape from function coloring: not green threads, but giving programmers the ability to define their own control flow abstractions. The fact that this requires deep language support is Nystrom's point, stated more generally.

**What the article gets wrong: Java.** Nystrom says Java "doesn't have this problem" because it uses threads. At the time (2015), this was mostly true — Java's core IO was synchronous and you used thread pools for concurrency. But Java *did* have a coloring problem, just in a different form: checked exceptions. A method that throws `IOException` forces every caller to either catch it or declare it — the same infectious propagation as async. And Project Loom (virtual threads in Java 21+) retroactively validates Nystrom's thesis: by making threads cheap enough to use per-task, Java eliminated the pressure to adopt async/await, keeping the language colorless.

**The article's blind spot: performance.** Nystrom's argument is about ergonomics and composability, not performance. He doesn't engage with *why* languages choose async IO over threads: OS threads are expensive (memory, context-switch overhead, kernel involvement), and event loops with non-blocking IO scale better for IO-bound workloads. Go's goroutines work because they're userspace threads — cheap enough to have millions of them. The implicit requirement is: if you want colorless functions, you need a runtime with lightweight threading. Languages without that runtime (JavaScript in the browser, early Rust, C) face a harder tradeoff.

**2026 relevance: agents amplify the coloring problem.** Every [[Agent Memory and Context|agent]] that writes code inherits its language's coloring model. An agent writing JavaScript will produce async functions and then struggle to compose them correctly — the same way humans do, but faster and with less judgment. An agent writing Go doesn't have this problem. In this sense, Nystrom's article is a harness-engineering argument disguised as a language rant: pick a colorless language and your agents produce more composable code with fewer categories of bug. The function-coloring property of a language becomes a multiplier on agent reliability.

---

*Sources: [[raw/what-color-is-your-function]], [[summary/what-color-is-your-function]]*
*Last updated: 2026-08-08*
