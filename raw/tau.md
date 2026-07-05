---
url: https://twotimespi.dev/
date_fetched: 2026-07-05
backfilled: true
---

An educational coding-agent project

 **Tau** is a small Python coding agent you read like a
              textbook. Watch it stream model output, call tools, manage sessions,
              and grow into a terminal UI — one readable layer at a time.

No hidden machinery. Every moving part is on the page.

Provider adapters turn model responses into provider-neutral events the rest of the agent consumes.

The reusable harness: messages, tools, transcript state, cancellation, queued prompts, sessions.

The coding environment: files, shell, durable sessions, skills, slash commands, and a Textual TUI.

The lesson

Tau answers the questions tutorials skip: What *is* an agent loop? Where do tool calls come from? How does the transcript grow? How do sessions survive the process exiting?

The core idea

The whole lesson is the boundary. A reusable harness must not depend on the terminal, file paths, or Rich rendering. Those wrap the harness — they never live inside it.

`~/.tau/sessions`One job per package. Study the provider layer, harness, and coding app on their own.

The agent emits a stream you can render, test, and export — not control flow buried in callbacks.

Educational, not a toy. Run it as a real terminal agent while reading the code behind it.

Built phase by phase, each with notes on what was added, why, and how it fits.

The inspiration

Tau borrows Pi's architectural lesson — keep the harness, the environment, and the UI apart. Not a line-by-line port; an educational Python take on the same core ideas.

A map for building your own agent: start with events, add a loop, wrap it in a harness, then give it tools and a UI.
