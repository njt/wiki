---
url: https://github.com/cpcloud/micasa
date_fetched: 2026-07-05
backfilled: true
---

Home maintenance, from the terminal.

- **When did I last change the furnace filter?**Schedules, due dates, full history. Set it and forget it.
- **What if we finally did the backyard?**Projects from napkin sketch to completion, or graceful abandonment.
- **How much would it cost to…**Quotes side by side, vendor history, and the math to actually decide.
- **Who did we use last time?**Every contractor, every quote, every job. All searchable.
- **How much have I spent on plumbing this year?**Ask a local LLM. Or don't. Offline until you say otherwise.

Vim-style modal keys: `nav` mode to browse, `edit` mode to change things. Sort by any column, jump to columns with fuzzy search, hide what you don't need, and drill into related records.

See the full keybinding reference.

`go install github.com/micasa-dev/micasa/cmd/micasa@latest`Or grab a binary for Linux, macOS, or Windows.

Then:

```
micasa demo         # sample data
micasa              # your house
```
Full docs at micasa.dev/docs.

Pure Go, zero CGO. Built on Charmbracelet + GORM + SQLite. TUI design inspired by VisiData. Developed with AI coding agents (Claude, Claude Code).

PRs welcome, including AI-assisted ones, as long as you've reviewed and curated the code. See the contributing guide.

```
nix develop                      # enter dev shell
go test -shuffle=on ./...        # run tests
```
Apache-2.0. See LICENSE.
