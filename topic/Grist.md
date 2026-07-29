# Grist

Grist is an open-source relational spreadsheet that combines the flexibility of a spreadsheet with the robustness of a database. Think of it as Airtable you can self-host, with Python formulas instead of a restricted formula language, and a self-contained SQLite file as the document format. It has ~233K lines of TypeScript and ~45K lines of Python, with significant French government contributions. The core architectural insight is the two-level action pipeline (User Actions → Doc Actions) flowing through a Python formula engine that uses cooperative exception-driven scheduling for dependency resolution.

---

## Architecture

Grist is a **Node.js + Python hybrid** with a browser-based SPA frontend. The server runs as either a Home Server (user management, API routing, static serving) or a Doc Worker (document processing, formula evaluation, SQLite storage) — or both in a single process. The same codebase (`FlexServer.ts`) determines its role from configuration.

### Layered Architecture

```
Browser (TypeScript SPA, GrainJS framework)
    ↕ WebSocket + HTTP REST
Node.js Server (Express, TypeScript)
    ├── ActiveDoc.ts — document lifecycle dispatcher
    ├── DocStorage.ts — SQLite read/write, action→SQL translation
    ├── GranularAccess.ts — formula-based access control (two-pass)
    └── NSandbox.ts — Python subprocess management
    ↕ stdin/stdout RPC pipes
Python Data Engine (sandbox/grist/)
    ├── engine.py — formula evaluation orchestrator
    ├── depend.py — dependency graph (Nodes, Edges, Relations)
    ├── gencode.py — schema→Python code compiler
    ├── relation.py — row-to-row dependency mapping
    └── useractions.py — all User Action implementations
```

### Document Format

A Grist document is a `.grist` file — a SQLite database. All data, metadata, formulas, attachments, and undo history live in one file. The schema has two tiers:

- **`_grist_*` tables** (managed by Python): Tables, columns, views, ACL rules, attachments, page layout
- **`_gristsys_*` tables** (managed by Node): Action log, undo history, file storage, plugin data

### Request Lifecycle

1. User edits a cell → `UpdateRecord` User Action created in the browser
2. Sent via WebSocket to Doc Worker
3. Node forwards to Python data engine via pipe RPC
4. Python evaluates the action, runs affected formulas, produces Doc Actions (lower-level `UpdateRecord`, `AddColumn`, etc.)
5. Node translates Doc Actions to SQL, updates SQLite
6. Node broadcasts Doc Actions to all connected browsers
7. All tabs update their in-memory data model

This two-level action system (User → Doc) is like a compiler pipeline. The authoritative list of User Actions is in `useractions.py` with the `@useraction` decorator.

## Key Techniques

### Cooperative Formula Evaluation via OrderError

The most ingenious implementation detail. Instead of topologically sorting a static dependency graph, Grist uses a dynamic, exception-driven scheduler:

1. Mark dirty cells in `engine.recompute_map`
2. Try to evaluate each dirty cell
3. If a dependency hasn't been computed yet, **throw `OrderError`** with the node and row_id of the missing dependency
4. The engine catches it, adds the current cell to the dependency's wait list, moves on
5. When the dependency is computed, waiting cells are re-triggered

This handles dynamic dependencies naturally — a static topological sort can't handle cases where the dependency structure changes based on data values. Circular references are detected via a `_locked_cells` set; if a cell tries to read itself, `CircularRefError` is raised.

The pattern is inspired by the Ninja build system's graph (`depend.py` notes this explicitly).

### Schema-to-Code Compilation

`gencode.py` takes the document schema and generates a Python module via `exec()`:

```python
# Generated code for a "Students" table with formulas
class Students:
    def Name(rec, table):
        return rec.school.Name  # Reference lookup
    def Grade(rec, table):
        return "A"  # literal formula
```

Each formula becomes a method. Reference column types generate typed accessor properties. Formula code is cached by (table_id, col_id, formula_text) to avoid recompilation on every schema change.

### Relation-Based Row Mapping

Dependencies aren't just "column B depends on column A." The `Relation` class specifies **which rows** need recalculation:

- `IdentityRelation` — same table, same row (direct cell reference)
- `ReferenceRelation` — follows a foreign key (row in table B depends on the referenced row in table A)
- `ComposedRelation` — chains of references (e.g., `rec.school.address.zip`)
- `SingleRowsIdentityRelation` — identity that refuses ALL_ROWS propagation (for trigger formulas)

This is critical for performance: when row 5 of `Schools` changes, only rows in `Students` that reference school 5 need formula recalculation — not every student.

### Two-Pass Access Control

`GranularAccess.ts` checks actions **twice** per request:

1. **Before** the Python engine processes them: "Is this user allowed to make this change?"
2. **After** the Python engine produces Doc Actions: "Should this client see these computed results?"

This handles the case where a user edits a cell they can see, triggering a formula recalculation in a column they can't access. The intermediate results are filtered before broadcast. Access rules are themselves formulas evaluated by the Python engine, which can reference cell values, user attributes, and the `user` object.

### Virtual DOM Scrolling

For tables with 10K+ rows, `koDomScrolly.js` renders only visible rows and reuses DOM nodes. As the user scrolls, the same `<div>` elements are repositioned and refilled with new row data. `BaseRowModel` observables are created only for visible rows — for the rest, data stays in typed arrays.

### Action Bundles with Envelope Encryption

Actions are packaged into `ActionBundle` objects with `Envelope`-based recipient addressing. Each envelope contains content encrypted for a specific set of recipients. This enables secure multi-instance synchronization — different clients may receive different subsets of actions based on their permissions. The `EncActionBundle` type adds encryption on top.

## Design Decisions

**Optimized for formula expressiveness**: Full Python syntax. Not a restricted subset. This is the product's core differentiator — Excel users can use Excel functions, Python users get the full language.

**Optimized for portability**: Everything in one SQLite file. Documents are self-contained, can be downloaded, backed up, and moved between servers. Any SQLite tool can read the data.

**Sacrificed memory efficiency**: The entire document (except on-demand tables) is loaded into memory — once in the Python engine, and once per browser tab. This caps practical document size based on available RAM.

**Sacrificed clean frontend layering**: The client codebase has visible legacy — knockout.js alongside the custom GrainJS framework, components scattered across three directories (`ui/`, `components/`, `ui2018/`). GridView.js is acknowledged as "one of the oldest pieces of code in Grist. And biggest."

**Optimized for deployment simplicity over scale-out**: While Grist supports multi-server deployments, the default is single-process. Each document is pinned to one Doc Worker, making SQLite's single-writer limitation acceptable.

**Multiple sandbox strategies**: Python formulas run in a subprocess that can be Docker, gVisor, macOS sandbox, or Pyodide/WASM (via Deno). This lets Grist run everywhere from a single Docker container to a hardened multi-tenant deployment.

## Comparison Notes

Unlike **Airtable**, Grist is open-source and self-hostable with Python formulas. Airtable has a more polished UI and better mobile support; Grist has more powerful computation and access control.

Unlike **Google Sheets/Excel**, Grist enforces column types (database-style) rather than allowing any type in any cell. The trade-off is less flexibility for individuals, more reliability for teams.

Unlike **Notion databases**, Grist handles much larger datasets (10K+ rows in a grid view) and provides proper chart, calendar, and card widgets. Notion is document-first; Grist is data-first.

Unlike **NocoDB/Baserow** (other open-source Airtable alternatives), Grist's Python formulas and dependency graph are more sophisticated. The self-contained SQLite file format makes Grist documents genuinely portable in a way API-centric alternatives aren't.

The most technically distinctive comparison: most spreadsheet engines use topological sort for formula evaluation. Grist's OrderError-based cooperative scheduling is more flexible — it handles dynamic dependencies and lazy evaluation that a static sort cannot express.

---
*Sources: [[raw/grist-core]]*
*Last updated: 2026-07-29*
