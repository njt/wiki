---
url: https://vale.sh/
title: "Vale — Your document has structure. Your rules should too."
author: Vale project
date_fetched: 2026-10-11
date_published: unknown
topics:
  - guardrails-and-feedback-loops
  - developer-tools
---

Vale's homepage is a compact pitch for a prose linter whose central idea is stated in its tagline: "Your document has structure. Your rules should too." Rather than running blanket checks over a whole file, Vale parses the markup first and lets you scope rules to specific parts of a document — headings, lists, table cells — while keeping code blocks and link URLs out of the way.

The page also lists the breadth of markup formats Vale understands: Markdown, AsciiDoc, reStructuredText, MDX, MyST, Quarto, Typst, HTML, XML, DITA, Org, and QDoc. Twelve formats is the practical claim: the same rule set travels across whatever documentation stack a team already uses.

The homepage is a fragment of the full product story — it covers only the "Scopes" feature — but the framing it leads with is the interesting part. Vale treats prose as something with parseable structure that deterministic rules can be applied to selectively, the same way a code linter treats syntax trees.
