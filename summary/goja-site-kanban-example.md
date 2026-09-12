---
url: https://github.com/wesen/2026-05-03--goja-hosting-site/blob/main/examples/kanban/scripts/app.js
title: "Goja-Site Kanban Example: Trail Notes App"
author: wesen
date_fetched: 2026-05-15
date_published: 2026-05-03
topics:
  - agent-architecture
---

Complete source of `app.js` from the kanban example in wesen's goja-hosting-site repository. The file implements a "Trail Notes: Cascade Loop" kanban board using goja-site's DSL-based web framework (kanban.dsl, ui.dsl, express). The application supports full CRUD for cards with drag-and-drop, session-based multi-user support, SQLite persistence, and a distinctive field-notes aesthetic. The DSL is a chainable, composable API that builds the board declaratively: columns, data binding, features (search, drag-drop, precise move), custom rendering, and action handlers. The code includes inline migration logic, seed data, CSS-in-JS styling, and both server-rendered HTML pages and a JSON API.

Key sections:
- Database schema and migration (CREATE TABLE, ALTER TABLE with ignoreDuplicateColumn pattern)
- Session-based card isolation (session_id column)
- Kanban board DSL: kanban.board().title().theme().className().columns().data().features().render().actions().build()
- CSS-in-JS stylesheet with box-shadow aesthetic and responsive breakpoints
- Client-side search via data-kb-search attribute
- Card movement with position normalization
- Server routes: GET / (HTML page), GET /style.css, GET /api/cards, POST /cards
- Seed data for a camping trip planning scenario
