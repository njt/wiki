# VHS

A terminal recording tool from Charm that generates GIFs and videos from scripted `.tape` files. You write what the terminal should do -- type this, press enter, wait -- and VHS plays it back in a virtual terminal, capturing every frame into a GIF, MP4, or WebM.

---

## Key Quotes

> "Write terminal GIFs as code for integration testing and demoing your CLI tools."

## Key Themes

#tool #cli #developer-experience #testing

The killer insight is treating terminal recordings as code. A `.tape` file is version-controllable, diffable, and reproducible. This means demo GIFs in your README can be regenerated automatically in CI when the CLI interface changes -- no more stale screenshots.

VHS sits at the intersection of testing and documentation. Golden-file testing (generate ASCII output, compare against known-good) is a lightweight integration test that catches regressions in CLI output formatting. The CI/CD integration via GitHub Actions makes this practical for real projects.

The Charm ecosystem (Bubble Tea, Lip Gloss, VHS) is quietly building the best terminal UI toolkit in existence. VHS is the showroom -- it makes Charm's own tools look good in READMEs.

## Critical Analysis

Strengths: deterministic recordings from code, multiple output formats, CI integration. The `.tape` DSL is simple enough that non-developers could write demo scripts.

Weaknesses: requires `ttyd` and `ffmpeg` as dependencies, which adds friction. The real limitation is that `.tape` files are fragile -- any change to prompts, timing, or output format breaks the recording. You're trading manual recording effort for script maintenance effort.

Connects to the broader theme in [[Agentic Coding]] about developer experience tooling. As CLI tools proliferate (driven by agent workflows), good documentation becomes more important, and VHS makes that documentation maintainable.

---
*Sources: [[summary/vhs]]*
*Last updated: 2026-05-14*
