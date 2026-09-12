---
url: https://www.rafa.ee/articles/progressive-enhanced-forms-htmx/
title: Building Progressively Enhanced Forms Using htmx
author: Rafa
date_fetched: 2026-08-07
topics:
  - software-engineering-craft
---

Rafa describes building a bookmark-editing form for their app "ties" using progressive enhancement: the form works without JavaScript, then HTMX adds niceties (active search, loading spinners) when JS is available. The article catalogs the techniques and tradeoffs discovered along the way.

The core challenge is **transient state** — user input that hasn't been persisted yet but must survive server roundtrips. In an SPA, this lives in JS memory; without JS, you use HTML-native mechanisms: form values (server echoes them back), query parameters/paths (survive reloads and bookmarking), and submit button `formaction` attributes (change the submit target per-button).

Rafa advocates a **build-without-JS-first** workflow: write the whole feature with plain HTML forms, then sprinkle HTMX on top. This prevents architecting into corners that require JavaScript. Testing is done by disabling JS in devtools or adding `hx-disable`.

On **hx-target scoping**, Rafa is candid about the tension: narrow swaps cause stale-data bugs when other page regions aren't updated, while full-page swaps are safer but occasionally lose in-flight user input. They default to full-page swaps as the least error-prone option.

The article closes with a curated reading list: Alexander Petros's "Unplanned Obsolescence" blog, _Resilient Web Design_ by Jeremy Keith, and _Plain Vanilla Web_ as a modern browser-capabilities reference.
