---
url: https://jeapostrophe.github.io/tech/jc-workflow/
title: "Don't Wait for Claude"
author: Jay McCarthy
date_fetched: 2026-05-15
date_published: 2026-03-27
---

# Don't Wait for Claude

Jay McCarthy describes a workflow for running multiple Claude Code sessions in parallel, and introduces `jc`, an open-source macOS app that orchestrates them.

## The Wait

A Claude task takes seven minutes. Most people wait idly. Even with optimized prompts, the bottleneck isn't Claude's speed — it's "the seven minutes where you're doing nothing." An hour yields only about four work cycles.

## The Switch

The obvious fix — working on something else during Claude's execution — runs into a human cognition problem: "Coming back is the hard part."

McCarthy references Boris Cherny's Claude Code workflow of running five parallel sessions with numbered tabs and system notifications, calling this "the state of the art for the human side."

Two common approaches: waiting idly, or making Claude fully autonomous with PR-review-style interaction. The latter means "checking in once an hour instead of once every seven minutes."

Core thesis: "The problem isn't that you can't *run* multiple sessions. It's that you can't *manage* them."

## The State

The solution: externalize your state so resuming a session requires no mental recall. This "isn't extra work. It's the review work you should be doing anyway, just done at the right time."

The eight-step workflow: send instructions → switch away → return via notification → re-read previous notes → review output → annotate as you go → send annotations → repeat.

Each annotation reduces what you need to remember — "by the time you're ready to send, the instruction writes itself."

## The DIY Version

McCarthy tried implementing this manually in Zed with custom keybindings, a TODO.md on one side, diff on the other, and terminal tabs per session. Three problems:

1. **Notes friction:** Jumping from diff pane to TODO, finding the right heading, scrolling past history to the WAIT section — "that's four actions to write one line."
2. **No notifications:** "Claude finishes and nothing happens." No badge, sound, or indicator.
3. **Navigation:** Multiple terminal tabs across projects. Tab names don't indicate which need attention. By the time you find the right one, you've "lost the thread of what you were doing."

"The practice is sound. The manual implementation leaks at every joint."

## The Tool

`jc` (github.com/jeapostrophe/jc) is an open-source macOS app that orchestrates multiple Claude Code sessions. Three-pane layout: Claude terminal, TODO editor, diff view.

Each session gets a section in a TODO.md with a `### WAIT` marker separating sent instructions from draft notes. A key press captures a note below WAIT from any context. Cmd-Enter sends all notes as numbered messages.

A single keybinding cycles through problems by priority: "permission prompts first, then unreviewed diffs, then unsent notes." You pick the next problem, not the next session.

Closing argument: "The difference between four cycles an hour and twelve isn't about Claude getting faster. It's about you getting better."

McCarthy adds: "If you have improvements, have your Claude open a PR against mine. I don't accept human-authored code."
