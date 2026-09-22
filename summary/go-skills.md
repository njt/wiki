---
url: https://github.com/spf13/go-skills
title: "Go Skills for Claude Code"
author: Steve Francia (spf13)
date_fetched: 2026-09-22
date_published: 2026-09-17
topics:
  - claude-code
  - guardrails-and-feedback-loops
---

# Go Skills for Claude Code

Six [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills) for Go work, published as a Claude Code plugin marketplace: `go` (idiomatic Go), `cobra-viper` (CLI architecture), `go-spec-reviewer` (design-doc review before implementation), `go-release` (release engineering), `wails` (desktop apps), and `fileflow-pathologize` (safe file operations). Install is `/plugin marketplace add spf13/go-skills`, then `/plugin install <skill>@go-skills`; other agents (Copilot, Cursor) consume the same directories by symlinking them into `.agents/skills`. There is no Go code in the repository — six markdown instruction files are the entire product.

The thesis is a course correction against LLM training data. Francia's claim: models trained on the whole internet generate Java-in-Go-syntax by default and then argue — Clean Architecture layers, `internal/` junk drawers, `pkg/` anti-patterns, BDD frameworks, static worker pools, third-party routers where the Go 1.22 stdlib ServeMux suffices. The skills restate Go's own defaults (flat domain packages, consumer-defined interfaces, channels over mutexes, table-driven tests, stdlib-first) with explicit prohibitions, current through Go 1.25. The golden rule: *clear is better than clever*.

The repo treats its own content as governed infrastructure. Each skill directory is a single-skill plugin (`SKILL.md` + `.claude-plugin/plugin.json`), the root `marketplace.json` lists all six, and a 197-line Python checker enforces `CONTRIBUTING.md` as CI invariants: one-to-one catalog↔directory correspondence, the README skills table listing each skill exactly once, descriptions of at least 40 characters (the trigger surface is load-bearing), tags equal to manifest keywords, one shared version across all six plugins, and tag↔version↔CHANGELOG consistency against git. Released as v1.0.0 on 2026-09-17, under MIT.

Authority is the distribution strategy: the skills are written by the creator of Cobra, Viper, and Hugo and former Go team lead at Google who co-designed Go modules — so the `cobra-viper` skill can document its own library's most common bug (the `AutomaticEnv` + `Unmarshal` gotcha) and the release skill can preach holding v1 forever the way Cobra and pflag have.
