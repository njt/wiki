---
url: https://oak-invest.github.io/kiso/
date_fetched: 2026-07-05
backfilled: true
---

# Publishing engine for Open Knowledge Format

Kiso is a publishing engine that turns Open Knowledge Format (OKF) bundles into static websites for humans and AI agents.

First principle

## OKF is the source of truth.

### Structured Markdown

Knowledge remains easy to edit, review, diff, and version in Git.

### Explicit metadata

Pages keep enough context to be validated, linked, and rendered consistently.

### Agent friendly

Generated HTML keeps clear links back to the original Markdown files.

Command

## What build does

                        The `build` command generates a static
                        website from an OKF bundle, including the original Markdown files, generated HTML pages,
                        llms.txt, and sitemap.xml.
                    

| source | Any OKF bundle directory like `examples/kb-google-example` | 
|---|---|
| destination | `public`or any static directory | 

GitHub Action

## Use in Github CI

You can use Kiso in a GitHub Action to automatically build and publish your OKF bundle to GitHub Pages or any other static hosting service.

```
 - name: Build with Kiso
   uses: oak-invest/kiso/applications/kiso-cli-action@v0.1.2
   with:
    command: build
    source: examples/kb-google-example
    destination: website/examples/kb-google-example-latest
```
### Browse

Readers get a structured static website with navigation and readable pages.

### Inspect

Each page can link back to its Markdown source for review and reuse.

### Publish

The result is plain static output that can be hosted locally or over HTTP.

## Try Kiso on the Google Analytics sample knowledge base.

The example shows how an OKF bundle becomes a navigable site for Humans and AI agents.
