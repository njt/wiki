# MAccConc — Exploring Linux Kernel Race Condition Interleavings

A Google Project Zero researcher built **MAccConc**, a toolchain that turns race condition bugs in the Linux kernel from heisenbugs into reproducible, explorable state machines: KCOV records every memory access via outline ASAN instrumentation, cross-thread accesses are identified as SKI-style "communication points," and a new `KCOV_SET_DI` ioctl lets userspace *force* specific thread orderings at points named by count-augmented stack traces. Three frontends ride on it — an automatic A-B-A interleaving tester, a terminal UI, and a GUI with interactive call graphs — aimed at confirming bug candidates, writing regression tests for race fixes, and eventually fuzzing.

---

## Key Quotes

> "Many security bugs are race conditions, where multi-threaded execution has to occur with the right interleaving for a negative effect to appear."

The whole project in one sentence. The security-research framing matters: this isn't an academic concurrency work, it's a vulnerability-discovery workflow problem — you read code, suspect a race, and then have no efficient way to prove the bug is real, which is exactly where exploitation research stalls.

> "For race condition bugs, it can be hard to achieve either outcome. For Linux kernel bugs, I often resort to recompiling the kernel after adding conditional `mdelay()` calls … Regardless of platform, this approach can be time consuming and can require trial and error."

The before-picture: senior kernel security work reduced to sprinkling spin-delay calls into a custom kernel build and hoping to hit the interleaving. DTrace's `chill()` offers a no-rebuild variant but can only stop at function boundaries. That the field's best practitioners worked this way in 2026 says how primitive concurrency tooling has been.

> "I am instead identifying memory accesses with count-augmented stack traces, where each stack trace element essentially consists of a callee function address and a number indicating how many calls to this callee should be skipped in the calling stack frame."

The core theoretical contribution. SKI identified accesses by snapshotting the VM so every run starts from identical state; MAccConc instead gives each access a *name* — "the second call to `_raw_spin_unlock` inside `unix_stream_read_generic`, at instruction X" — that survives across runs and is independent of heap addresses. It's how a human already reads a stack trace, formalized into a stable identifier. The cost was upstream: it needed function entry/exit events in SanitizerCoverage, which the author landed in LLVM 23.1.0.

> "It discovers one ordering where `dup(5)` returns `5`, which is working as intended but might be a somewhat surprising result."

The demo is well chosen: concurrent `dup(5)` and `close(5)`, where the tool finds the interleaving in which the file descriptor gets reused and `dup()` hands back the very fd you just closed. No crash, no sanitizer report — just a semantic surprise only a systematic interleaving search would surface. It shows the tool explores *behavior*, not only memory-safety violations.

> "My kernel patches are in a clean state; the userspace tooling is a bit more hacky, in particular the GUI implementation. The command-line tooling can only handle two concurrent threads."

The honest status report. The kernel side is being upstreamed properly (patches posted for review with the blog post); the GUI is research-grade; the automatic tester is two-thread-only. This is infrastructure-first sequencing — get the mechanism into the kernel, let the pretty tools catch up.

## Key Themes

**#tool MAccConc** — three frontends over one kernel mechanism: `kcov-autorace` (automatic A-B-A interleaving tester), a terminal UI, and a GUI (`kcov-vsock-client` harness) where you click two memory accesses to create an ordering constraint, re-run, and see the forced interleaving annotated in the trace.

**#concept Communication points** — inherited from the SKI paper: pairs of memory accesses on two threads over overlapping ranges where at least one is a write. Searching the trace for these pairs is how "interesting interleavings" get defined before any ordering is forced.

**#pattern Delay injection** — `KCOV_SET_DI` arms wake/wait actions (`DI_STACK_WAKE_PRE`, `DI_STACK_WAIT`, `DI_STACK_WAKE_POST`) on a shared flag array at named stack-trace points. Two usage styles: constraint-style (A-happens-before-B, used by the GUI/TUI) and fully-specified context-switch-style ordering (used by the A-B-A tester), which the author argues is more deterministic and easier to reason about.

**#concept In-kernel instrumentation over VM trickery** — unlike SKI's patched QEMU/TCG with snapshots, collection happens inside the kernel through KCOV fed by outline-mode ASAN helpers (with `asan-opt-same-temp` disabling call-merging). The stated bet: in-kernel collection can later expose higher-level events (lock acquire/release) and could work on bare metal. KCOV was chosen over ftrace for its simpler buffer, always-on static instrumentation, and higher event-rate orientation.

**#pattern Upstreaming as part of the research** — the dependencies were pushed into LLVM (SanitizerCoverage function entry/exit, landed 23.1.0) and proposed into DWARF (`DW_AT_alloc_type`, accepted into the DWARF 6 draft) so memory-access traces can carry allocation-site type information. The kernel patch series goes to upstream review with the post. The tool is a forcing function for infrastructure everyone else inherits.

## Critical Analysis

The identification problem is the insight that makes this more than a syzkaller-adjacent demo. Everything hard about concurrency testing across runs — comparing traces, re-targeting a schedule, writing a regression test that means something — requires a *stable name* for "the point where thread A touches X." Addresses churn (fresh allocations), instruction addresses are ambiguous (`memcpy` is called from everywhere), and snapshots (SKI's answer) are heavyweight and VM-bound. Count-augmented stack traces are the elegant third way, and the author paid the real cost of elegance: a compiler feature that didn't exist. That's the difference between a blog demo and load-bearing infrastructure — MAccConc arrived with its prerequisites upstreamed.

The engineering choices are consistently argued rather than defaulted, and the article shows the reasoning: KCOV over ftrace (buffer simplicity, static instrumentation, event-rate design), ASAN-over-TSAN with the tradeoff stated (TSAN would give atomicity info but you can't have both sanitizers, and TSAN hooks would break UAF detection), kernel-side collection over QEMU patching (higher-level events later, bare metal in principle). Just as valuable is the candor about what *doesn't* work: ASAN doesn't instrument direct stack accesses, so races over on-stack objects like wait queues are invisible; background work (RCU callbacks, loopback RX) isn't wired to remote coverage yet, though there's a draft patch; forcing an *impossible* ordering just burns CPU in a semi-deadlock until the timeout. The future-work list is really a published defect list.

The scope limits are the ones to watch. Two threads in the CLI is enough for proving bug candidates — the manual-reading-then-proving workflow the author describes — but real kernel races are often three-plus contexts including softirq/RCU, and the GUI's `kcov-vsock-client` path hints at how much harness work more contexts require. The constraint-style vs. fully-specified-ordering split also reads like a design still mid-pivot: the author explicitly prefers the fully-specified style and plans to move the manual tools onto it. And the fuzzing story — the part that would connect this to automated bug discovery at scale — is entirely future work; Snowboard-style test-case composition is sketched, not built.

For this wiki, MAccConc is the sharpest example yet of a pattern that keeps recurring: the hardest tooling problems are *naming and forcing* problems. Deterministic scheduling needed stable names for executions (here: count-augmented stack traces; in [[DDB — Source-Level Interactive Debugging for Distributed Applications]], stable intents across autoscaling); testability needed the ability to inject failure on demand. The tools differ wildly; the shape is the same.

## Cross-Links

- [[A Field Guide to Bugs]] — the Heisenbug ("attach a debugger and the bug evaporates") is MAccConc's entire reason to exist, and this source strengthens that page's taxonomy claim from the other direction: the bug class is real and stable enough that a systematic counter-weapon can be built against it, replacing lucky timing with forced interleavings.
- [[How SQLite Tests Software]] — Hipp's design-for-testability doctrine ("every failure mode can be injected on demand") is here applied to an untestable-by-construction legacy codebase; it nuances that page by showing the pattern when you *can't* re-architect for testability from day one — you retrofit injection seams as external instrumentation and kernel ioctls instead.
- [[DDB — Source-Level Interactive Debugging for Distributed Applications]] — both projects extend interactive debugging into a domain the industry had normalized as impossible (cross-process stacks there, thread interleavings here), and MAccConc complicates DDB's "this should have existed ten years ago" by showing what quietly gates such tools: the LLVM and DWARF features the author had to land first.
- [[Process-Based Concurrency BEAM OTP]] — that page argues shared-memory threading is the wrong concurrency model; MAccConc is the definitive itemized bill for staying on it, an entire kernel-instrumentation toolchain existing solely because threads share memory. It sharpens the contrast: BEAM deletes this bug class by construction, while the kernel pays to explore it.

---
*Sources: [[raw/maccconc-race-condition-html]], [[summary/maccconc-race-condition-html]]*
*Last updated: 2026-09-13*
