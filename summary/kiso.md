---
url: https://oak-invest.github.io/kiso/
title: Kiso
author: oak-invest
date_fetched: 2026-07-03
date_published: unknown
---

# What is Kiso?

Kiso is a publishing engine that processes **Open Knowledge Format (OKF)** bundles and converts them into static websites. It "turns Open Knowledge Format (OKF) bundles into static websites for humans and AI agents."

The core functionality is a `build` command that takes an OKF bundle directory as input and outputs a static website:

```
kiso-cli build --source=<input_dir> --destination=<output_dir>
```

The generated output includes the original Markdown files, generated HTML pages, an `llms.txt` file, and a `sitemap.xml`.

## Key Features

### First Principle — OKF as Source of Truth

1. **Structured Markdown** — Content stays easy to edit, review, track changes (diff), and version with Git.
2. **Explicit metadata** — Pages keep enough context to be validated, linked, and rendered consistently.
3. **Agent friendly** — Generated HTML keeps clear links back to the original Markdown files.

### Three-Step Workflow

- **Browse** — Readers get a structured static website with navigation and readable pages.
- **Inspect** — Each page can link back to its Markdown source for review and reuse.
- **Publish** — The output is plain static content, hostable locally or over HTTP.

### GitHub Action

Kiso can be integrated into CI via `oak-invest/kiso/applications/kiso-cli-action@v0.1.2`, with configurable `command`, `source`, and `destination` inputs for automatic building and publishing to GitHub Pages.

## Technical Details

- Language: Java (88.6%), HTML (7.9%), CSS (2.2%)
- License: Apache 2.0
- Latest release: v0.1.2 (Jun 26, 2026)
- 122 commits, 11 stars
- OKF specification: https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md
- Contact: stephane.traumat@oak-invest.com
