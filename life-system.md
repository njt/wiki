# life-system

A personal life operating system built on plain-text markdown files with Claude Code as the accountability partner. Inspired by John Carmack's .plan files and Benjamin Franklin's daily questions ("What good shall I do this day?"). The files are the source of truth; the AI is the partner who never forgets what you wrote and calls you out when your daily actions drift from your stated priorities.

---

## Key Quotes

> "The files are the source of truth. Claude is the accountability partner who never forgets what you wrote."

## Key Themes

#personal-systems #journaling #plain-text #productivity #Claude-Code

The design philosophy is radically simple: markdown files for your 10-year vision, annual goals, values, daily journal, decision records, and people notes. Claude reads all of it at the start of each session and holds context. No database, no app, no proprietary format -- just files and an AI that remembers what's in them.

The Franklin-inspired daily ritual (morning question, evening reflection) gives the system temporal structure. This isn't just a note-taking system; it's a feedback loop between intention and action, with the AI providing the accountability that most personal systems lack after the first week.

The wiki-link convention (`[[filename]]`) for cross-referencing mirrors the approach used in this very wiki, and connects to [[robot.wtf]]'s git-backed markdown philosophy and [[Rowboat]]'s Obsidian-compatible vault.

## Critical Analysis

The strength is the simplicity. No infrastructure to maintain, no services to run, no databases to back up. Just files. This means it works offline, survives any tool migration, and never locks you in. The tradeoff is that Claude Code needs to read all the files at session start, which means your life plan is bounded by the context window.

The weakness is the same as every personal productivity system: it only works if you use it. The morning/evening ritual requires discipline that most people won't sustain. Claude as accountability partner is an interesting innovation -- it can notice when your daily journal doesn't mention your top annual goal -- but it can't force you to open the terminal.

The anti-goals concept (things you explicitly choose NOT to pursue) is underrated. Most goal-setting systems only track what you want to do, not what you're choosing to avoid. Anti-goals are constraints that clarify priorities.

This is the lightest-weight entry in the personal agent space. Compare with [[mira-OSS]] (PostgreSQL + pgvector + Vault) or [[Hermes]] (Docker container, MCP tools, cron scheduler). life-system is just files and a shell alias.

---
*Sources: [[raw/life-system]]*
*Last updated: 2026-05-14*
