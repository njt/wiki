# SQL Pagination — Offset vs Seek Method

Markus Winand's definitive field guide to the two fundamental SQL pagination strategies: the offset method (ubiquitous, degrades with depth) and the seek method (performant, constrains UX). The piece is part of his "Use The Index, Luke" series and distills a problem every web developer hits — "why is page 100 slow?" — into a single, index-aware insight: counting from the beginning is the wrong algorithm when you know where the last page ended.

---

## Key Quotes

> "The seek method avoids both problems because it uses the *values* of the previous page as a delimiter. That means it searches for the values that must *come behind* the last entry from the previous page."

The central architectural insight of the piece. Offset pagination answers "give me rows 11–20"; seek pagination answers "give me the next 10 rows after the one I last saw." The second question is index-friendly; the first forces a full scan from the start. This is the same distinction as array indexing vs. linked-list traversal — one is O(1), the other O(n).

> "Paging requires a deterministic sort order."

Winand's most important warning, bolded in the original. Without a deterministic `ORDER BY`, the database is free to return rows with identical sort keys in any order — and parallel query execution means it increasingly *will*. The fix is extending the `ORDER BY` clause with additional columns until every row has a unique position. This is not a theoretical concern; it's a production bug waiting to happen.

> "The `where` clause consists of two parts. The first part considers the `SALE_DATE` only and uses a less than or equal to (`<=`) condition — it selects more rows as needed. This part of the `where` clause is simple enough so that all databases can use it to access the index."

The approximated seek method for databases without row-value support. Select slightly too many rows with an index-friendly predicate, then filter the excess. This is the same pattern as cursor-based pagination APIs use: `WHERE created_at <= ?` fetches the day's worth, then application code (or a second filter predicate) drops the already-seen entries. The cleverness is in making the first predicate simple enough that every query planner can turn it into an index range scan.

> "These two functions — skipping pages and browsing backwards — are not needed when using an infinite scrolling mechanism."

Winand's pragmatic closer. If your UI uses infinite scroll, the seek method's limitations (no arbitrary page jumps, no reverse browsing) don't matter. The UX constraint eliminates the engineering problem. This is a rare instance of frontend architecture simplifying backend architecture rather than complicating it.

## Key Themes

- **#pattern — Seek method (keyset pagination)**: The performant alternative to offset pagination. Uses the last-seen row's values as a lower bound, enabling index range scans that skip directly to the right position. Requires a deterministic sort order and careful `WHERE` clause construction. The database work is O(log n) rather than O(n).
- **#pattern — Row values syntax**: The SQL standard's multi-column comparison syntax (`(a, b) < (x, y)`) that makes multi-column seek predicates elegant. Only properly supported in Db2 LUW and PostgreSQL; Oracle, SQL Server, and MySQL require decomposed comparisons. A reminder that "standard SQL" and "what your database supports" are different things.
- **#concept — Deterministic sort order**: Pagination's hidden prerequisite. Without it, the same query can return rows in different orders on successive executions, breaking page continuity. The fix is extending `ORDER BY` with additional columns (typically the primary key) until row positions are unique — and ensuring the index matches.
- **#tool — Index access predicates vs. filter predicates**: The execution plan distinction that makes or breaks seek pagination performance. An *access* predicate narrows the index scan range (fast); a *filter* predicate discards rows after reading them (slower but still okay for the few excess rows in the approximated seek method).
- **#comparison — Offset vs. seek**: The offset method wins on simplicity and arbitrary-page access; the seek method wins on performance stability and insertion-proof results. The choice depends on UX requirements: numbered pages → offset; infinite scroll → seek.

## Critical Analysis

**Winand's genius is explaining *why*, not just *how*.** The piece doesn't just give you the SQL — it walks through the execution plan, shows the index access path diagram, and explains what the database engine is doing at each step. This is what makes "Use The Index, Luke" one of the most-cited SQL performance resources: it teaches diagnostic reasoning, not recipes.

**The database compatibility matrix is the piece's hidden value.** Knowing that row values work in PostgreSQL and Db2 but not Oracle/SQL Server/MySQL is practical knowledge that saves hours of debugging. Winand provides workarounds for every database, which is the difference between a theoretical pattern and one you can actually ship.

**The "deterministic sort order" warning is more important now than when it was written.** Modern databases increasingly use parallel query execution, which makes non-deterministic ordering more common. A pagination query that worked reliably for years can break when the query planner decides to parallelize. Winand's advice to add the primary key to `ORDER BY` is not defensive coding — it's what prevents a production incident.

**The piece handles the row-values limitation honestly.** Rather than hand-waving "most databases support this," Winand explicitly lists which databases support row values as access predicates (Db2 LUW, PostgreSQL), which support them but can't use them for index access (MySQL), and which don't support them at all (SQL Server 2017). This is the kind of specificity that makes the article actionable.

**A limitation: the analysis predates some modern pagination patterns.** Cursor-based pagination in GraphQL and REST APIs is essentially the seek method with a different name. The article doesn't connect the SQL pattern to API design, which is the bridge most web developers need. The "infinite scrolling" mention at the end gestures at this but doesn't develop it.

**The performance chart (Figure 7.4) would be more valuable with the raw data.** Winand says the difference is "clearly visible from about page 20 onwards" but doesn't give numbers. For a piece this rigorous about execution plans, the absence of benchmark data on the seek method's advantage is a gap — one that the reader has to fill by trying it themselves.

**Connection to [[Dapper Performance Trap]]:** Both pieces are about the gap between "the query looks correct" and "the query uses the index." Winand's offset method produces correct results but loses the index; Griffin's `nvarchar` default produces correct results but loses the index. The same diagnostic pattern: look at the execution plan, find where the index scan becomes a scan (or a convert-implicit defeats the seek), and fix the predicate so the database can use what it has.

**Connection to [[Text-to-SQL in the Real World]]:** Winand's article represents the kind of deep, database-specific knowledge that Stonebraker and Chen's BEAVER benchmark shows LLMs cannot replicate. An LLM might generate `OFFSET 10 FETCH NEXT 10 ROWS ONLY` — it's in every training example. It will not realize the query needs a deterministic `ORDER BY` clause extended with the primary key, nor will it decompose a multi-column seek predicate for Oracle. This is the kind of expertise that separates generated SQL from production SQL.

**Connection to [[Databases and Data]]:** The hub page identifies "agent-aware database patterns" as a gap. Winand's piece fills part of that gap: agents that generate paginated queries need to default to the seek method, not the offset method, and must extend `ORDER BY` clauses to be deterministic. This is a concrete example of the kind of pattern that belongs in an agent-oriented database playbook.

**Connection to [[Software Engineering Craft]]:** The "deterministic sort order" requirement is a software engineering fundamental disguising itself as a database detail. It's the same class of bug as non-deterministic iteration over hash maps — the code works until it doesn't, and when it breaks, the symptoms (missing rows, duplicate rows across pages) are baffling. Winand's insistence on getting this right is the database equivalent of "don't rely on hash ordering."

---

*Sources: [[raw/fetch-next-page]], [[summary/fetch-next-page]]*
*Last updated: 2026-08-08*
