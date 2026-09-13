---
url: https://www.vldb.org/pvldb/vol19/p4494-schmidt.pdf
date_fetched: 2026-09-13
---

    QueryBrew: System-Agnostic SQL-to-SQL Query Optimization
                                Tobias Schmidt∗                                                                            Maximilian Reif∗
                      Technische Universität München                                                          Technische Universität München
                         tobias.schmidt@in.tum.de                                                                     reif@in.tum.de

                                   Altan Birler∗                                                                        Thomas Neumann
                      Technische Universität München                                                          Technische Universität München
                            altan.birler@tum.de                                                                    neumann@in.tum.de
ABSTRACT                                                                                               eryBrew
                                                                                                       Optimizer as a Service (QOaaS)
                                                                                                                                          Target System: PostgreSQL | ClickHouse |
                                                                                                                                          DuckDB | SQL Server | …
Efficient SQL execution heavily depends on the query optimizer’s
ability to find efficient query plans. Existing query optimizers differ                                       ery                                    Any SQL                        ery
                                                                                                 SQL                                    SQL
widely in their capabilities, leading to significant differences in exe-                                     Optimizer                             Database System                   Result
cution complexity across many queries. In this demonstration, we
                                                                                                        Simpliﬁcation                         System Speciﬁc Optimization
present QueryBrew, a system-agnostic query optimization service                                         General Unnesting                     Physical Operator Selection
that decouples the optimizer from the database engine through                                           (Outer) Join Ordering
                                                                                                        SQL Dialect Translation
                                                                                                                                              Index Selection
                                                                                                                                              ery Execution
SQL-to-SQL rewriting. QueryBrew takes arbitrary SQL queries, re-
fines them using the state-of-the-art Umbra optimizer, and distills                                                         Schema and Table Statistics

the optimized execution plan back into an operator-oriented SQL
                                                                                              Figure 1: QueryBrew’s SQL-to-SQL optimization approach.
representation. This allows any target SQL database, such as Post-
                                                                                              Queries are optimized by a state-of-the-art optimizer (e.g.,
greSQL, DuckDB, ClickHouse, or SQL Server, to inherit advanced
                                                                                              Umbra) and transformed back to SQL. The optimized query
optimization techniques, such as general unnesting, simplification,
                                                                                              benefits from simplification rules, general unnesting, and ad-
and adaptive join ordering, without modification. The QueryBrew
                                                                                              vanced join ordering [5], and runs in existing SQL databases.
UI lets users explore and analyze the changes made by Umbra’s
query optimizer and how they affect execution in the target system.
SQL-to-SQL optimization can improve runtimes by more than one                                     The query optimizer is the component with the greatest potential
order of magnitude for correlated and many-join queries.                                      to make or break queries. Its decisions not only affect runtime
                                                                                              constants but can change a query’s runtime complexity entirely. It
PVLDB Reference Format:                                                                       is a high-risk and high-reward component. Changes to the optimizer
Tobias Schmidt, Maximilian Reif, Altan Birler, and Thomas Neumann.                            can yield immense benefits, but they are also likely to break some
QueryBrew: System-Agnostic SQL-to-SQL Query Optimization. PVLDB,                              customers’ workloads in unexpected ways (e.g., by removing one
19(12): 4494 - 4497, 2026.                                                                    of two mistakes that cancelled each other out). Thus, big players
doi:10.14778/3827998.3828048                                                                  in the industry approach developments in the optimizer with a
                                                                                              risk-averse, calculated approach. Unfortunately, this means that
PVLDB Artifact Availability:
                                                                                              optimizers often lag behind the latest innovations [14].
The source code, data, and/or other artifacts have been made available at
                                                                                                  Decoupling the optimizer from the database engine would accel-
https://querybrew.db.cit.tum.de.
                                                                                              erate innovation and enable independent development. However,
                                                                                              optimizers are deeply intertwined with the database system, its sta-
1    INTRODUCTION                                                                             tistics, and its operators; hence, extracting them is non-trivial. Even
Databases have gotten significantly faster over the last decades.                             the many database solutions at Google, such as BigQuery, Span-
Vectorized execution in DuckDB and SQL Server and compiled                                    ner, F1, BigTable, Dremel, and Procella, share the SQL frontend
execution in PostgreSQL significantly speed up analytical queries.                            GoogleSQL but do not share a single optimizer [2].
High-bandwidth storage devices and massively parallel machines al-                                There have been attempts to design a common intermediate
low processing multi-terabyte datasets on a single node. Distributed                          representation for query plans such as Substrait [1]. CompoDB [7]
engines such as Snowflake, Databricks, BigQuery, and Redshift are                             exploits Substrait to build a modular data system with an exchange-
widely deployed in the industry and support petabyte-scale ana-                               able query optimizer and execution engine. They rely on DuckDB,
lytics. However, one particular component has lagged behind this                              DataFusion, Calcite, and Ibis for optimization. However, only a
pace of innovation: the query optimizer.                                                      few engines (DuckDB, DataFusion, and Acero) are supported, as
                                                                                              Substrait has not seen wide adoption due to the inherent difficulty
∗ Authors contributed equally to this research.
                                                                                              of unifying the semantics of existing systems with their various id-
This work is licensed under the Creative Commons BY-NC-ND 4.0 International
License. Visit https://creativecommons.org/licenses/by-nc-nd/4.0/ to view a copy of
                                                                                              iosyncrasies. Microsoft takes this idea one step further and proposes
this license. For any use beyond those covered by this license, obtain permission by          Query Optimizer as a Service (QOaaS). They make Fabric’s Unified
emailing info@vldb.org. Copyright is held by the owner/author(s). Publication rights          Query Optimizer [6] available to other engines, such as Spark, via
licensed to the VLDB Endowment.
Proceedings of the VLDB Endowment, Vol. 19, No. 12 ISSN 2150-8097.                            Substrait-to-Substrait optimization. The logical plan is given to the
doi:10.14778/3827998.3828048                                                                  optimizer, which produces an optimized, physical plan [15].




                                                                                       4494
 Input: Human-Wrien ery                       Optimization              Decorrelated ery Plan               Output: Operator-Oriented ery

 select category, count(*)                                                                 ↕                    with sort_8 as (…),
 from item i                                                                                                           group_by_7 as (…),
 where current_price >                                                                     Γ
                                                                                                                       join_inner_6 as (…),
    (select avg(current_price)                                                            ⨝
     from item                                                                                                         group_by_5 as (…),
     where category = i.category                                                                 Γ                     join_cross_4 as (…),
            or color = i.color)                                             Item
                                                                                                                       scan_3 as (…),
 group by category                                                                             +
 order by count;                                                                                                       magic_2 as (…),
                                                                                           Γ         Item              scan_1 as (…),
                                                                                                                select * from sort_8

Figure 2: QueryBrew in action: Correlated query on the TPC-DS item table. Umbra’s query optimizer decorrelates the query
using general unnesting. We transform the optimized query plan into an operator-oriented representation using one CTE per
operator that can canonically be executed by other database systems.


    We approach the decoupling of query optimizers in a practical                query plan is not a typical query tree but a directed acyclic graph
way as shown in Figure 1. Our optimization service, QueryBrew,                   (DAG) of relational operators, enabling more efficient execution.
takes SQL as input, optimizes the query internally, and brews SQL as                QueryBrew translates the optimized query plan back to SQL,
output. We found that rewriting queries in SQL can greatly improve               retaining the optimizations made by Umbra. It preserves the ex-
performance across a wide range of queries. Because most relational              ecution order of the operators from the optimized plan, allowing
databases use SQL as their standard interface, QueryBrew integrates              other systems to benefit from Umbra’s join optimizer. We achieve
easily with existing systems. In addition, the target system’s query             this using Common Table Expressions (CTEs) in the exported SQL.
optimizer can further apply system-specific optimizations and select             Every operator in the optimized plan is represented as an individual
the best physical operators, taking the system’s implementation                  CTE, and operators are connected by referencing the CTEs of their
details into account.                                                            inputs. Consider the following query:
    We are not the first to explore SQL-to-SQL optimization. Both
LLM-based [8] and human-centered [3] rewriting approaches have                     select i_category, count(distinct i_color)
been proposed; however, these surface-level techniques operate on                  from item group by i_category;
the query text or the abstract syntax tree. Since many sophisticated
optimizations operate on relational algebra, the capabilities of such            It counts the number of distinct colors for each category in the item
text-based techniques are inherently limited. OpenIVM [4] also                   table from the TPC-DS benchmark.
operates on relational algebra, but it generates SQL for incremental                 Umbra’s optimized query plan translated to SQL looks as follows:
view maintenance rather than for query optimization.
                                                                                   WITH scan_1 AS (
    QueryBrew demonstrates that complex, correlated queries can
                                                                                     SELECT i_category AS v5, i_color AS v6 FROM item
be optimized directly within SQL. By leveraging schema and sta-
                                                                                   ), groupby_2 AS (
tistics, it simplifies, unnests, and utilizes cost-based reordering via
                                                                                     SELECT v5 AS v3, v6 AS v4 FROM scan_1 s GROUP BY v5, v6
adaptive optimizations and enumeration of join plans to find the
                                                                                   ), groupby_3 AS (
best execution plan. Through a structured back-translation of the
                                                                                     SELECT v3 AS v1, count(v4) AS v2
optimized query plan to SQL, target systems can benefit from these
                                                                                     FROM groupby_2 s GROUP BY v3)
optimizations without reimplementing them. Additionally, an intu-
                                                                                   SELECT v1 AS i_category, v2 AS count FROM groupby_3;
itive interface lets users easily analyze SQL-to-SQL optimizations
and benchmark queries across different systems.                                  Every operator from the plan is given a unique name (e.g., scan_1,
                                                                                 groupby_2, groupby_3). While the scan reads the two required
                                                                                 columns from the item table, the two group by operators compute
2   SOLUTION OVERVIEW                                                            the distinct count. Here we can already observe one of the opti-
Approach. QueryBrew builds a refined SQL statement by passing                    mizations performed by Umbra: It splits the distinct count into two
an input query through Umbra’s [11] state-of-the-art optimizer                   group by operators (groupby_2 and groupby_3), one to compute
and distilling the resulting optimized plan back into SQL. Umbra                 all distinct combinations of category and color, and another one to
implements a wide range of (cost-based) optimization techniques,                 count the number of colors for each category. Thanks to Umbra’s
including operator simplification, general unnesting [10, 12], adap-             general unnesting, converting an operator is independent of the
tive join reordering [13], and common subtree elimination. These                 overall query structure: each operator simply references the CTEs
techniques exploit detailed statistics, data samples, type informa-              of its inputs. The translation to SQL is canonical for most operators;
tion, and functional dependencies available to the database to esti-             only a few non-standard operators (group-joins, mark-joins, and
mate cardinalities and simplify or eliminate operators. The resulting            magic sets) require special care.




                                                                          4495
Integration. The CTEs provide a structured, more optimized rep-                Table 1: Runtimes on the original correlated query from
resentation of the original queries, allowing the target systems to            Figure 2 versus Umbra’s optimized query plan on TPC-DS
exploit some of Umbra’s advanced optimization techniques without               scale factor 1 GB.
implementing them themselves. For instance, Umbra implements
general unnesting, which decorrelates arbitrary queries and re-                                           Original                 Optimized Query
moves dependent joins from the optimized plans. The CTEs also                                           time correct             time correct speedup
capture the join order, including the build and probe sides of the
                                                                                   PostgreSQL         40.8 s        Ë            2.1 s       Ë         19.9×
joins. To avoid a second join reordering by the target system, we
                                                                                   ClickHouse          4.4 s        é            0.2 s       Ë         21.8×
force it to follow the join order of the optimized query plan using
                                                                                   SQL Server1           –          Ë              –         Ë         2.84×
system-specific settings and hints where possible. SQL-to-SQL opti-
                                                                                   DuckDB             99 ms         Ë           12 ms        Ë         8.07×
mization also enables the target systems to further optimize queries
                                                                                   Umbra              18 ms         Ë           18 ms        Ë         1.00×
and adapt them to their specific execution and storage engines.
    Figure 2 depicts a query on the TPC-DS item table. While the
query (on the left) is easy to write and understand for humans or
LLMs, its naive execution leads to quadratic runtime due to the                   The goal of our demonstration is to show that query optimiza-
correlated subquery. Conceptually, the subquery is evaluated for               tion through SQL is feasible and that SQL-to-SQL optimizations
every tuple of the outer query, but the actual data allows for a               integrate well with existing database systems. The QueryBrew in-
more efficient evaluation in which the subquery is evaluated only              terface allows users to specify their own queries and compare the
once for each combination of its free variables. To achieve that,              original and optimized versions. The query plan view, in particular,
the Umbra query optimizer translates the query into relational                 gives insight into potential optimizations. It can be used to explore
algebra, automatically decorrelates it using its general unnesting             the differences between the original and optimized query plans,
implementation, and applies further optimizations. The resulting               such as more efficient join orders or the advantage of unnesting by
query plan is then translated back into SQL as an operator-oriented            eliminating dependent joins.
query, with one CTE per operator. Our implementation supports                     While the CTE-based representation gives a structured and sim-
the SQL-to-SQL translation of all Umbra plan operators into the                plified representation of the query, it can lead to performance degra-
SQL dialects of ClickHouse, DuckDB, PostgreSQL, and SQL Server.                dation in some systems, as potential optimizations are missed by the
    To demonstrate the advantage of this approach, we evaluate                 target system’s optimizer. In our experiments, we observe speedups
the human-written and the optimized operator-oriented query from               larger than 100× and only minor slowdowns in comparison (<5×)
Figure 2 in ClickHouse, DuckDB, PostgreSQL, and SQL Server:                    as Umbra’s optimizer improves the asymptotic complexity of the
Table 1 shows the execution times of the original and optimized                queries, whereas alternative execution strategies mostly result in a
queries on the four systems and Umbra. For systems that do not                 constant overhead. In such cases, users can run the original query
implement unnesting (ClickHouse and PostgreSQL), executing this                and avoid slowdowns caused by the CTE-based representation.
query takes considerable time, even though the item table has only
18,000 entries. Although DuckDB and SQL Server both unnest the                 3     DEMONSTRATION PROPOSAL
query, Umbra’s query plan further improves the execution times. To             In our demonstration, visitors can explore different optimizations
our surprise, ClickHouse (version 25.11) returns the wrong result              that Umbra applies to SQL queries. Original and optimized queries
for the original query; however, running the optimized version                 can be executed on four different database systems (ClickHouse,
produces the correct result. The simplified CTE representation does            DuckDB, PostgreSQL, and SQL Server). Our interface supports
not trigger the incorrect code path in ClickHouse.                             comparing runtimes and query plans and displays query results to
Statistics. The key to finding a good query plan is accurate sta-              validate the correctness of the optimizations. We preload four well-
tistics. Umbra relies on HyperLogLog sketches for distinct count               known analytical benchmarks (TPC-H, TPC-DS, SSB, and JOB), and
estimation, AMS sketches for join selectivity, and samples for filter          users can run both existing and custom queries for these datasets.
selectivity. As QueryBrew optimizes queries for other databases,                  To illustrate the potential of QueryBrew, we present three scenar-
it cannot collect statistics when data is inserted or updated. Nev-            ios in this section that demonstrate how query execution improves
ertheless, the required statistics can be computed in SQL, using               through unnesting and join reordering. Additionally, we explore
native hash functions, aggregations, and simple arithmetic oper-               the potential of this approach for SQL dialect translation.
ations. Hence, through SQL, we achieve a full decoupling of the                Scenario 1: Unnesting arbitrary queries. While general unnest-
optimizer from the query engine and storage: A perfect match for               ing can decorrelate arbitrary queries, only a few systems integrate
today’s open table formats and multi-engine landscape.                         the full algorithm as proposed by Neumann and Kemper. Imple-
Interface. QueryBrew provides a user interface for exploring and               menting this optimization is non-trivial and requires substantial
testing optimized query plans. It allows running queries on mul-               engineering effort. Through QueryBrew’s SQL-to-SQL optimiza-
tiple database systems, comparing their results and runtimes, and              tions, unnesting is available to all database systems. We invite
loading Umbra’s optimized query plan. It consists of the following             visitors to test this algorithm and explore the difference between
components: (1) a unified query editor and schema viewer, (2) a                systems that offer this optimization (Umbra, DuckDB, SQL Server)
per-system editor that allows adapting and optimizing queries for              and systems that do not (PostgreSQL, ClickHouse).
individual systems, (3) a result view, and lastly (4) a query plan
view with statistics. Figure 3 shows the full user interface.                  1 Due to SQL Server’s DeWitt clause, we do not report absolute times.




                                                                        4496
                                                                                                            SQL to SQL
    ! eryBrew                                                                                             Optimization



                                                                                 System
                                                                                ery Editor




                                                                            Result View                                                         Plan View

                    Schema Viewer




                        Uniﬁed
                       ery Editor



                                                       Figure 3: QueryBrew interface


Scenario 2: Identifying optimal join orders. Finding a good join                users explore and analyze SQL-to-SQL optimization and benchmark
order for complex queries is difficult and requires robust cardinality          the original and optimized queries across different target systems.
estimation. Choosing the wrong order can heavily impact the size
of intermediate results and therefore query execution time. For                 REFERENCES
some queries of JOB, we observe runtime improvements of more                     [1] 2021. Substrait. Retrieved March 1, 2026 from https://github.com/substrait-
                                                                                     io/substrait
than one order of magnitude (e.g., on ClickHouse or SQL Server).                 [2] David F. Bacon, Nathan Bales, Nicolas Bruno, Brian F. Cooper, Adam Dickinson,
Our demonstration allows users to compare Umbra’s join order                         Andrew Fikes, Campbell Fraser, Andrey Gubarev, Milind Joshi, Eugene Kogan,
with that of other systems.                                                          Alexander Lloyd, Sergey Melnik, Rajesh Rao, David Shue, Christopher Taylor,
                                                                                     Marcel van der Holst, and Dale Woodford. 2017. Spanner: Becoming a SQL
Scenario 3: SQL Dialect Translation. SQL dialects differ across                      System. In SIGMOD Conference. ACM, 331–343.
systems; as a side effect of SQL-to-SQL optimization, QueryBrew                  [3] Qiushi Bai, Sadeem Alsudais, and Chen Li. 2023. QueryBooster: Improving SQL
also supports translating queries from the PostgreSQL dialect to                     Performance Using Middleware Services for Human-Centered Query Rewriting.
                                                                                     Proc. VLDB Endow. 16, 11 (2023), 2911–2924.
other dialects. The user can specify the target dialect, and the                 [4] Ilaria Battiston, Kriti Kathuria, and Peter Boncz. 2024. OpenIVM: a SQL-to-SQL
operator-oriented query is generated to be compatible with the                       Compiler for Incremental Computations. In SIGMOD Conference, Pablo Barceló,
                                                                                     Nayat Sánchez-Pi, Alexandra Meliou, and S. Sudarshan (Eds.). ACM, 516–519.
target system. Examples of dialect-specific syntax include the miss-             [5] Altan Birler and Thomas Neumann. 2025. Efficient Enumeration of the Complete
ing boolean type in SQL Server and the renaming of functions for                     Join Search Space. In Proceedings of the 19th International Symposium on Database
ClickHouse and SQL Server. Unlike other SQL converters, such as                      Programming Languages. ACM, 1–12.
                                                                                 [6] Nicolas Bruno, César A. Galindo-Legaria, Milind Joshi, Esteban Calvo Vargas,
SQLGlot [9], QueryBrew benefits from Umbra’s optimizations and                       Kabita Mahapatra, Sharon Ravindran, Guoheng Chen, Ernesto Cervantes Juárez,
translates arbitrarily complex queries.                                              and Beysim Sezgin. 2024. Unified Query Optimization in the Fabric Data Ware-
                                                                                     house. In SIGMOD Conference Companion. ACM, 18–30.
                                                                                 [7] Haralampos Gavriilidis, Lennart Behme, Christian Munz, Varun Pandey, and
4   CONCLUSION                                                                       Volker Markl. 2025. CompoDB: A Demonstration of Modular Data Systems in
QueryBrew is a system-agnostic SQL-to-SQL optimization service                       Practice. In EDBT. OpenProceedings.org, 1094–1097.
                                                                                 [8] Jie Liu and Barzan Mozafari. 2024. GenRewrite: Query Rewriting via Large
that leverages Umbra’s optimizer to exploit state-of-the-art opti-                   Language Models. CoRR abs/2403.09060 (2024).
mization techniques. Input queries are transformed into relational               [9] Toby Mao. [n. d.]. SQLGlot. Retrieved March 1, 2026 from https://github.com/
                                                                                     tobymao/sqlglot
algebra, then simplified, unnested, and reordered, and finally trans-           [10] Thomas Neumann. 2025. Improving Unnesting of Complex Queries. In BTW
lated back into an operator-oriented SQL query. The operator-                        (LNI, Vol. P-361). Gesellschaft für Informatik e.V., 25–47.
oriented query can then be executed canonically by the target                   [11] Thomas Neumann and Michael J. Freitag. 2020. Umbra: A Disk-Based System
                                                                                     with In-Memory Performance. In CIDR. www.cidrdb.org.
system. SQL-to-SQL rewriting gives the target system a simpler                  [12] Thomas Neumann and Alfons Kemper. 2015. Unnesting Arbitrary Queries. In
and more structured representation of the input query without                        BTW (LNI, Vol. P-241). GI, 383–402.
changing its semantics. This helps query optimizers discover more               [13] Thomas Neumann and Bernhard Radke. 2018. Adaptive Optimization of Very
                                                                                     Large Join Queries. In SIGMOD Conference. ACM, 677–692.
efficient plans even when they lack some advanced optimizations.                [14] Yuanyuan Tian. 2025. Query Optimization in the Wild: Realities and Trends.
   QueryBrew supports the dialects of various systems (SQL Server,                   CoRR abs/2510.20082 (2025).
                                                                                [15] Yuanyuan Tian, Jesús Camacho-Rodríguez, Carlo Curino, César A. Galindo-
ClickHouse, DuckDB, and PostgreSQL) and matches their syntax                         Legaria, Ashit Gosalia, Brian Kroth, Sergiy Matusevych, Nicolas Bruno, Ashvin
and semantic subtleties. Using this SQL-to-SQL optimization, we                      Agrawal, Stefan Grafberger, Beysim Sezgin, Milan Potocnik, Mahesh Behera,
observe speedups by more than one order of magnitude on standard                     Milind Joshi, and Xiaoyu Li. 2025. Towards Query Optimizer as a Service
                                                                                     (QOaaS) in a Unified LakeHouse Platform: Can One QO Rule Them All?. In
benchmarks. Our demo features an easy-to-use website that lets                       CIDR. www.cidrdb.org.




                                                                         4497

