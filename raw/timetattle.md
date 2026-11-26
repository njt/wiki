---
url: https://github.com/bensyverson/timetattle
date_fetched: 2026-11-26
---

# Timetattle

Reconstructs *when* work happened, and *for whom*, from the traces work
leaves behind — Claude Code transcripts, git history, task databases, file
timestamps and macOS presence data.

It exists because of a specific failure. A fixed-price contract was
budgeted at 40 hours and consumed roughly 50. The overrun was detectable in
real time from around the halfway point, and nothing was watching; it
surfaced three weeks later, only by combining four evidence sources with
human recollection.

## What this is, and what it is not

**Timetattle produces insight, not a system of record.** It is a signal for when
work happened, not an authority on what anyone owes, and nothing it reports is a
timesheet. On the current corpus **67.3% of presence** — the wall clock the
sittings cover — is inferred by the clustering rather than observed by anything,
and the **engaged** hours a project is billed for are more inferred still: 178.7h
observed out of 1216.0h. Every report prints those alongside the number instead
of behind it, and both come off one run of `timetattle metrics --all`: the first
from its `PRESENCE` line, the second from the `observed` and `absolute` columns
of its per-project table. The tool is most useful exactly where it disagrees
with what you remember.

Hours attributed to a client are **directional rather than final**. Higher
attribution fidelity makes them more useful and is worth pursuing, but no figure
here is meant to drive an invoice on its own — which is why the reports separate
what was observed from what was inferred, and name what nobody recorded at all,
rather than resolving the three into a single confident total.

That is structural rather than a disclaimer, and it shapes every layer.
Observations are immutable and are never edited. Inference is derived,
disposable and re-runnable, because the model will change many times. Human
corrections are overlays keyed to a wall-clock span, never rows in the evidence —
an append-only log of observations cannot express "that was a meeting, not work"
without inventing retraction semantics that would tax every read forever. What
was observed, what was inferred, and what nobody recorded stay separable all the
way to the report.

Timetattle answers two questions:

1. **How is capacity actually spent?** Utilization and the shape of the working
   day — how much, when, in what blocks, and how it divides across concurrent
   work. Capacity is measured against **presence**, which is not screen-on time:
   a render or a coding agent running with the laptop shut is counted, because
   it is work being produced and time being spent.
2. **Where did the time go?** Reconstruct what was worked on and when, well
   enough to know whether a contract is underwater.

## Status

Early build, pre-launch. The harvester works; everything else is in progress.

The design record — the v1 plan, the spike findings, and the dated documents
that measure and correct each other — is kept outside this repository. It is
written against one real corpus of one person's working days, so it names real
clients, real directories and real hours throughout, and none of that is
separable from the measurements without making them irreproducible. What it
concluded lives here instead, in [the documentation](docs/content/docs/) and in
the comments, which are written to stand on their own for that reason.

## Install

```console
$ brew install bensyverson/tap/timetattle
$ brew services start timetattle      # harvest every hour
```

Then grant it Full Disk Access (`brew info timetattle` says where). macOS only.

## Quick start

From a clone instead of Homebrew, `make agent` does the install and the
scheduling. Use one or the other: both schedule the same launchd agent.

```console
$ make agent                   # go install, sign, schedule the harvester
$ timetattle harvest
$ timetattle status            # is the evidence still arriving?
$ timetattle orient            # what should I do about it?
```

`timetattle orient` takes no arguments and is the one command to run with no
memory of a previous session: it says what the state is and names the single
action worth taking. Three commands answer three different questions and none
answers another's — **orient** says what to *do*, **status** says whether
collection is still *working*, **metrics** says what the *numbers* are.

`timetattle` with **no arguments at all** prints a headline for whatever project
the current directory belongs to, and then prints the usage below it. The usage
never disappears, so the bare command means the same thing in every directory
and still tells you what the commands are; the headline is the extra answer a
registered directory affords. Outside one, it says the directory is not
registered — which is a different fact from a project with no hours in it. It
reads the derivation [`sessionize`](docs/content/docs/sessions.md) already
stored rather than re-clustering the corpus, so it answers in about two seconds
where a full report takes fifteen, and it says on every run how much evidence
arrived after that derivation and is therefore in no sitting yet.

`make agent` (which runs `install-agent.sh`) signs before scheduling, and
that order is deliberate — and it is why routine rebuilds should go through
`make install`, which re-signs, rather than a bare `go install`, which does not.
macOS keys Full Disk Access to a binary's code signature; Go links ad-hoc,
with no designated requirement to match, so macOS falls back to hashing the
code and **every `go install` silently revokes the grant** while the entry
still looks enabled in System Settings. Signing with a stable identity stores
the grant against the identity instead, so rebuilds keep it. You need a
code-signing identity in your own keychain — `scripts/sign.sh` explains how to
create a self-signed one in five minutes if you have no Developer ID.

**Do this first.** Four kinds of evidence decay if nothing captures them, and
`harvest` preserves all four:

- **`knowledgeC`**, the only source that directly observes presence, on a
  ~28-day window.
- **Biome's `ScreenTime.AppUsage` stream**, now the source of application
  attribution — `knowledgeC`'s `/app/usage` stalled for a full day in July 2026
  and resumed on its own, so it is a cross-check rather than something to depend
  on. Biome fills fixed 512 KB segments in place and eventually discards old
  ones.
- **Motion's Autosave Vault**, which Motion prunes for a reason nobody has
  confirmed yet.
- **File modification times**, which decay invisibly: a file records only its
  *last* edit, so work done today on a file last touched weeks ago doesn't add
  evidence for that earlier day, it deletes it. This one needs roots
  configured before it does anything — see
  [Harvesting presence data](docs/content/docs/harvest.md).

Data not captured before it is pruned or overwritten cannot be recovered by
any means. Every other source Timetattle reads persists indefinitely and can
be parsed at leisure.

`harvest` then ingests what it captured into an event store at
`~/.timetattle/timetattle.db`, reading every source in full on every run.
Running it twice adds nothing: the store deduplicates by content. Point it
somewhere else with `-store`.

One unreadable source loses only its own evidence — the other captures and the
whole ingestion still run — and the command exits non-zero naming every source
that failed.

`timetattle status` is how you find out that it stopped. It reads the recorded
history of every source and says when each last ran, whether it succeeded, and
how long it has been failing, distinguishing "ran and found nothing" from
"never ran at all". A source that reads once per repository reports the total
across all of them, and says `partial` — never `ok` — when it read some and
lost others; `-inputs` says which. Run it when a day looks emptier than it was.

Both require Full Disk Access, granted **per binary** and keyed to that
binary's code signature — which is why `install-agent.sh` signs before it
schedules. Grant it once after the first install, and again only if the
signing identity changes. Left unsigned, the grant is keyed to a hash of the
code instead and every `go install` silently revokes it while the entry still
looks enabled; that cost this project 44 hours of capture on 2026-07-30 and 33
hours three days earlier. See
[Harvesting presence data](docs/content/docs/harvest.md) for how to recognise a
revoked grant, why scheduled runs need their own, and why nothing cheaper than
hashing a Biome segment can tell whether it changed.

## How it thinks about time

**Almost every source yields point observations. Presence is the only source
that directly observes an interval.** Commits, file saves, file births and
Claude messages are all instants. Screen-on is the exception.

Duration is therefore always inferred by clustering *across* sources, never
interpolated *within* one. Ignoring this produces spectacular errors: naively
treating each file's creation-to-modification span as working time claimed
70+ hours on a 13.9-hour day.

Two metrics, which cannot collapse into one:

- **Engaged** time is per-project and may overlap, because concurrent work on
  two projects is billable to both. It drives billing and effective rate, and it
  counts work done with the screen off exactly as it counts work done with the
  screen on.
- **Presence** is wall clock and is never double-counted. It is the
  denominator for capacity — a half-time retainer measured against engaged
  time could read as satisfied at 30% of real attention. **It is not screen-on
  time**, and it deliberately includes hours nobody was at the keyboard for: a
  dark backlight observes a shut lid, and a render, a compile or a coding agent
  working behind one is capacity being spent.

**Both figures are mostly inference, and the inference is not yet fitted.**
67.3% of the presence in the live store is margin — lead-in, tail-out and
bridged silence — that the clustering parameters invented, and 85.3% of the
engaged hours are. `timetattle review`
is where that gets calibrated: it puts a whole working day on one screen and
confirms it in one keystroke, and a confirmed day says whether *every* instant
in it was work rather than only where work was. Nothing downstream is believable
until enough days have been through it.

## Documentation

- [Orienting cold](docs/content/docs/orient.md) — the `orient` command, the
  boundary against `status` and `metrics`, the five different things that have
  to happen to evidence no project owns, and why a count of observations is
  never quoted as an estimate of time.
- [Fitting the terminal](docs/content/docs/terminal.md) — why every report
  shortens itself to the terminal, why a pipe is never shortened at all, and the
  order a table gives up its room in.
- [Harvesting presence data](docs/content/docs/harvest.md) — the `harvest`
  command, what a snapshot contains, and what it deliberately discards.
- [Adding an evidence source](docs/content/docs/sources.md) — teaching
  Timetattle about a new application, usually with a JSON manifest and no code.
- [Filesystem observations](docs/content/docs/filesystem.md) — turning a
  project's own birth and modification times into durable events, and why
  that is a different job from `internal/filedays`.
- [Git history](docs/content/docs/git-history.md) — reading a repository's
  own log and reflog as evidence, and why they cannot share a clock.
- [Job task databases](docs/content/docs/job-tasks.md) — reading the `job`
  tool's own `.jobs.db` as evidence of what the work *was*, why planning
  counts, and why an expired claim is never given an end.
- [Registering a project](docs/content/docs/registry.md) — the `register` and
  `consolidate` commands, how a directory's identity is found again from any
  path beneath it, why a worktree resolves to the repository it was forked
  from, and how evidence stranded by a rename is recovered.
- [Application focus from Biome](docs/content/docs/app-focus.md) — where the
  focus intervals come from, and the two places the raw data will mislead you.
- [Presence from knowledgeC](docs/content/docs/presence.md) — reading the
  backlight, the lock and application usage out of a snapshot, and the four
  states a day is actually in.
- [Save-as chains](docs/content/docs/chains.md) — recovering iteration rhythm
  from sibling filenames, for applications that branch instead of editing, and
  why a name is never split at its dashes.
- [Claude Code transcripts](docs/content/docs/transcripts.md) — folding
  subagent sessions into their parent, attributing by `cwd` rather than a
  lossy directory name, and telling a human turn from an unattended one.
- [The event store](docs/content/docs/store.md) — where observations are kept,
  why they are never edited, and why re-reading the same window is free.
- [Sittings and configuration](docs/content/docs/sessions.md) — the
  `sessionize` command, the idle threshold nobody has fitted yet, and the one
  file Timetattle is configured with.
- [Engaged time and presence](docs/content/docs/metrics.md) — the `metrics`
  command, the two normalisations, why every engaged hour in this tool is
  inferred rather than observed, and the three separate reasons a stretch of
  presence belongs to no project.
- [The dashboard](docs/content/docs/dashboard.md) — the `dashboard` command,
  `metrics`' human-facing sibling: the same scoping and data, a trailing
  `-days` window instead of an open span, and color detected once and shared
  by every section.
- [Adjustments](docs/content/docs/adjustments.md) — the `adjust` command, why
  human corrections are an overlay keyed to wall clock rather than an input to
  the clustering, and how "I reviewed this day and cannot remember it" is
  recorded as distinct from both confirming and correcting.
- [Reviewing a day](docs/content/docs/review.md) — the `review` command, one
  keystroke per day, why confirming a day is a complete accounting of it rather
  than a shrug, and the four states a day can be in.
- [Fitting the parameters](docs/content/docs/fitting.md) — the `fit` command,
  why the gap is inert below the margins, why the best score is usually a
  plateau rather than an answer, and how a reviewer confirming what he was shown
  produces labels a fit cannot learn from.
- [Releasing](docs/content/docs/distribution.md) — how a version reaches
  Homebrew, and how the binary is signed.
- [Running the tests](docs/content/docs/testing.md) — the two test runs, why
  the ordinary one does not touch your machine's data, what that means it has
  not verified, and the gate that keeps the two from merging back together.
- [The publication gate](docs/content/docs/confidentiality.md) — what must
  never reach a public mirror, the pre-commit check that hunts it, where its
  list of names comes from now that it derives one instead of being handed
  one, and why it refuses to answer rather than answering wrongly.

One thing sits outside that list. `project/` holds dated design documents and
measurement findings, several of which correct each other in place; that trail is
part of the record rather than a draft of it. It also holds
[`project/backlog.md`](project/backlog.md), which records work deliberately
deferred — what it is, why it is parked, and what would un-park it — so an
absence is legible as a decision rather than an oversight.

## Development

Install the pre-commit hook once per clone. Git does not copy hooks when it
clones, so this is easy to skip and hard to notice — the checks simply never
run, and nothing says so.

```sh
ln -sf ../../scripts/git-hooks/pre-commit .git/hooks/pre-commit
```

A symlink rather than a copy, so edits to the script take effect without
reinstalling. The hook runs the confidentiality gate first, then `go vet`,
`go fix`, `gofmt`, `go mod tidy`, `go test` and `go build`, and refuses loose
`*.go` files directly under `scripts/` — those escape `./...` and would go
unchecked.

The gate needs one piece of setup, once, because the names it hunts live outside
the repository and it refuses to report clean when it cannot find them:

```sh
make classify > ~/.timetattle/publication-gate.json   # read it before saving it
scripts/gatecheck                                     # is the gate wired, and does it fire?
```

[The publication gate](docs/content/docs/confidentiality.md) has the argument:
what the gate hunts, why the hook asks a narrower question than `make gate`
does, and why CI is not where this check can live.

### The two test runs

```sh
go test ./...                  # machine-independent. Green means green.
go test -tags realdata ./...   # the corpus checks, on a machine that has a corpus.
```

The first reads nothing under your home directory and nothing beside the
checkout, so it runs identically anywhere — and it therefore **does not run the
real-data checks at all**. They are not skipped; behind `//go:build realdata`
they are not compiled. The pre-commit hook runs the first one, which means a
commit that passed every hook has not been checked against real data. Run the
second before merging anything that changes how evidence is read.

`scripts/foreignsuite` runs the suite with `HOME` pointed at an empty directory
and prints what skipped, and `internal/corpusgate` fails the ordinary suite if
an untagged test starts reading the machine again.
[Running the tests](docs/content/docs/testing.md) has the whole argument,
including why the corpus checks exist rather than being replaced by fixtures.

## Privacy

Timetattle reads personal data and is built to keep as little of it as
possible. Snapshots discard `knowledgeC`'s metadata table entirely — web page
URLs, serialized Siri interactions, contact identifiers and notification
identifiers — keeping only event timing and application bundle identifiers.

`~/Library/Biome/streams/restricted` holds 153 streams on a normal Mac,
including Safari navigations, Mail harvesting and Bluetooth proximity.
Timetattle copies exactly one of them, `ScreenTime.AppUsage`, named
explicitly in `snapshot.Streams`. It never globs and never copies the parent
directory, and a test fails if that list changes — so the capture cannot be
widened by accident.

Everything stays on the machine. There is no network component. Client
contract values live in global configuration, never inside a repository, so
they cannot be committed by accident.

## Author and license

By Ben Syverson. [MIT](LICENSE).
