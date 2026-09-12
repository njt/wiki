# Micasa

A modal TUI for tracking home projects, maintenance schedules, appliances, and vendor quotes. Pure Go, vim-style navigation, SQLite persistence. VisiData-inspired interface for browsing and editing household data.

---

## Key Themes

#tool #tui #home-maintenance #go

The premise is simple: your house is a system with maintenance schedules, vendor relationships, project timelines, and cost histories. Most people track this in a mix of spreadsheets, notes apps, and memory. Micasa gives it a proper data model with a terminal interface.

The vendor quote comparison and spending-by-category tracking are the features that justify a dedicated tool over a spreadsheet. "How much have we spent on plumbing this year?" is a question that's hard to answer from scattered receipts but trivial with structured data.

The local LLM integration for cost analysis is a nice forward-looking feature -- ask natural language questions about your home maintenance history without uploading your financial data to the cloud.

## Critical Analysis

This is the kind of tool that's immensely useful if you actually use it, and completely useless if you don't keep it updated. The TUI interface is fast for power users but creates a steep onboarding curve. The `micasa demo` command with sample data is smart -- let people explore before committing their own data.

Honestly, the "developed with AI coding agents (Claude, Claude Code)" note in the README is the most interesting meta-detail. This is a vibe-coded home maintenance app, and it works. The quality bar for personal tools is "does it solve my problem," not "is the code beautiful."

Built on Charmbracelet's TUI components -- the same ecosystem as [[VHS]]. Charm is becoming the de facto standard for terminal UIs in Go.

---
*Sources: [[summary/micasa]]*
*Last updated: 2026-05-14*
