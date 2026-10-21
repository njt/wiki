---
url: https://don-smith.github.io/archie/
title: "Archie — an architecture agent for a code repository"
author: Don Smith
date_fetched: 2026-10-21
date_published: unknown
topics:
  - coding-agents-and-frameworks
  - agent-architecture
---

Archie is an architecture agent that installs into a repository as a set of agent skills plus a pinned runtime and works inside whichever coding agent you already use. It refuses to give one universal architecture verdict; instead it routes between seven named capabilities, each with its own output semantics.

The capabilities: **Architecture Assessment** (a validated fact model of a whole system before redesign), **Architecture Review** (a layer-by-layer review of one bounded module, producing triaged findings and a phased polish plan), **Architecture Docs** (pages with a claims ledger, citing the code they describe), **LikeC4 authoring** (C4 models compiled against one named question), **Conformance onboarding** (observing the import graph of a TypeScript repo before anyone writes rules), **Architecture Contracts** (maintainer-stated dependency rules checked deterministically on every run), and **HTML Design** (self-contained readable output — the site itself is one).

The stance on authority is the load-bearing idea: architecture decisions stay with the maintainers, and Archie makes evidence and consequences visible first. Every finding is labelled fact, inference, or unknown, and the meaning of each result is deliberately kept below approval: review findings are advice to triage, an observed import graph never promotes itself into an active rule (a maintainer writes the rule), and a compiling C4 model has valid syntax, not truth — approved claims and their evidence are what make it true.

Installation is deliberately unglamorous: no npm publish, no `curl | bash`. You clone and read `install.sh` first; it checks Node 24, npm, git, and APM; it leaves behind a pinned set of skills, hash-verified runtime tarballs, and a release record, writes `.agents/skills/` (with symlinks for Claude Code), `.archie/runtime/`, and `.archie/` artifacts beside the code. It never pushes, never tags, never touches your `package.json`, and uninstalling replays the install journal while keeping your assessments. Version 0.3.0, MIT-licensed, verified by use in real repositories rather than a formal gate.
