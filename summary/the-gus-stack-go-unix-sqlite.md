---
url: https://housecat.com/blog/the-gus-stack-go-unix-sqlite
title: "The GUS Stack — Go, Unix, SQLite"
author: Noah Zoschke
date_fetched: 2026-07-18
date_published: 2026-07-14
---

Noah Zoschke proposes the GUS Stack — Go, Unix, SQLite — as a simple,
well-designed foundation for building apps quickly, particularly when paired
with AI coding agents. The argument is that a shared, minimal stack eliminates
the friction of rebuilding scaffolding from scratch each time.

Unix provides the operating system layer across development (macOS/Darwin) and
deployment (Linux on platforms like fly.io or Hetzner). SQLite is chosen for
its embedded, single-file design and ubiquity. Go contributes a batteries-included
standard library, stable syntax, and fast single-binary compilation.

Zoschke prefers server-side rendering with HTMX over the TypeScript-heavy
"GUTS" variant promoted by exe.dev. His recommended Go ecosystem includes
**templ** for type-safe HTML templates, **sqlc** for type-safe SQL-to-Go code
generation, **modernc.org/sqlite** for a pure-Go SQLite driver, **goose** for
migrations, and **dbos** for durable workflows.

A notable section covers using headless Chrome (via the **rod** library and
**rodney** CLI) to give AI agents visual feedback on rendered pages. Writing
DOM-state assertions against happy-path click-throughs lets agents produce
features that work on the first attempt and don't regress.

A GitHub template at `github.com/housecat-inc/scratch` is offered as a starting
point.
