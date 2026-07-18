---
url: https://clig.dev/
title: Command Line Interface Guidelines
author: Aanand Prasad, Ben Firshman, Carl Tashian, Eva Parish
date_fetched: 2026-07-18
date_published: 2021
site: clig.dev
repository: cli-guidelines/cli-guidelines
---

# Command Line Interface Guidelines

*An open-source guide from [cli-guidelines/cli-guidelines](https://github.com/cli-guidelines/cli-guidelines) on GitHub*

## Authors

The guide is authored by **Aanand Prasad** (Squarespace engineer, Docker Compose co-creator), **Ben Firshman** (Replicate co-founder, Docker Compose co-creator), **Carl Tashian** (Smallstep engineer), and **Eva Parish** (technical writer). Design by Mark Hurrell.

## Foreword

The authors reflect on how CLI computing has evolved since the 1980s. Back then, "the command line of the past was *machine-first*: little more than a REPL on top of a scripting platform." Today's CLI landscape is different — general-purpose interpreted languages have reduced shell scripting's role, and the modern command line is "human-first: a text-based UI that affords access to all kinds of tools, systems and platforms." The guide revisits UNIX best practices for modern CLI design.

## Introduction

The document covers both high-level philosophy and concrete guidelines (leaning toward the latter). It doesn't cover full-screen terminal programs like emacs/vim. It's language-agnostic. The intended audience includes CLI creators looking for design principles, professional CLI UI designers, and anyone wanting to avoid "obvious missteps of the variety that go against 40 years of CLI design conventions."

## Philosophy

### Human-first design

Traditional UNIX commands assumed they'd be used primarily by other programs. Today, many are used mainly by humans but still carry that legacy. "If a command is going to be used primarily by humans, it should be designed for humans first."

### Simple parts that work together

A core UNIX tenet: small, cleanly-interfaced programs combine to build larger systems. Pipes, stdin/stdout/stderr, signals, exit codes, and plain text ensure composability. JSON adds structure for web integration. "Designing for composability does not need to be at odds with designing for humans first."

### Consistency across programs

Terminal conventions become muscle memory. CLIs should follow existing patterns — that's what makes them "intuitive and guessable" and users efficient. But when convention compromises usability, breaking from tradition may be warranted, if done carefully.

### Saying (just) enough

"The terminal is a world of pure information." Too little output (silent hangs) frustrates; too much (debugging dumps) overwhelms. Getting this balance right is "absolutely crucial if software is to empower and serve its users."

### Ease of discovery

GUIs naturally make functionality discoverable — everything's on screen. CLIs are assumed to require memorization, but "Discoverable CLIs have comprehensive help texts, provide lots of examples, suggest what command to run next."

### Conversation as the norm

"Running a program usually involves more than one invocation." Users learn through trial and error, exploring, dry-running operations — a back-and-forth with the software. Acknowledging this conversational nature lets you design better: suggest corrections, clarify intermediate state, confirm before destructive actions. "The user is conversing with your software, whether you intended it or not."

### Robustness

Robustness is both objective (graceful error handling, idempotency) and subjective (the feel of solidity). "Subjective robustness requires attention to detail and thinking hard about what can go wrong." Simplicity also aids robustness — complex code tends toward fragility.

### Empathy

CLI tools are a programmer's creative toolkit and should be enjoyable. "Delighting the user means *exceeding their expectations* at every turn, and that starts with empathy."

### Chaos

The terminal world is messy with inconsistencies everywhere. Yet this chaos has been a source of power — "the terminal...places very few constraints on what you can build." Sometimes you must break rules; "Do so with intention and clarity of purpose."

## Guidelines

### The Basics

- **Use a command-line argument parsing library.** Many language-specific options are listed: docopt, Cobra, clap, Click, Argparse, Typer, TTY, oclif, clikt, picocli, optparse-applicative, and others.
- **Return zero exit code on success, non-zero on failure.** Map non-zero codes to key failure modes.
- **Send primary output to `stdout`** — that's where piping sends things.
- **Send messaging to `stderr`.** Logs, errors, etc. — this keeps them visible to users without feeding into piped commands.

### Help

- Display extensive help when `-h` or `--help` is passed, including for subcommands.
- When run without required arguments, display concise help: program description, 1–2 examples, flag descriptions (if not too many), and an instruction to use `--help` for more. The `jq` command exemplifies this.
- `-h` and `--help` should both work; don't overload `-h`. For git-like tools, `help`, `help subcommand`, and `subcommand --help` should all work.
- Provide a support path (website/GitHub link).
- Link to web documentation, ideally anchored to specific pages for subcommands.
- "Lead with examples" — users prefer examples over other docs. Show actual output when helpful.
- If you have too many examples, put them elsewhere (cheat sheet or web page).
- Display the most common flags/commands first in help text (Git's output is the model).
- Use formatting (bold headings) in a terminal-independent way.
- Suggest corrections when users make mistakes: "`brew update jq` tells you that you should run `brew upgrade jq`." Ask before auto-correcting — assuming what they meant can be dangerous, especially for state-modifying operations.
- If `stdin` is interactive but the command expects piped input, display help and quit rather than hanging.

### Documentation

Help text gives brief, immediate guidance. Documentation provides full detail — purpose, scope, and complete usage.

- **Provide web-based documentation** — searchable, linkable, the most inclusive format.
- **Provide terminal-based documentation** — fast, version-synced, offline-capable.
- **Consider providing man pages.** Many users reflexively check `man mycmd`. Tools like ronn can generate both man pages and web docs.
- Make terminal docs accessible via the tool itself (e.g., `npm help ls` as equivalent to `man npm-ls`).

### Output

- "Human-readable output is paramount." Detect TTY status to know if a human is reading.
- Have machine-readable output where it doesn't hurt usability. "Expect the output of every program to become the input to another, as yet unknown, program."
- Use `--plain` when human-readable formatting breaks machine parsing (e.g., multi-line table cells → one record per line).
- Use `--json` for structured output, enabling complex data handling and web integration via `curl` and `jq`.
- Display output on success but keep it brief — but "It's rare that printing nothing at all is the best default." Use `-q` to suppress non-essential output for scripts.
- "If you change state, tell the user." `git push` is the exemplar — it explains exactly what's happening.
- Make system state easy to view (like `git status` which shows current state and hints for modifying it).
- "Suggest commands the user should run" — helps users learn workflows and discover functionality.
- Actions crossing internal/external boundaries should normally be explicit (reading/writing files, remote server calls).
- Use ASCII art for information density — `ls` permissions format is the example.
- "Use color with intention" — highlight key items, use red for errors. Overuse dilutes meaning.
- Disable color when not in a TTY, when `NO_COLOR` is set, when `TERM=dumb`, when `--no-color` is passed, or when a program-specific env var is set.
- If `stdout` isn't a TTY, disable animations (prevents progress bars from "turning into Christmas trees in CI log output").
- Use symbols and emoji where they add clarity — the yubikey-agent example uses emoji for structure and attention-grabbing.
- Don't output information only understandable by the software's creators unless in verbose mode.
- Don't treat `stderr` like a log file by default — skip log level labels and extraneous context unless in verbose mode.
- "Use a pager (e.g. `less`) if you are outputting a lot of text." Recommended `less` options: `-FIRX`. Only page if stdin/stdout is interactive.

### Errors

- "Catch errors and rewrite them for humans." Guide users toward solutions ("Can't write to file.txt. You might need to make it writable by running 'chmod +w file.txt'").
- Signal-to-noise ratio matters. Group similar errors under a single header instead of repeating.
- Put the most important info at the end of error output. Use red text intentionally and sparingly.
- For unexpected errors, provide debug/traceback info and bug submission instructions. Consider writing debug logs to a file instead of printing them.
- Make bug reporting effortless — provide pre-populated URLs.

### Arguments and flags

Terminology note: *arguments* are positional parameters; *flags* are named parameters with `-` or `--` prefixes.

- "Prefer flags to args" — more typing but clearer, more future-proof.
- Have full-length versions of all flags (both `-h` and `--help`).
- Reserve one-letter flags for commonly used options.
- Multiple arguments are fine for simple actions against multiple files (`rm file1.txt file2.txt`).
- "If you've got two or more arguments for different things, you're probably doing something wrong" — except for primary actions like `cp <source> <destination>`.
- Use standard flag names where conventions exist. Listed common flags: `-a/--all`, `-d/--debug`, `-f/--force`, `--json`, `-h/--help`, `-n/--dry-run`, `--no-input`, `-o/--output`, `-p/--port`, `-q/--quiet`, `-u/--user`, `--version`, `-v` (ambiguous for verbose/version).
- "Make the default the right thing for most users" — configuring things is good, but most won't find and remember flags.
- Prompt for user input when arguments or flags are missing. Never *require* a prompt — always allow flags/args. Skip prompting if `stdin` isn't a TTY.
- Confirm before dangerous actions, with escalating confirmation for mild/moderate/severe risks. Severe risks might require typing the resource name. Always scriptable via `--force` or similar.
- Support `-` to read from stdin or write to stdout when files are involved.
- For optional flag values, allow a special word like "none" rather than a blank value.
- Make arguments, flags, and subcommands order-independent where possible.
- "Do not read secrets directly from flags" — flag values leak into `ps` output and shell history. Use `--password-file` or stdin instead.

### Interactivity

- Only prompt if `stdin` is a TTY. Otherwise, show an error telling the user what flag to pass.
- "If `--no-input` is passed, don't prompt or do anything interactive" — fail and explain how to pass needed info as flags.
- For password prompts, don't echo the typing.
- Make escape clear and consistent. Ctrl-C should always work. For wrappers (SSH, tmux), document how to exit.

### Subcommands

- Be consistent across subcommands with flag names and output formatting.
- Use consistent naming for multiple subcommand levels. `noun verb` or `verb noun` both work; `noun verb` is more common (e.g., `docker container create`).
- Avoid ambiguous or similarly-named commands (like "update" vs "upgrade").

### Robustness (guidelines section)

- Validate user input everywhere — check early, bail before damage, make errors understandable.
- "Responsive is more important than fast." Print something within 100ms. Show something before network requests.
- Show progress for long operations. Good progress bars make programs feel faster. Show estimated time remaining or animation so users know it's still working. Libraries mentioned: tqdm (Python), progressbar (Go), node-progress (Node).
- Do things in parallel thoughtfully — avoid confusingly interleaved output. Use libraries that natively support multiple progress bars. If errors occur, surface the logs (don't hide them behind progress bars).
- Make things time out — configure network timeouts with reasonable defaults.
- "Make it recoverable" — transient failures (e.g., network drops) should allow retry without starting over.
- "Make it crash-only" — defer cleanup to the next run, enabling immediate exit on failure.
- "People are going to misuse your program" — plan for scripts, bad connections, parallel instances, and exotic environments.

### Future-proofing

- Keep changes additive — add new flags instead of breaking old ones.
- Warn before breaking changes — tell users about deprecation in the program itself, show them how to adapt, and ideally detect when they've updated their usage.
- Changing human-oriented output is usually OK — encourage `--plain`/`--json` for stable script consumption.
- "Don't have a catch-all subcommand" — it prevents adding any new subcommand without breaking existing scripts.
- "Don't allow arbitrary abbreviations of subcommands" — you'll be stuck unable to add similarly-named commands. Use explicit, stable aliases instead.
- "Don't create a 'time bomb'" — will your command still work in 20 years, or does it depend on a server you control today?

### Signals and control characters

- On Ctrl-C (SIGINT), exit as soon as possible. Say something immediately, add timeout to cleanup.
- If Ctrl-C hits during long cleanup, skip it and tell the user what a second Ctrl-C will do. Docker Compose's two-press shutdown is the model.
- Expect to start in situations where cleanup wasn't completed.

### Configuration

Three configuration categories with recommendations:

1. **Varies per invocation** (debug level, dry-run): Use flags. Environment variables may supplement.
2. **Generally stable, varies per project/user** (paths, color settings, proxy): Use flags and environment variables. Possibly a config file.
3. **Stable within a project** (build files like `Makefile`, `package.json`): Use a version-controlled, command-specific file.

- Follow the XDG Base Directory Specification — use `~/.config` to reduce home directory dotfile clutter.
- If modifying another program's configuration, ask for consent and be specific. Prefer creating new config files over appending to existing ones.
- Configuration precedence (highest to lowest): flags → shell environment variables → project-level config (`.env`) → user-level config → system-wide config.

### Environment variables

- Environment variables are "for behavior that *varies with the context* in which a command is run" — the terminal session.
- Variable names must contain only uppercase letters, numbers, and underscores (don't start with a number).
- Aim for single-line values.
- Avoid commandeering widely used POSIX standard env var names.
- Check general-purpose env vars where applicable: `NO_COLOR`, `FORCE_COLOR`, `DEBUG`, `EDITOR`, `HTTP_PROXY`/`HTTPS_PROXY`/`ALL_PROXY`/`NO_PROXY`, `SHELL`, `TERM`/`TERMINFO`/`TERMCAP`, `TMPDIR`, `HOME`, `PAGER`, `LINES`, `COLUMNS`.
- Read `.env` files where appropriate for directory-specific configuration, using language-specific dotenv libraries.
- Don't use `.env` as a substitute for proper config files. `.env` files have limitations: not commonly in source control, string-only type, poorly organized, encoding issues, often contain secrets.
- "Do not read secrets from environment variables" — they leak into process state, Docker inspect output, systemd logs, and more. Use credential files, pipes, AF_UNIX sockets, or secret management services.

### Naming

The section quotes Neal Stephenson on UNIX's "obsessive use of abbreviations and avoidance of capital letters."

- Make it a simple, memorable word — not so generic that it conflicts (ImageMagick and Windows both used `convert`).
- "Use only lowercase letters, and dashes if you really need to."
- Keep it short but not *too* short — the shortest names are for universal utilities.
- "Make it easy to type" — the `plum` → `fig` rename is the cautionary tale: one-handed awkwardness replaced by smooth-flowing letters.

### Distribution

- Distribute as a single binary where possible. Use tools like PyInstaller for interpreted languages. Otherwise, use the platform's native package manager.
- Language-specific tools (linters, etc.) can assume the user has that interpreter.
- "Make it easy to uninstall" — put removal instructions near the install instructions.

### Analytics

- "Do not phone home usage or crash data without consent." Be explicit about what's collected, why, anonymization practices, and retention.
- Ideally opt-in; if opt-out, clearly communicate on first run and make it easy to disable. Examples: Angular.js (opt-in), Homebrew (with FAQ), Next.js (opt-out).
- Consider alternatives: instrument web docs, instrument downloads, or simply "Talk to your users" directly.

## Further Reading

The guide recommends several external resources:

- *The Unix Programming Environment*, Kernighan and Pike
- POSIX Utility Conventions
- GNU Coding Standards (Program Behavior)
- "12 Factor CLI Apps" by Jeff Dickey
- Heroku CLI Style Guide

---

*This is an open-source guide. Propose changes at [github.com/cli-guidelines/cli-guidelines](https://github.com/cli-guidelines/cli-guidelines). Join the discussion on [Discord](https://discord.gg/EbAW5rUCkE).*
