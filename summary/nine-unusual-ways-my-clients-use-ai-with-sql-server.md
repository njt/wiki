---
url: https://blog.sqlauthority.com/2026/08/06/nine-unusual-ways-my-clients-use-ai-with-sql-server/
title: "Nine Unusual Ways My Clients Use AI with SQL Server"
author: Pinal Dave
site: SQL Authority (blog.sqlauthority.com)
date_published: 2026-08-06
---

Pinal Dave presents nine real consulting engagements where AI was used not to generate SQL queries, but to read, classify, and extract from existing database artifacts at volume — with human verification as the non-negotiable final step. Every case follows the same shape: the machine reads at scale and proposes; a human verifies and decides. The common thread is that none of these are hard problems — they are large, tedious, low-judgment reading tasks that a competent person *could* do given three months and no interruptions, a resource that has never existed.

The nine workflows:

1. **Business rule extraction from legacy PL/SQL** — 800 Oracle packages, 400K lines, 19 years old. AI extracted an inventory of plain-English business rules with line-number citations, surfacing ~60 obsolete or wrong rules the business didn't know about. Key detail: feeding table DDL alongside code improved reasoning quality more than prompt wording.

2. **Reverse engineering a vendor's undocumented schema** — Five inputs assembled per table (DDL, FK graph, value distributions, vendor's own readable views, Extended Events capture of one screen) produced a 60-page data dictionary for a database the vendor contractually refused to document.

3. **Decoding a German ERP from the early 2000s** — Two-pass approach: expand abbreviated German column names, then derive domain meaning from context. Compound-noun ambiguity was the failure mode; test data caught what confidence couldn't.

4. **M&A due diligence schema mapping** — Column-by-column mapping across two company schemas with three required outputs: target column, confidence level, *and specific reason*. The reason column was what let humans triage 900 columns in a week and catch that `CUSTOMER_STATUS` meant Active in one company and Archived in the other.

5. **Compliance documents turned into SQL checks** — A 300-page audit document became a test suite of queries that return zero rows when compliant. Section-number traceability in query comments made it credible to auditors. About a third of requirements weren't testable in SQL at all.

6. **Column-level data lineage across 40 procedures** — Traced a finance number through views, stored procedures, and SSIS packages. AI read inside dynamic SQL (which `sys.sql_expression_dependencies` can't see) and found the number was being rounded twice, causing a one-cent reconciliation argument every month for years.

7. **SQL Agent job graveyard audit** — 240 accumulated jobs across 15 years. AI classified each into four buckets (needed, dead, unclear, broken-but-silent). Found jobs writing to dropped tables, copying to decommissioned shares, and one emailing a report to ex-employees' still-resolving mailboxes. Rule: disable with a note, delete after 90 quiet days.

8. **Query plan regression diagnosis** — Diffed pre- and post-upgrade execution plans. Correctly identified cardinality estimator changes causing join-order shifts on correlated columns (city + postal code). Wrongly recommended a database-wide legacy setting that would have pessimized hundreds of other queries. "It knew the mechanism. It did not know the blast radius."

9. **Dialect drift verification after migration** — Instead of asking AI to convert queries, asked it to list every semantic difference between source and target platforms that could affect the query. 180 test cases; 19 failed on first run. NULL ordering, collation, empty-string semantics, integer division, date arithmetic — every category produces wrong answers with no error message.

The article closes by noting that none of these clients bought a product. They got a workflow, a mandatory verification step, and a written note about failure modes. Their own people run it all now.
