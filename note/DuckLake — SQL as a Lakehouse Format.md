# DuckLake: SQL as a Lakehouse Format

DuckDB's May 2025 announcement of DuckLake, an open lakehouse table format that moves all table metadata into a plain SQL database (catalog and per-table state alike) while data files stay as immutable Parquet on blob storage. The thesis: once the lakehouse stack already contains a database for catalog consistency, encoding metadata as files on blob storage is an unnecessary compromise — "It's all just SQL."

---

The post's historical framing is the strongest part. Storage/compute separation (BigQuery, Snowflake) plus open formats (Parquet on S3) produced the data lake; the need to *change* data produced Iceberg and Delta Lake, which encode snapshots, schemas and manifests as JSON/Avro files specifically to avoid needing a database. But the catalog layer everyone actually deployed reintroduced a database anyway, to get an atomic pointer swap on blob stores with mercurial consistency. DuckLake's move is simply to notice this and finish the job: define the entire format as relational tables and SQL transactions. One `INSERT` becomes one ACID transaction writing the Parquet path, statistics, a snapshot row, and a change log — the same shape BigQuery (Spanner) and Snowflake (FoundationDB) use internally, just with open formats at the bottom.

The claimed wins follow directly from the design. Reads plan with a single catalog query instead of a cascade of HTTP round trips through manifest files. Small changes can be inlined into the metadata store rather than writing tiny Parquet files, sidestepping the small-files/compaction problem that entire companies exist to manage. Concurrency reduces to SQL transaction conflict resolution ("even the venerable Postgres can run thousands of transactions per second"), and snapshots become a few metadata rows, allowing millions of them. The `ducklake` extension turns DuckDB into a client-server warehouse participant: many DuckDB instances attaching to one catalog, with MIT licensing under the DuckDB Foundation.

**Key quotes:**

> Both formats were specifically designed to *not* require a database. Their designers went to great lengths to encode all information needed to efficiently read and update tables into files on the blob store... Almost as if a specialized data management system of sorts would be a good idea.

The sarcasm is the argument. The lakehouse format ecosystem exists to work around the absence of a database — manifest batching, small-file compaction, cleanup procedures "not very well understood nor supported by open-source implementations" — and DuckLake's provocation is that the absence was never real, since the catalog database was already there.

> Storing *data* files in open formats on blob storage is a great idea for scalability and to prevent lock-in. Managing metadata is a complex and interconnected data management task best left to a database management system.

A crisp two-truths split that draws the line exactly where Iceberg and Delta chose not to. Data stays open and dumb; metadata goes back to the discipline that spent fifty years learning to manage interconnected state.

> There are no Avro or JSON files. There is no additional catalog server or additional API to integrate with. **It's all just SQL.** We all know SQL.

Simplicity as the selling point, aimed at operators: any ACID SQL database with primary keys works, migration between catalog backends requires no data movement, and most organizations already run something suitable.

> One could run a thousand compute nodes running appends to a table at a one-second interval and it would work fine.

The concurrency claim is the boldest. Iceberg's commit path spends long stretches in a conflict-prone critical span of HTTP requests; DuckLake collapses commits to one transaction and leans on the database's native conflict handling.

**Critical take.** This is a genuinely good piece of format design rhetoric — locate the hidden dependency your competitors pretend not to have, then build on it deliberately. The honest caveats are in the fine print. Metadata-in-a-database reintroduces a stateful, operationally-loaded component at the center of the lake, and "any halfway competent SQL database" hides the question of what happens when thousands of compute nodes hammer one Postgres — the post's answer (migrate to Spanner) concedes the ceiling. The optional inlining of small changes *into the catalog database* is also a quiet reversal of the format's own principle: that's data, not metadata, living in the metadata store, and it works only because DuckLake accepts that the database tier must be provisioned for more than tiny metadata transactions. The Iceberg-compatible-file claim is a smart hedge — migration is metadata-only — but compatibility of files is not compatibility of semantics, so the cross-engine story is narrower than "use any engine." Finally, "experimental" is stated plainly, which is to the post's credit.

**Key themes:** #concept #tool #pattern

**Relations.** This strengthens [[Iceberg Table Cleanup]], which documents exactly the compaction and cleanup burden DuckLake claims to design away — the two pages are the diagnosis and the bet against it. It nuances [[Icebug Format]], another take on the lakehouse-format design space, by showing the most radical simplification available: delete the file-based metadata layer entirely. And it complicates [[DuckDB ADBC Extension]]'s picture of DuckDB as an embeddable analytical engine — with DuckLake, DuckDB becomes a stateless compute node against a shared catalog, a architectural jump the ADBC story doesn't cover.

---
*Sources: [[raw/ducklake-01]], [[summary/ducklake-01]]*
*Last updated: 2026-11-03*
