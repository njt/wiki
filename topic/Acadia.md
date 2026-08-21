# Acadia

Evan Czaplicki's public-alpha answer to a decade of database discomfort: a language that brings Elm's custom types, friendly errors, and functional pipelines to SQL. The thesis is that the object-relational impedance mismatch is self-inflicted — stop mapping objects to relations, and make the language's types *be* the schema, compiling `map`/`filter`/`select` to SQL and verifying migrations at compile time.

---

## What It Is

Acadia is a database programming language with two faces. On the schema side, a table is a typed value carrying everything the database needs to know:

```elm
foods : Table Security.Unrestricted Food
foods := Table.table
  { primary = .id
  , security = Security.unrestricted
  , indexes = []
  , constraints = []
  }
```

The primary key, a row-level security policy, indexes, and constraints live in the type — not scattered across migration files. On the query side, you write ordinary functional pipelines that compile to SQL at compile time:

```elm
getFood : Cookies -> FoodID -> Transaction String
getFood _ id =
  access foods Security.Unrestricted
    |> filter (\f -> f.id == id)
    |> map .name
    |> select
```

which becomes `SELECT f.name FROM "Foods.foods" AS f WHERE f.id = $1`. Acadia prints the SQL it generates, so you can evaluate its quality yourself.

## Key Quotes

> "I have been trying to bring the benefits of 'languages like Elm' to SQL."

The whole project in a sentence. Elm's gift was taking a hard problem — frontend state — and letting the compiler do the scary work; Acadia aims the same trick at the database.

> "Why do I have to convert my precise and expressive types into some weird binary layout by hand? Or to JSON? Or some combination of nullable columns?"

The sharpest diagnosis in the essay. It's [[Parse Don't Validate]] applied to the database boundary: converting a custom type into a storage format is a lossy, error-prone boundary that most stacks accept as normal.

> "When I go into a live database, my body goes on high alert. What if the commands I am running are slightly off? I feel like I am moments away from disaster."

The emotional core of "verified migrations." Czaplicki is naming a fear most backend programmers carry and never articulate: migrations are the one place a type error becomes a data-loss event. His fix is to let the compiler verify them beforehand, since it already knows the column types on both sides.

> "I know the data is in there, but it is not guaranteed to be in there."

The phrase that stalled his 2017 Elm server-side-rendering experiments. SQL's type system and Elm's type system disagree about whether a row exists, so no amount of client-side type safety could guarantee the data was real. End-to-end types — sharing types between client, server, and database — is the resolution.

> "The types in our database were not quite what we wanted, and at every level built on top of these types, we were doing a bunch of error-prone work to pointlessly convert the data between different formats."

The insight harvested from Elm companies in 2019: the backend, not the frontend, was the bottleneck — and the root cause was the type mismatch at the database layer, not any single framework's awkwardness.

> "I thought it would look like Elm, with map and filter..." — Tereza

The one-line design review that killed the "SQL syntax with a modern type system" prototype, turning Acadia from a clunky stored-procedure compiler into a functional language whose queries read like list processing.

> "How does it avoid the issues you see with Object-Relational Mappings (ORMs)? (No objects!)"

The entire ORM critique compressed into a parenthetical. The problems [[The Vietnam of Computer Science]] catalogued — partial objects, entity identity, the N+1 query — are all downstream of forcing objects onto relations. Acadia simply never introduces objects: queries are functions over tables.

## Key Themes

- **#tool** Acadia — an Elm-style language for database programming, public alpha August 2026
- **#concept** End-to-end types — one set of types shared across client, server, and database; a column change becomes a compile error
- **#concept** Verified migrations — the compiler checks schema changes against known column types before anything touches a live database
- **#pattern** Functional query pipelines — `filter`/`map`/`select` compiled to SQL at compile time, with `:=` let-bindings composing multi-step transactions that commit only if every step succeeds
- **#person** Evan Czaplicki — creator of Elm, now applying its design philosophy to databases

## Critical Assessment

**This is the most coherent entry yet in the "functional-relational mapping" niche.** The [[Object-Relational Impedance Mismatch]] article named functional-relational mapping as the elegant-but-niche third path, with Slick and LINQ as the closest mainstream approximations. Acadia is the purest realization of that path, and it is no accident it comes from the Elm ecosystem — Elm spent a decade teaching its community to trust the compiler over the runtime.

**"No objects!" is doing more work than it looks.** ORMs fail in six specific ways, and every one of them is a consequence of the "O," not the "M." By compiling queries as functions over typed tables, Acadia sidesteps the whole class structurally rather than papering over it. It is the strongest available argument for Neward's option 5 — "integration of relational concepts into languages" — made concrete.

**The verified-migrations goal is the real innovation, not the query syntax.** [[Malloy]] already proved you can compile a real language to SQL for analytics; where Malloy stops at the query layer, Acadia reaches down into schema changes. Making migrations compiler-verified — rather than a runtime operation you rehearse in staging and still fear in production — is a genuinely new claim, and the part most worth watching.

**The honest caveats are on the table.** It is a public alpha: window functions and custom aggregates didn't make the MVP, it runs on SQLite under the hood (with a drop-down-to-SQL escape hatch), and only Elm and Haskell are integrated so far. And the whole thing is a bet on an audience that already self-selects for type safety. Whether Acadia escapes that niche is the open question — but Czaplicki's near-decade timeline (2017 SSR experiments → 2026 alpha) is itself evidence that he treated "may not be possible" as a real possibility, not a marketing line.

---

## Related

- [[Object-Relational Impedance Mismatch]] — Acadia is the "functional-relational mapping" strategy this article names as elegant-but-niche, made concrete
- [[The Vietnam of Computer Science]] — Acadia is option 5 (relational concepts integrated into the language) realized in Elm-land; "No objects!" answers all six problems by never introducing objects
- [[Malloy]] — the closest cousin: another real language compiled to SQL, but aimed at the analytics layer where Acadia targets the application tier
- [[Parse Don't Validate]] — end-to-end types are the database-boundary version of King's thesis: parse once into a precise type instead of converting to JSON and nullable columns at every layer
- [[Constraint Decay]] — databases are the primary failure driver for coding agents; Acadia's premise is that the fix is one source of truth for types, not more translation layers
- [[Text-to-SQL in the Real World]] — the LLM path to the same problem; Acadia is the deterministic-language alternative

---
*Sources: [[raw/rethinking-database-programming]], [[summary/rethinking-database-programming]]*
*Last updated: 2026-08-21*
