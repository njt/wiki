---
url: https://github.com/meta/astryx
title: "Astryx — Meta's open source design system, built for people and agents"
author: Meta Platforms
date_fetched: 2026-11-11
date_published: 2026-11-11
topics:
  - developer-tools
  - agent-coding-workflow
---

Astryx is Meta's open source React design system, extracted after eight years of internal use where it powered 13,000+ apps and became the company's largest. Built on React 19 and StyleX, it ships 150+ accessible components, seven customizable themes (CSS custom property overrides, no forking), dark mode, page/block templates, and a CLI (`@astryxdesign/cli`) as one system. Consuming it requires no build plugin or styling library: pre-built CSS imports plus typed React components, and overrides work via `className` with Tailwind, CSS modules, or plain CSS — the StyleX authoring layer is invisible to consumers.

Its distinguishing claim is being "built for people and agents": the API, docs, and CLI are designed together so an AI assistant and a human build the same way from the same reference. Concretely, that means a global-installable CLI whose `astryx search`/`component`/`docs`/`template` commands are the single discovery surface; a machine-managed skill block (with a content hash) it injects into AGENTS.md, CLAUDE.md, and equivalent agent instruction files; a `swizzle` command that ejects a component's full source into the consumer's project; and `astryx init` writing agent docs into the repo. Every CLI command also exposes a typed JSON envelope (`--json`), so agents and build tools consume the same API the CLI prints.

The repo is also notable for its governance apparatus: a `docs/specs/` tree of versioned knowledge records with `draft`/`current`/`archived` authority states, an AGENTS.md that routes every change to its owning spec before review, and an `internal/vibe-tests` harness that measures how well LLMs generate correct component code under different naming/API configurations — the design system's conventions are tested against models, not just asserted. Currently in beta, MIT licensed.
