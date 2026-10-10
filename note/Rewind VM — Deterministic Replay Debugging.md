# Rewind VM — Deterministic Replay Debugging

Rewind VM is a deterministic Linux VM (by Farid Zakaria) in which any command — a Nix build, a test suite, a plain `make check` — runs so reproducibly that a failure can be scrubbed back step-by-step like a video, inspected, and forked from the exact step where a run went wrong. Nothing is recorded; determinism *is* the recording. Engine and CLI are MIT open source; the timeline desktop app is $49/$99.

---

The core inversion: traditional record-and-replay debuggers (rr, chronon) capture execution. Rewind instead makes execution a pure function of its inputs — a read-only erofs root filesystem mapped into memory, one vCPU, and a patched kernel where interrupts, time, the timestamp counter, and the hardware RNG advance only when the VM hands control back to the Rewind platform. Each handoff is a "step." Snapshots ("keyframes") every quarter-second via KVM's dirty-page log go into a content-addressed BLAKE3+zstd store, so a page shared across keyframes, runs, and forks is stored once. To reach step N, restore the last keyframe at or before N and replay forward.

The killer feature is `rewind check`: it reruns the workload under perturbed schedules until one run ends differently, then *names the step whose reschedule decides the outcome*. That turns the classic flaky-test ritual ("add a printf, it passes") into a search with a witness. For Nix users the pitch is tight: `rewind nix .#mylib` uses the derivation hash as the unit of reproduction, pins the VM kernel itself as a derivation, hashes each output against the host store, and starts the VM's clock at midnight UTC of the run day so a replay a year later sees the same date.

The case studies give it credibility beyond a demo: an unreported SIGPIPE flake in Nix's own `gc-closure.sh`, a known `SQLITE_BUSY_SNAPSHOT` schema-migration hang pinned with gdb inside the VM, and a devenv lost-output bug traced to `tokio::select!` racing a buffered line. Honest limits are listed: one vCPU (parallelism = many machines), no network but loopback, uninterruptible non-syscall spin loops, and replays only within a CPU vendor.

> "Nothing is recorded while the workload runs. A run is a function of its inputs, so any step of it can be computed again."

This is the design decision that makes the whole product coherent — and the reason it scales to CI: a failing run is just a directory of inputs plus events-with-steps, reusable forever.

> "`rewind check` builds under perturbed schedules, stops at the first build that ends differently, and names the step that decides it: without that one perturbation, the build passes."

Naming the deciding step converts a heisenbug from folklore into a diagnosis. Most flaky-test tooling stops at "it's flaky"; this points at the interleaving.

Key themes: #tool #pattern (determinism-as-recording) #concept (schedule perturbation as a search problem)

## Critical take

This is the strongest piece of debugging infrastructure I've seen aimed specifically at *nondeterminism* rather than at crashes. The opinionated bet — determinism plus fork-from-step beats record-everything — is the same bet Nix makes about builds, extended into time. The Nix integration is where it shines: Nix already solved closed-world inputs but famously not closed-world *schedules*, and `rewind check` slots exactly into `nix build --check`'s "may not be deterministic" gap.

The weaknesses are real and admitted: single vCPU means you debug the serialized interleaving, not the true concurrent one; a busy loop without syscalls is invisible to the step model; and the determinism guarantee doesn't cross CPU vendors. The $49 app pricing is a sensible indie model over an MIT core, and the "real transcripts only" discipline in the tutorials and case studies — including finding a bug Nix's tracker never saw — is exactly the evidence a tool like this needs to be believed.

## Relations

- Strengthens [[A Field Guide to Bugs]] with a new tooling class for the hardest bug family it describes — the heisenbug — by making the failure replay identically and naming the deciding interleaving.
- Nuances [[Software Engineering Craft]]'s debugging tradition: record-and-replay determinism (rr-style) reframed as *compute-on-demand* — no recording overhead, replay from pure inputs.
- Complicates [[The New Software Lifecycle]]'s claim that verification is the bottleneck: Rewind attacks the flaky/irreproducible-failure half of verification that most agent-loop harnesses quietly ignore, and `rewind check` is a plausible verification tool inside an agent loop.
- Complements [[The GUS Stack — Go, Unix, SQLite]]: both argue for boring, pinned, training-data-dense foundations; Rewind shows what a fully pinned world (Nix closure + pinned kernel + frozen clock) buys when things go wrong.

---
*Sources: [[raw/rewindvm-dev]], [[summary/rewindvm-dev]]*
*Last updated: 2026-10-10*
