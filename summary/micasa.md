---
title: "Micasa"
url: https://github.com/cpcloud/micasa
date_fetched: 2026-05-14
section: "Random"
topics:
  - developer-tools
---

# Micasa: Home Maintenance TUI

Modal TUI for tracking home projects, maintenance schedules, appliances, and vendor quotes. Written in pure Go.

## Key Features
- Track maintenance tasks with due dates and complete history
- Project management from concept through completion (including abandoned projects)
- Vendor and quote comparison side by side
- Searchable contractor history
- Spending patterns by category
- Local LLM integration for cost analysis
- VisiData-inspired interface with vim-style modal navigation

## Technical Stack
- Pure Go, zero CGO
- Charmbracelet for TUI components
- GORM with SQLite for persistence
- Developed with AI coding agents (Claude, Claude Code)

Install: `go install github.com/micasa-dev/micasa/cmd/micasa@latest`
Sample data: `micasa demo`
Apache-2.0 license.
