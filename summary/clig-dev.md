---
url: https://clig.dev/
title: "Command Line Interface Guidelines"
author: Aanand Prasad, Ben Firshman, Carl Tashian, Eva Parish
date_fetched: 2026-07-18
date_published: 2021
topics:
  - software-engineering-craft
---

An open-source guide to CLI design by the co-creators of Docker Compose and
colleagues, covering philosophy and concrete guidelines for building
human-friendly command-line tools.

The philosophy section argues that modern CLIs should be human-first rather than
machine-first — designed for people to discover, converse with, and enjoy. Core
principles include composability (simple parts piped together), consistency with
existing conventions, balanced output (not too little, not too much), and
empathy for the user.

The bulk of the guide is practical, language-agnostic guidelines across 15
sections: basics (exit codes, stdout vs stderr), help text (lead with examples,
suggest corrections), documentation (web and terminal, consider man pages),
output formatting (detect TTY, offer `--json` and `--plain`, use color with
intention, page long output with `less`), error messages (rewrite for humans,
make bug reporting easy), arguments and flags (prefer flags to positional args,
use standard names, confirm before dangerous actions), interactivity (only
prompt for a TTY, don't echo passwords), subcommands (be consistent, prefer
`noun verb`), robustness (validate early, print within 100ms, show progress,
make operations time out and recover), future-proofing (keep changes additive,
warn before breaking, don't use catch-all subcommands), signals (respond to
Ctrl-C immediately), configuration (flags > env vars > config files, follow XDG
Base Directory), environment variables (use for context-varying behavior, don't
store secrets in them), naming (short, memorable, lowercase, easy to type), and
distribution (single binary, easy uninstall).

Notable: the guide takes a firm stance against reading secrets from flags or
environment variables, directing authors toward credential files or secret
management services instead. It also recommends against phoning home analytics
data without explicit consent, ideally opt-in.

The source is maintained on GitHub as an open community reference.
