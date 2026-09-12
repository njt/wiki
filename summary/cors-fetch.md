---
url: https://tools.simonwillison.net/cors-fetch
title: CORS Fetch Tester
author: Simon Willison
date_fetched: 2026-07-05
date_published: unknown
topics:
  - developer-tools
---

# CORS Fetch Tester

A browser-based debugging utility by Simon Willison. Its purpose: "Send HTTP requests and inspect what the browser lets you see through CORS."

## Features

- **Import from curl** — paste a curl command and have it parsed into the form
- **URL input** — with a note: "If you omit the scheme, `https://` will be added"
- **HTTP method selector**: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
- **Request Headers** section with "Add header" button
- **Request Body** with three options: None, JSON, Form (URL-encoded)
- **Send request** button
- **Response panel** split into:
  - **Headers** — annotated with "Only headers exposed by CORS are visible"
  - **Body** — displayed as UTF-8 text, with Copy and Format JSON buttons
- **Inline Images** section

## How It Works

The tool performs a `fetch()` call from the browser runtime. The browser's CORS implementation automatically enforces cross-origin restrictions. The tool shows only what the browser allows through: headers are filtered through `Access-Control-Expose-Headers`, and response bodies are shown if CORS permits.

A warning notes: "If CORS isn't enabled, the browser will block the response."

## Source

Hosted on Simon Willison's tools subdomain: https://tools.simonwillison.net/cors-fetch
Part of his collection of single-purpose browser-based developer utilities.
