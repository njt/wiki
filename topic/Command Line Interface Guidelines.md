# Command Line Interface Guidelines

The canonical open-source reference for modern CLI design, authored by Docker Compose co-creators Aanand Prasad and Ben Firshman alongside Carl Tashian (Smallstep) and Eva Parish. Covers philosophy, concrete guidelines, and anti-patterns across help text, output formatting, error handling, arguments/flags, subcommands, configuration, environment variables, naming, distribution, and analytics. The organizing thesis: CLIs have shifted from machine-first to human-first, and design should follow.

---

## Key Quotes

> "The command line of the past was *machine-first*: little more than a REPL on top of a scripting platform."

The historical diagnosis that sets up the entire guide. The authors argue general-purpose interpreted languages (Python, Ruby, JS) have absorbed the scripting role, leaving CLIs free to become "human-first: a text-based UI."

> "Expect the output of every program to become the input to another, as yet unknown, program."

The UNIX pipes ethos restated for the JSON era. This is the guide's tightrope walk: design for humans at the terminal AND for machines in pipelines. The `--plain` and `--json` flags are the escape hatch — human-readable by default, machine-parseable on request.

> "If you change state, tell the user."

Four words that indict half the CLIs in existence. `git push` is held up as the exemplar — it narrates what's happening as it happens. Most tools either say nothing (silent success is indistinguishable from silent failure) or dump debug logs (noise that drowns signal).

> "The user is conversing with your software, whether you intended it or not."

The most quietly radical claim in the document. CLI usage isn't a single atomic invocation — it's a back-and-forth of trial, error, exploration, and correction. Designing for conversation means suggesting corrections on typos, showing intermediate state, confirming before destruction, and hinting at the next command.

> "Catch errors and rewrite them for humans."

Not "display errors." *Rewrite* them. The guide insists errors should guide users toward solutions, not just report what went wrong. The example — "Can't write to file.txt. You might need to make it writable by running 'chmod +w file.txt'" — is the standard most CLIs fail to meet.

> "Responsive is more important than fast."

A UX insight that's easy to miss in engineering discussions. Print something within 100ms. Show a spinner before a network request. The user's perception of speed is shaped by the first byte of feedback, not total wall time.

> "Do not read secrets directly from flags."

Flag values are visible in `ps` output and shell history. This one guideline, if universally followed, would eliminate an entire class of credential-leak vulnerabilities. The alternatives — `--password-file`, stdin, credential files, secret management services — are all listed.

> "Make the default the right thing for most users."

Configuration is good. Most users won't find or remember your flags. The defaults are your product.

---

## Key Themes

- **#concept** Human-first CLI design — the terminal as text-based UI, not a thin wrapper over a scripting platform
- **#pattern** Conversation as interaction model — CLIs are back-and-forth dialogues, not single atomic invocations
- **#pattern** Dual-channel output — stdout for primary data (pipeable), stderr for messaging (visible to humans), `--json`/`--plain` for machine consumers
- **#pattern** Progressive disclosure — concise output by default, `--help` for detail, `--verbose` for diagnostics, web docs for comprehensive reference
- **#concept** Subjective robustness — beyond error handling: the *feel* of solidity that comes from anticipating misuse and recovering gracefully
- **#pattern** Escalating confirmation — mild/moderate/severe risk tiers with corresponding confirmation requirements, always overridable via `--force`
- **#concept** TTY-awareness — detecting whether a human is reading and adapting behavior (color, animations, prompts, pagers)
- **#pattern** Standard flag vocabulary — `-a/--all`, `-d/--debug`, `-f/--force`, `--json`, `-n/--dry-run`, `-q/--quiet`, `--version` as a shared language across tools
- **#concept** Crash-only software — defer cleanup to next run, enable immediate safe exit on failure

---

## Critical Analysis

This is the best single document on CLI design I've read. It earns its authority through the unusual combination of: (1) authors who've built widely-used CLIs (Docker Compose), (2) concrete, falsifiable guidelines rather than vague principles, (3) willingness to acknowledge when conventions should be broken.

The "human-first" framing parallels [[Make Better Documents]], Anil Dash's field manual for business communication, where every rule traces back to the same question: what does the reader need? Dash and clig.dev share the insight that communication is a designed experience where the recipient's context matters more than the author's intent — whether the medium is a CLI or a slide deck.

The "human-first" framing is correct but incomplete. The guide was published in 2021, before coding agents became a significant CLI consumer. Reading it alongside [[10 Principles for Agent-Native CLIs]] reveals the tension: clig.dev optimizes for the human at the terminal; Chow optimizes for the agent at the API. The two aren't contradictory — Chow's principles (structured output, teachable errors, bounded responses) are essentially clig.dev's machine-readable channel recommendations taken to their logical conclusion. But clig.dev's human-first defaults (color, progress bars, pagers) are exactly the things Chow says break agent consumption. The resolution isn't one over the other — it's clig.dev's own insight applied symmetrically: *detect your consumer*. TTY awareness for humans; structured output for agents. The guide already has the architecture for this (`--json`, `--plain`, TTY detection); it just needs the agent use case added to the decision matrix.

The strongest section is **errors**. "Catch errors and rewrite them for humans" is deceptively simple. Most CLI authors believe they're already doing this. They're not. They're passing through library error messages written by people who didn't know what command the user was running, what they were trying to accomplish, or what they should do next. A good error message is a diagnosis and a prescription. The guide's concrete example — suggesting the exact `chmod` command — sets a bar that genuinely changes how you evaluate CLI quality.

The **configuration** section is the most practical taxonomy I've seen. The three-category split (varies per invocation → flags; varies per project/user → env vars + config files; stable within a project → version-controlled command-specific files) eliminates the endless "should this be a flag or a config file?" debates. The XDG Base Directory endorsement (`~/.config`) is correct and still under-adopted. The warning about `.env` files — "don't use them as a substitute for proper config files" — is important and rarely stated.

The **robustness** guidelines contain the document's most underrated insight: "Responsive is more important than fast." Print something within 100ms. This is a UX law that most CLI authors violate by doing network requests before producing any output. The user stares at a blank terminal wondering if the command even launched.

What's missing: the guide is light on testing. There's no section on how to verify your CLI follows these guidelines — no mention of integration tests, output snapshot testing, or accessibility testing for screen readers. The "Further Reading" section is surprisingly thin (five entries) given the breadth of the topic. The guide doesn't address shell completion, which is table-stakes discoverability in 2026. And the analytics section, while principled, doesn't grapple with the reality that most CLI maintainers have no visibility into how their tools are actually used — the alternative of "talk to your users" is noble but scales poorly.

The guide's greatest contribution is its existence as a shared reference point. Before clig.dev, CLI design advice was scattered across man pages, blog posts, and oral tradition. Now there's a URL you can point to. That's infrastructure.

---

*Sources: [[raw/clig-dev]]*
*Last updated: 2026-07-18*
