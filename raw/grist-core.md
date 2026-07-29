---
url: https://github.com/gristlabs/grist-core
title: Grist
author: Grist Labs Inc.
date_fetched: 2026-07-29
date_published: 2014
---

# Grist — Architectural Analysis

## Project Overview

Grist is a modern relational spreadsheet: a hybrid that combines the flexibility of a spreadsheet with the robustness of a database. It's an open-source (Apache 2.0) project by Grist Labs (NYC), with significant contributions from the French government (ANCT Données et Territoires, DINUM). The `grist-core` repository contains the community edition server.

The project is a large TypeScript + Python codebase:
- ~233K lines of TypeScript (app/ directory: server, client, common)
- ~45K lines of Python (sandbox/ directory: data engine, formula runtime)
- ~490K lines in yarn.lock (dependencies)
- Additional C++, shell, Docker, and configuration files

The codebase is structured as:
- `app/server/` — Node.js Express server, document management, SQLite storage
- `app/client/` — Browser-side SPA: grid views, dashboards, widgets
- `app/common/` — Shared code between server and client
- `app/gen-server/` — Home database management (Postgres/SQLite)
- `app/plugin/` — Plugin API types and interfaces
- `sandbox/grist/` — Python data engine: formula evaluation, dependency tracking
- `sandbox/grist/functions/` — Built-in spreadsheet functions (802 lines of info functions alone)
- `stubs/` — Stub implementations for the community edition
- `buildtools/` — Build pipeline, Webpack config, deployment scripts
- `test/` — Mocha-based integration and browser tests

## Architecture

### Deployment Topology

Grist can run as a single process or as a horizontally-scaled deployment:

- **Home Servers**: Handle user-facing requests (listing docs, sharing, API). Single Node process, communicates with HomeDB (Postgres), Redis, and Doc Workers. Never opens documents or starts subprocesses.
- **Doc Workers**: Handle in-document interactions. Each open document is assigned to exactly one Doc Worker. Doc Workers bring SQLite files from S3 to local disk, spawn sandboxed Python interpreters, and communicate with browsers via WebSocket.
- **ALB**: Application Load Balancer handles SSL and routes HTTP requests (to Home Servers) and WebSocket connections (directly to Doc Workers).
- **Storage**: HomeDB (Postgres) for users/orgs/permissions, S3 for document files, Redis for Doc Worker assignment tracking.

The key insight: Home Servers and Doc Workers run the **same code** (`app/server/lib/FlexServer.ts`). Configuration and environment variables determine which role the process plays. This is a clean monorepo-with-dual-deployment pattern.

### Document Model

A Grist document is a `.grist` file — a SQLite database with a specific schema:

- `_grist_*` tables: managed by the Python data engine. These are metadata tables describing the document structure: `_grist_Tables`, `_grist_Tables_column`, `_grist_Views`, `_grist_Views_section`, `_grist_ACLRules`, `_grist_Attachments`, etc.
- `_gristsys_*` tables: managed by Node.js. These are system tables: `_gristsys_Action` (action log), `_gristsys_ActionHistory` (undo), `_gristsys_FileInfo` (attachments), `_gristsys_PluginData`.
- User tables: named after the user's own data.

The SQLite file is the unit of portability — everything is self-contained. Attachments can optionally be stored externally (S3-compatible) to keep `.grist` files small.

### Core Request Flow

1. User types into a cell in the browser
2. Client creates a **User Action** (e.g., `UpdateRecord`, `AddRecord`, `RenameTable`)
3. Sent via WebSocket to the Doc Worker (Node.js)
4. Node forwards to the Python data engine via RPC over stdin/stdout pipes
5. Python engine processes the User Action, evaluates affected formulas, produces **Doc Actions** (lower-level: `AddRecord`, `UpdateRecord`, `AddColumn`, etc.)
6. Node translates Doc Actions to SQL and updates the local SQLite file
7. Node broadcasts Doc Actions to all connected browsers via WebSocket
8. All clients update their in-memory data model

### Key Components (with file references)

**Server-side (`app/server/lib/`)**:
- `FlexServer.ts` — Express endpoint setup, initializes all components. The "Flex" means it can be Home Server, Doc Worker, or both.
- `ActiveDoc.ts` (~2,000+ lines) — The central dispatcher for an open document. Connects NSandbox, DocStorage, GranularAccess, DocClients. Shuttles user actions and doc actions between them. Methods include `applyUserActions`, `fetchTable`, `applyProposal`, `getFormulaTimingInfo`.
- `GranularAccess.ts` — Granular access control. Checks user actions before AND after the data engine processes them. Filters what each client sees based on permissions. Handles the complex task of access rules expressed as formulas.
- `NSandbox.ts` — Manages the Python subprocess. Spawns a child process with pipes for RPC-like communication. Supports multiple sandbox flavors (Docker, gVisor, macOS sandbox, unsandboxed).
- `DocStorage.ts` — SQLite operations. Translates every Doc Action into SQL. Manages the `_gristsys_*` system tables. Handles action logging, attachments, and snapshots.
- `HostedStorageManager.ts` — S3 sync. Fetches `.grist` files from S3 when docs are opened, syncs back on changes, creates periodic snapshots.

**Home DB (`app/gen-server/lib/homedb/`)**:
- `HomeDBManager.ts` — ORM-style manager for the HomeDB (Postgres or SQLite). Handles users, orgs, workspaces, documents, sharing, billing. Entity definitions in `app/gen-server/entity/`.

**Common (`app/common/`)**:
- `TableData.ts` — In-memory table data with Doc Action application logic
- `DocData.ts` — Set of TableData objects (all data for a document)
- `gutil.ts` — Utility functions
- `DocActions.ts` — Type definitions for all Doc Actions and User Actions
- `ActionBundle.ts` — Action packaging with envelope-based encryption for multi-instance sync

### Python Data Engine (`sandbox/grist/`)

- `engine.py` (~2,000+ lines) — The core Engine class. Owns the dependency graph, recompute map, and the update loop. Manages tables, schema, formula evaluation order via cooperative exception-based scheduling.
- `gencode.py` — Dynamic Python code generator. Converts the document schema (tables + columns + formulas) into a Python module at runtime. Each table becomes a class; each formula column becomes a method. Maintains a formula cache to avoid recompilation.
- `depend.py` — Dependency graph implementation. Nodes represent columns (table_id, col_id). Edges represent dependencies with Relations for row mapping. Inspired by the Ninja build system's graph.
- `relation.py` — Relations map rows between tables. `IdentityRelation` for same-table, `ReferenceRelation` for foreign keys, `ComposedRelation` for chains. Relations determine which dependent rows need recalculation when source rows change.
- `useractions.py` — Implementation of all User Actions. Complex actions (like `CreateViewSection`) are implemented here because Python makes it easier and enables single-step undo.
- `docmodel.py` — Convenience interface to document metadata.
- `schema.py` — Schema definitions for metadata tables.
- `acl.py` — Access control rule parsing and evaluation within the data engine.

## Key Techniques

### 1. Dynamic Code Generation (gencode.py)

The schema (ordered dict of tables, each with ordered dict of columns) is compiled into a Python module string and executed via `exec()`. This is the most distinctive architectural choice in Grist.

For each table, it generates a class like:
```python
class Students:
    def Name(rec, table):
        return rec.school.Name  # a Reference lookup formula
    def Grade(rec, table):
        return "A"  # a literal formula
```

Formula columns become methods. Non-formula columns become empty slots. Reference column types (e.g., `Ref:Schools`) generate typed accessor properties. Summary tables get special group-by column handling.

The generated code is cached per formula text. When a formula changes, only that column's method is regenerated — not the entire module. The `GenCode` class maintains a `_formula_cache` dictionary keyed by (table_id, col_id, formula_text).

### 2. Cooperative Formula Evaluation via OrderError

This is the most clever implementation detail. Instead of topologically sorting the dependency graph and evaluating in order (which requires knowing all dependencies upfront), Grist uses a dynamic, exception-driven approach:

1. When a cell needs recomputation, it's marked dirty in `engine.recompute_map`
2. The engine calls `_update_loop()` which iterates through dirty nodes
3. When evaluating a formula, if a dependency hasn't been computed yet, an `OrderError` is thrown
4. The OrderError carries the node and row_id of the unevaluated dependency
5. The engine catches OrderError, adds the requiring cell to the dependency's wait list, and moves on
6. When the dependency is finally computed, it triggers re-evaluation of waiting cells

This is essentially cooperative scheduling for formula evaluation. It handles dynamic dependencies (where the set of dependencies changes based on data values) naturally, which a static topological sort cannot. It also handles circular references gracefully — when a cell depends on itself, the engine detects the locked cell and raises `CircularRefError`.

The approach also distinguishes between formula columns and trigger formulas. Trigger formulas use `SingleRowsIdentityRelation` which refuses to propagate `ALL_ROWS` changes — they only recalculate for specific row changes, not full-column changes like renames.

### 3. Relation-Based Dependency Tracking

Dependencies are more than "column B depends on column A." The `Relation` class specifies WHICH rows in the dependent column need recalculation when specific rows change in the source column.

- `IdentityRelation` — same table, row N in column B depends on row N in column A (direct field reference)
- `ReferenceRelation` — foreign key: row N in table B depends on the referenced row in table A
- `ComposedRelation` — chain of relations (e.g., `rec.school.address.zip` composes ReferenceRelation(Person→School) + ReferenceRelation(School→Address))
- `SingleRowsIdentityRelation` — identity but refuses ALL_ROWS propagation, used for trigger formulas

The edge set in the dependency graph tracks which relations have been observed. When a new relation between two nodes is discovered during formula evaluation, it's added to the graph dynamically.

### 4. Two-Level Action System

User Actions (high-level, what the user intended) → Doc Actions (low-level, what actually changes)

User Actions are defined in `useractions.py` with the `@useraction` decorator. Examples:
- `UpdateRecord(table_id, row_id, col_values)` — user edits a cell
- `AddRecord(table_id, row_id, col_values)` — user adds a row
- `CreateViewSection(table_id, parent_id, type, ...)` — user adds a widget

Doc Actions are the atomic operations:
- Data: `AddRecord`, `BulkAddRecord`, `RemoveRecord`, `UpdateRecord`, `BulkUpdateRecord`
- Schema: `AddColumn`, `RemoveColumn`, `RenameColumn`, `ModifyColumn`, `AddTable`, `RemoveTable`, `RenameTable`

The Python engine translates User Actions into Doc Actions, evaluating formulas along the way. The Node server translates Doc Actions into SQL. Both Python and Node know about Doc Actions.

This is like a compiler pipeline: User Actions are the source language, Doc Actions are the IR. The separation means:
- Undo is straightforward (inverse Doc Actions)
- The access control layer can filter Doc Actions before clients see them
- Different clients can receive different subsets of Doc Actions based on their permissions

### 5. SQLite as a Document Format

The `.grist` file is a valid SQLite database. Any SQLite tool can read it. The schema is:

- **Storage Version** (PRAGMA user_version): Tracks changes to how data is stored on disk and changes to `_gristsys_*` tables
- **Schema Version**: Tracks changes to data-engine metadata (`_grist_*` tables)

These two versions enable migrations. The `DocStorage.docStorageSchema` has a `create()` method and a `migrations` array that are applied sequentially.

The SQLite file stores everything — attachments as blobs in `_gristsys_Files`, action history in `_gristsys_Action` and `_gristsys_ActionHistory`, undo information in `_gristsys_Action_step`. Attachments can optionally be stored externally (S3) via `HostedStorageManager`.

### 6. Virtual Scrolling via DOM Reuse (koDomScrolly)

For rendering tables with 10K+ rows, the client uses `koDomScrolly.js`. Instead of creating DOM nodes for every row (which would bring the browser to its knees), it:
- Renders only the visible rows
- Reuses DOM nodes — as the user scrolls, the same `<div>` elements are updated with new row data and repositioned
- Creates `BaseRowModel` observables only for visible rows
- Uses the scroll position to compute which rows should be visible and maps DOM elements to those positions

### 7. Granular Access Control

Access rules are formulas that evaluate to boolean. They can reference:
- Cell values in any table
- User attributes (email, name, role)
- Special `user` object for attribute-based conditions

The `GranularAccess.ts` module checks actions twice:
1. **Before** the data engine processes them: "Is this user allowed to make this change?"
2. **After** the data engine produces Doc Actions: "Should this client see the resulting changes?"

This two-pass approach handles the case where a user modifies a cell they can see, which triggers a formula recalculation in a column they can't see. The intermediate results are filtered before broadcast.

### 8. Sandbox Subprocess Architecture

The Python data engine runs in a separate process for isolation (user formulas are arbitrary Python code). Communication is via pipes with a custom RPC protocol. The `NSandbox.ts` module supports multiple sandbox flavors:
- **Docker**: Python in a Docker container
- **gVisor**: Google's application kernel for stronger isolation
- **macOS sandbox**: Native macOS sandboxing
- **Pyodide/Deno**: WASM-based Python for environments where separate processes aren't available (Windows, or when Docker isn't practical)
- **Unsandboxed**: Direct subprocess (for development or trusted environments)

### 9. Envelope-Based Action Bundles

Actions are packaged into envelopes for multi-instance synchronization. Each envelope contains encrypted content addressed to a specific set of recipients. This enables:
- Multi-master document collaboration (not fully implemented in the open-source edition)
- Encrypted action sharing between instances
- Fine-grained access control at the action level

The `ActionBundle` type includes `envelopes` (sets of recipients), `stored` (persistent Doc Actions), `calc` (computed Doc Actions), and `info` (metadata). The `EncActionBundle` extends this with encryption.

## Design Decisions and Trade-offs

### Optimized for: Formula expressiveness and correctness
- Full Python syntax for formulas (not a restricted subset)
- Dependency tracking that handles dynamic dependencies
- Two-pass access control for correctness even with complex formulas

### Sacrificed: Memory efficiency
- Entire document is loaded into Python memory for formula evaluation
- Each browser tab maintains a full copy in JavaScript memory
- Exception: "on-demand" tables allow partial loading for large datasets

### Optimized for: Portability
- Single-file SQLite format means documents are self-contained
- Can be downloaded, backed up, moved between servers
- Any SQLite tool can read numeric/text data

### Sacrificed: Concurrent write performance
- Each document is assigned to exactly one Doc Worker
- Multiple users editing the same document go through the same worker
- SQLite is inherently single-writer (WAL mode helps but doesn't eliminate this)

### Optimized for: Deployment simplicity
- Single Docker container can run everything
- Same codebase for Home Server and Doc Worker
- Multiple sandbox options adapt to different environments

### Sacrificed: Clean separation of concerns in client code
- The client-side codebase has legacy from multiple UI eras: knockout.js, GrainJS (custom reactive framework), and newer components
- Components shuffled between `app/client/ui`, `app/client/components`, and `app/client/ui2018`
- The `GridView.js` is described in the docs as "one of the oldest pieces of code in Grist. And biggest."

### Optimized for: Extensibility
- Plugin system via `app/plugin/` with typed APIs for custom widgets, import sources, file parsers
- REST API for programmatic access
- Webhook system for integrations
- AI Formula Assistant for formula generation

## Comparison to Related Systems

**vs. Airtable**: Grist is open-source and self-hostable. Both are relational spreadsheet-database hybrids. Grist's formula system is Python-based and arguably more powerful (full Python, not a formula language). Airtable has better mobile support and a more polished UI. Grist's access control with formula-based rules is more flexible.

**vs. Google Sheets / Excel**: These are pure spreadsheets — every cell can have a different type. Grist enforces column types (like a database). Grist has proper relational features (References, Reference Lists, summary tables) that are awkward or impossible in traditional spreadsheets. Grist's Python formulas are more powerful but less accessible than spreadsheet formula languages.

**vs. Notion databases**: Notion databases are simpler, more document-oriented. Grist is more of a serious data tool with proper spreadsheet-like features (charts, dashboards, complex formulas). Grist handles much larger datasets.

**vs. NocoDB / Baserow**: These are also open-source Airtable alternatives. Grist distinguishes itself with Python formulas (vs. formula languages), the two-level action system, the dependency graph architecture, and the self-contained SQLite file format.

## Key Numbers

- ~233K lines TypeScript (client + server + common)
- ~45K lines Python (data engine + functions)
- ~490K lines yarn.lock
- Document size limit: configurable, typically millions of rows per table
- Max SQLite variables per statement: 500 (self-imposed limit, SQLite supports 999)
- Attachment expiry: 7 days after soft-delete
- Multiple document storage versions tracked via PRAGMA user_version
