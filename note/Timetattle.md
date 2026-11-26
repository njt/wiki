# Timetattle

Timetattle (Ben Syverson, MIT, macOS-only Go) reconstructs when work happened and for which client from forensic traces — Claude Code transcripts, git history, Motion autosaves, file mtimes, knowledgeC backlight data and Biome's ScreenTime.AppUsage stream — born from a fixed-price contract that silently overran by 25%. Its real contribution is epistemic architecture: an append-only event store where inference is a disposable derivation and every figure is labelled observed, inferred, or unrecorded. It is one of the few tools in the wiki that treats *agent activity itself as billable evidence* — a coding agent working with the lid shut counts as capacity spent.

---

## Architecture

~60k lines of Go, one `cmd/timetattle` binary, SQLite event store at `~/.timetattle/timetattle.db`, launched hourly by a Homebrew-scheduled launchd agent.

- **Capture vs. derive split.** Only `harvest` is urgent — knowledgeC (~28-day window) and Biome (fixed 512 KB segments, eventually discarded) delete themselves. Everything else (git, transcripts, files) is durable, so every other subcommand is "a reader or a re-runnable derivation". This asymmetry drives the whole command surface: `status` asks whether collection still *works*, `metrics` what the *numbers* are, `orient` what to *do*.
- **Immutable event store** (`internal/store/store.go`): events are never edited. When a harvest sees an app still frontmost, it stores an open-ended event; the next harvest stores a second, complete event; collapsing them happens in a `canonical_event` *view* on the way out. Deduplication is by content, not watermark, because sources re-read full windows and legitimately re-emit evidence. Client attribution lives in an append-only `client_project` log (it's a fact about a moment, not a row), folded on read.
- **Three source roles as configuration, not code** (`internal/sessions`): EVIDENCE (clusters into sittings), PRESENCE (never creates a sitting; may only suppress timer-written observations), IGNORED (read then dropped — a stream whose *meaning* is unestablished, like `/device/isLocked`, where one 15h32m "locked" interval would manufacture a fifteen-hour sitting out of a night's sleep).
- **Declarative sources** (`internal/manifest`): file-based evidence (Motion autosave vault, Blender `.blend1` backups) is described in JSON manifests, no Go required; versioned with a mandatory `manifest_version` because manifests live outside the repo, and `**` globs are refused outright.
- **The two test runs**: `go test ./...` is machine-independent (untagged corpus tests behind `//go:build realdata` are not even compiled), and `internal/corpusgate` *fails the ordinary suite* if an untagged test starts touching the real machine — a gate that keeps the two suites from merging back.

## Key techniques

- **Clustering across sources, never interpolating within one.** Commits, saves and Claude messages are instants; presence is the only interval source. The idle gap threshold is a parameter, not a constant: measured presence ranges 530h→1200h across candidate values (2.3× spread), and the gap distribution is sharply bimodal (93.5% of gaps under ten seconds) — which does *not* make the threshold safe, because one 45-minute gap costs 45 minutes whether or not it has company.
- **Three presence states, one-directional suppression.** Observed-absent, observed-present, nothing-recorded are distinct; reading silence as darkness deleted 1h27m of confirmed work in one measured bug. The rule survived a dramatic revision: darkness used to split sittings, until the "dark lid" case (render, compile, unattended agent) showed that `knowledgec-backlit` observes a *lid*, a proxy for a proxy. The fix was the `Cause` field: only wall-clock-timer artifacts may be suppressed by darkness, which recovered 35.8 hours of engaged time the old rule deleted.
- **`internal/fit` refuses confident answers it doesn't have.** Parameter search over Gap/LeadIn/TailOut excludes inert points (gap is provably inert below LeadIn+TailOut — verified as a falsifiable prediction on the live corpus), reports `Result.Flat` bands rather than minimising over them, and warns that a reviewer confirming what he was shown produces labels a fit cannot learn from.
- **Claude Code transcript forensics** (`internal/transcripts`): distinguishes human turns from machine turns — including tool results relayed as "user"-role records — folds subagent sessions into parents, attributes by the record's `cwd` not a lossy directory name, dedupes replayed records (compaction and side-question subagents re-emit earlier turns with the same uuid; 6,822 of 6,922 repeated uuids trace to this), and keeps explicit zero token figures apart from absent ones with pointers. A symlinked `~/.claude` root once silently zeroed the project's largest source (722,463 events → 0) because `filepath.WalkDir` Lstats its root; the fix logs "the root was never walked" so *observed nothing* and *nothing was observed* never look alike.
- **Biome pairing discipline** (`internal/biome`): unpaired stop records (~1 in 60) are wake artifacts, not corruption; back-dating them from the previous record would charge five minutes of a sleeping machine to Safari instead of two seconds of use.
- **A publication gate** (`scripts/gatecheck`, `make classify`) that derives its list of real client names from the corpus and *refuses to answer* rather than answering wrongly — a confidentiality check for a repo whose design docs name real clients.

## Design decisions

- **Insight, not system of record.** Every report prints its own inference ratio (67.3% of presence, 85.3% of engaged hours are clustering margin) beside the number instead of behind it. Nothing is meant to drive an invoice alone. This is structural: observations are immutable, inference is disposable, corrections are overlays — so "that was a meeting, not work" never invents retraction semantics.
- **Engaged vs. presence can't collapse into one metric.** Engaged is per-project and may overlap (concurrent work bills to both); presence is wall clock and never double-counts, and is the only honest denominator for capacity — a retainer measured against engaged time could read satisfied at 30% of real attention.
- **Signing before scheduling.** macOS keys Full Disk Access to a binary's code signature; Go links ad-hoc, so every bare `go install` silently revokes the grant while System Settings still shows it enabled — which cost 44 hours of capture on 2026-07-30. `install-agent.sh` signs first, deliberately.
- **Ergonomics as epistemology.** `orient` answers cold in ~2s by reading the stored derivation rather than re-clustering, and always says how much evidence arrived after it. Bare `timetattle` prints a project headline *plus* the usage, so the command never stops explaining itself.

## Comparison notes

- Where [[What a Useful AI Trace Should Actually Contain]] argues agent traces need six deliberate elements to be debuggable, Timetattle is the consumer side of the same insight: it reads Claude Code transcripts as historical evidence and shows what metadata actually survives (explicit zeros vs. absent fields, replayed uuids, human-vs-machine turns).
- [[Pi Observers]] instruments the agent *live*, with reconciled advisories; Timetattle does the opposite — capture now, infer later, re-run the inference forever. Both share the underlying conviction that agent activity leaves structured, machine-readable evidence worth mining.
- Unlike time trackers (screen-on time, timers you start), Timetattle insists presence is not screen-on and that an unattended agent session is work — a claim with direct billing consequences that no conventional tracker can express.
- Its immutable-store-plus-derived-views design is event sourcing in the small, applied to forensics: like the wiki's event-sourcing notes, it discovers that retraction is the hard problem, and routes around it with an overlay instead of edits.

#tool #project #agents #macos #forensics

---
*Sources: [[raw/timetattle]], [[summary/timetattle]]*
*Last updated: 2026-11-26*
