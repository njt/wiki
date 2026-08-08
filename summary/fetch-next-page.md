---
url: https://use-the-index-luke.com/sql/partial-results/fetch-next-page
title: Fetching the Next Page — SQL Pagination Patterns
author: Markus Winand
site: Use The Index, Luke
date_fetched: 2026-08-08
---

Markus Winand compares the two fundamental approaches to SQL pagination: the widely-used **offset method** and the more performant **seek method** (also known as keyset pagination).

The offset method uses `OFFSET` (or equivalent constructs like `ROWNUM` in older Oracle) to skip a fixed number of rows before fetching the page. It's simple and supported across all major databases — Db2, MySQL, Oracle, PostgreSQL, and SQL Server — but has two structural disadvantages: pages drift when new rows are inserted, and response time degrades linearly as you page further back, because the database must count every row from the beginning.

The seek method avoids both problems by using the *values* of the previous page's last row as a delimiter. Instead of `OFFSET 10`, you write `WHERE sale_date < ?` (for descending order), letting the database use the index to skip directly to the right position. The key requirement is a **deterministic sort order** — without it, the database may shuffle rows with identical sort keys, breaking pagination. The fix is extending the `ORDER BY` clause with additional columns (typically the primary key) until every row has a unique position.

The article covers the "row values" syntax (`WHERE (sale_date, sale_id) < (?, ?)`) which elegantly expresses multi-column seek predicates, but notes it's only properly supported in Db2 LUW and PostgreSQL. For databases without row-value support (Oracle, SQL Server, MySQL), Winand provides an approximated seek method using decomposed comparisons (`sale_date <= ? AND NOT (sale_date = ? AND sale_id >= ?)`) that still enables index access on the leading column.

Performance measurements (Figure 7.4) show the seek method maintaining constant response time regardless of page depth, while the offset method degrades visibly from about page 20 onwards. The trade-off: seek method requires careful `WHERE` clause construction, cannot skip to arbitrary pages, and needs reversed comparisons to change browsing direction — limitations that infinite-scrolling UIs sidestep entirely.
