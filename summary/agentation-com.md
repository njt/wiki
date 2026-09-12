---
url: https://www.agentation.com/
title: "Agentation — Visual feedback for agents"
author: unknown
date_fetched: 2026-09-11
date_published: n.d.
topics:
  - developer-tools
  - agent-coding-workflow
---

Agentation is a browser tool that turns UI annotations into structured context for AI coding agents. You click an element on a rendered page, write a note, and copy out formatted markdown that an agent can act on — or, with MCP, skip the copy-paste entirely and let the agent see what you're pointing at in real time.

The annotation isn't free-text opinion; it carries machine-actionable coordinates: the element's CSS selector, its source file path, the React component tree it lives in, and its computed styles. The pitch is that instead of describing "the blue button in the sidebar" and hoping the agent guesses right, you hand it `.sidebar > button.primary` and it greps for that directly.

The tool is bidirectional. Through MCP integration and an "Annotation Format Schema," agents can list, clarify, resolve, and clear annotations — "your feedback becomes a conversation, not a one-way ticket into the void." A short best-practices section counsels specificity, one issue per annotation, and including expected-vs-actual context.

Licensing is free for individuals and companies for internal use, with a commercial license required only for redistribution. The page is a product landing page — no pricing, no schema spec, no implementation detail.
