---
url: https://housecat.com/blog/the-gus-stack-go-unix-sqlite
title: The GUS Stack — Go, Unix, SQLite
author: Noah Zoschke
date_fetched: 2026-07-18
date_published: 2026-07-14
site: housecat.com
---

# The GUS Stack — Go, Unix, SQLite

Noah Zoschke describes a software foundation called the GUS Stack (Go, Unix, SQLite) that works well for building apps quickly — especially when using AI agents. He notes that chatting with an agent to spin up an app is now easier than drafting planning docs, and having a "simple and well-designed shared foundation" saves rebuilding from scratch each time.

## Stack Components

**Unix** underpins both development (MacBook/Darwin) and deployment (Linux on platforms like exe.dev, fly.io, or Hetzner).

**SQLite** is praised for its "embedded engine and single-file-on-disk design," making it "the most widely deployed database engine in the world" and an obvious starting point for small apps with persistent disk.

**Go** offers a "batteries included" standard library, stable syntax since 2012, and produces single binaries for fast bootstrapping and quick rebuilds.

The author diverges from exe.dev's "GUTS" stack (which adds TypeScript) by preferring server-side rendering with minimal JS, using HTMX "as much as possible."

## Go Ecosystem Favorites

The article lists several Go libraries:

- **cockroachdb/errors** for errors with stack traces
- **templ** for type-safe HTML templates (paired with HTMX and Tailwind CSS)
- **fuego** for generating an OpenAPI spec from web handlers
- **sqlc** for type-safe code generated from SQL
- **modernc.org/sqlite** for a pure Go SQLite library
- **goose** for SQL and Go migrations
- **dbos** for durable workflows in SQLite

## Automatic Testing Approach

The author emphasizes that "agentic coding works best with fast feedback." For giving agents visual confirmation of rendered apps, he uses headless Chrome via DevTools Protocol through **rod** (library in tests) and **rodney** (CLI for one-off validations). He notes that instructing an agent to "verify the page layout with rodney" catches polish issues, and writing tests that verify DOM state after clicking through happy paths results in features "that work in one shot and don't regress."

## Try It Out

The article points to a GitHub template at `github.com/housecat-inc/scratch`, suggesting one could use it to "build a dinner party RSVP website." It also directs readers to Housecat's main site for a full-blown product example.
