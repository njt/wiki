# DSL-Driven Kanban Boards (Goja-Site)

wesen's goja-site is a hosting platform for JavaScript web apps running on the goja runtime (a Go-based JS engine). Its kanban example demonstrates a chainable DSL architecture where an entire interactive board is built declaratively and mounted onto an Express-style router — database, rendering, drag-and-drop, search, and actions all composed from JavaScript DSLs before a single route is registered.

---

## Key Quotes

> `const board = kanban.board("trail-notes").title("Trail Notes: Cascade Loop").theme("field-notes").className("board").columns(...).data(...).features(...).render(...).actions(...).build();`

This single chain is the entire board definition. Columns, data binding, search mode, drag-drop behavior, card rendering, and move handlers are all composed before `.build()` materializes the board. The DSL is not just a configuration object — it's a fluent builder where each method call adds a layer of behavior.

> `board.mount(app, "/_kanban");`

The composed board is mounted as a route handler on the Express app. The board owns its own rendering, its own API endpoints (through action handlers), and its own client-side behavior. This is the "component as self-contained application" pattern that React Server Components are still working toward.

> `function ignoreDuplicateColumn(fn) { try { fn(); } catch (e) { /* older demo dbs may already have the column */ } }`

The migration strategy: try each ALTER TABLE in sequence, silently catch failures. It's pragmatic and demo-appropriate, but this is exactly the kind of "good enough for now" that becomes production debt. The function name is refreshingly honest about what it does.

---

## Key Themes

#tool #DSL #kanban #pattern #web-framework

**The DSL as application definition.** The `kanban.dsl` and `ui.dsl` modules are not templates or config files — they're JavaScript APIs that compose into a complete application. The pattern is: define your board declaratively, mount it on a router, and the framework handles rendering, state, and client-side interactivity. This inverts the usual web framework model where routes come first and UI is an afterthought.

**Server-side rendering with client-side behavior.** The board renders HTML on the server (`board.render()`) but includes `data-kb-search` attributes and drag-and-drop that work on the client. The action handler (`cardMoved`) runs on the server, updates the database, and returns `{ ok: true, refresh: true }` — the framework handles re-rendering. This is a clean separation: the developer writes server logic, the framework handles the client update.

**Inline everything.** The stylesheet is a JavaScript string returned by `stylesheet()`. The database migration runs at module load time. The seed data is hardcoded. There are no separate CSS files, no migration tool, no seed scripts. For a demo or small app, this eliminates the file-finding problem that plagues traditional web frameworks. For anything larger, it's a maintainability hazard. The right call for the context.

**Kanban as the universal coordination surface.** The trail-notes theme (camping trip planning) is a clever disguise — this is the same kanban pattern used for agent coordination in [[Managing Agents via Kanban Boards]], [[ralph-ban]], [[weft]], and [[Dorothy]]. The columns (To Do, In Progress, Done, Someday) are the standard agent-task lifecycle. wesen built a general-purpose coordination board and dressed it in hiking gear.

---

## Source-Level Architecture

Reading the full `app.js` (326 lines, 15 functions) reveals the two-DSL architecture in detail:

**Layer 1 — `ui.dsl`**: An HTML generation DSL. `ui.page()`, `ui.form()`, `ui.input()`, `ui.select()`, `ui.div()`, `ui.h1()` etc. all return HTML strings. The entire page is one expression tree — no template files, no JSX, just nested function calls that compose into a complete document. The `ui.fragment()` call in card rendering shows this is a proper virtual DOM-like composition model, not string concatenation.

```javascript
return ui.page({ title: "Trail Notes: Cascade Loop" },
  ui.link({ rel: "stylesheet", href: "/style.css" }),
  ui.main({ class: "page" },
    ui.header({ class: "hero" }, /* ... */),
    board.render({ query: filters, session: req.session }),
    ui.footer({ class: "footer" }, /* ... */)
  )
);
```

**Layer 2 — `kanban.dsl`**: A board composition DSL. The builder chain `.columns().data().features().render().actions().build()` is not just configuration — each stage receives a context object with access to the partially-built board. The `.data()` stage binds four callbacks (`cards`, `id`, `column`, `position`, `searchText`) that the framework calls at render time. The `.actions()` stage registers server-side handlers that return response objects like `{ ok: true, refresh: true, card: moved, toast: "Moved card" }` — the framework uses these to decide whether to re-render or just show a toast.

**Session isolation by default.** Every query includes `WHERE session_id = ?`. The `sessionId()` function accepts a string, an object with `.id`, or falls back to `"default"`. This means the same SQLite database serves multiple independent users without any auth middleware — each user sees only their own cards. `seedIfEmpty()` is called on every `listCards()` invocation, which means the first read by any session populates 10 demo cards. Clever but fragile: a concurrent first read could double-seed.

**Position management as a solved problem.** Cards are ordered at 10-point intervals (`(index + 1) * 10`). After every move, both source and destination columns are re-normalized — `normalizeColumn()` iterates every card in the column and rewrites its position. This is O(n) per move, which is correct for hundreds of cards. The `nextPosition()` function uses `COALESCE(MAX(position), 0) + 10` for inserts. The 10-point gap leaves room for manual position tweaks between existing cards, though the code never exploits this.

**Migration as exception-swallowing.** `ignoreDuplicateColumn()` wraps each `ALTER TABLE` in try/catch, silently discarding errors. The comment says "older demo dbs may already have the column." This is a demo-appropriate pattern that would become a production liability — there's no way to distinguish "column already exists" from "table doesn't exist" from "database is corrupt." The honest function name (`ignoreDuplicateColumn`) is better documentation than most migration tools provide.

**The CSS is a 52-line template literal.** Custom properties (`--ink`, `--paper`, `--soft`, `--muted`, `--line`) create a cohesive monospace aesthetic. Box shadows on borders (`box-shadow: 3px 3px 0 var(--line)`) and the `.mascot` rotation (`transform: rotate(-1deg)`) are the only visual flourishes on an otherwise brutalist design. Two responsive breakpoints (1100px and 680px) handle tablet and mobile. The `image-rendering: pixelated` + `filter: grayscale(1) contrast(1.2)` on card images is a deliberate stylistic choice — trail map sketches, not photos.

**Five routes, no middleware.** The Express app registers: `GET /` (HTML page), `GET /style.css` (inline CSS served as `text/css; charset=utf-8`), `GET /favicon.ico` (204, because there isn't one), `GET /api/cards` (JSON list), and `POST /cards` (create + redirect). No auth, no validation beyond a title-required check, no CSRF protection. This is a demo, and it acts like one.

## Critical Analysis

The DSL pattern here is genuinely interesting — not because DSLs are new, but because this particular flavor (JavaScript DSLs composing into a self-contained mounted component) solves a real coordination problem between framework and application code. Most web frameworks force you to spread your application across routes, templates, stylesheets, and client-side JS. goja-site lets you define the whole thing in one place and mount it. That's the right instinct.

The tradeoff is that you're now writing JavaScript for a Go runtime. goja is not Node.js — it's a Go implementation of ECMAScript with its own performance characteristics and limitations. The `require("database")` and `require("kanban.dsl")` calls resolve to Go-backed modules, not npm packages. This means the ecosystem is whatever the goja-site authors built. For the kanban example that's sufficient; for a real application, the absence of the npm ecosystem would be felt immediately.

The most transferable idea is the **builder DSL pattern for UI components**. The chain `.columns().data().features().render().actions().build()` separates concerns cleanly while keeping the definition in one place. Each method in the chain receives a context object with access to the partially-built board, so later stages can reference decisions made earlier. This is the same pattern [[Swamp Club]] uses with Zod-typed models and DAG execution, but goja-site does it with fewer abstractions.

The inline-everything approach (CSS, migrations, seeds) works for the demo scale but represents a choice about where complexity lives. The [[The Dark Factory is a DOT File]] thesis says the spec is the durable artifact and the code is disposable — but this code is both spec and implementation. There's no separate DOT file; the DSL chain *is* the spec. Whether that's elegant or entangled depends on how much the DSL constrains what you can express.

Position normalization after every move (re-numbering all cards in a column at 10-point intervals) is a classic SQLite pattern for ordered lists. It's O(n) per move but fine for hundreds of cards. If this ever needed to scale, the fix would be fractional indexing or a linked list — but for a personal kanban board, the simple approach is correct.

---

*Sources: [[summary/goja-site-kanban-example]], [[summary/goja-site-kanban-app-js]]*
*Last updated: 2026-05-15*
