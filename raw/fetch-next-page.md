---
url: https://use-the-index-luke.com/sql/partial-results/fetch-next-page
date_fetched: 2026-08-08
---

After implementing a pipelined top-N query to retrieve the first page efficiently, you will often also need another query to fetch the next pages. The resulting challenge is that it has to skip the rows from the previous pages. There are two different methods to meet this challenge: firstly the *offset method*, which numbers the rows from the beginning and uses a filter on this row number to discard the rows before the requested page. The second method, which I call the *seek method*, searches the last entry of the previous page and fetches only the following rows.

The following examples show the more widely used offset method. Its main advantage is that it is very easy to handle—especially with databases that have a dedicated keyword for it (`offset`). This keyword was even taken into the SQL standard as part of the `fetch first` extension.

- Db2 (LUW)
- Db2 supports - `offset`since release 11.1. The standard conforming alternative using- `ROW_NUMBER()`window function (see next section) works in earlier releases. There are two other ways to get offset functionality, none of them recommendable: (1) using- `db2set DB2_COMPATIBILITY_VECTOR=MYS`to enable- `limit`and- `offset`like MySQL supports it. This does, however, not allow to combine- `fetch first`with- `offset`; (2) using- `db2set DB2_COMPATIBILITY_VECTOR=ORA`to get Oracle’s- `ROWNUM`pseudo column (see Oracle example).
- MySQL
- MySQL and PostgreSQL offer the - `offset`clause for discarding the specified number of rows from the beginning of a top-N query. The- `limit`clause is applied afterwards.- `SELECT * FROM sales ORDER BY sale_date DESC LIMIT 10`- **OFFSET 10**
- Oracle
- The Oracle database supports - `offset`since release 12c. Earlier releases provide the pseudo column- `ROWNUM`that numbers the rows in the result set automatically. It is, however, not possible to apply a greater than or equal to (- `>=`- `SELECT * FROM ( SELECT tmp.*,`- **rownum rn**FROM ( SELECT * FROM sales ORDER BY sale_date DESC ) tmp WHERE- **rownum <= 20**) WHERE- **rn > 10**- Note the use of the alias - `RN`for the lower bound and the- `ROWNUM`pseudo column itself for the upper bound (thanks to Tom Kyte).
- PostgreSQL
- The - `fetch first`extension defines an- `offset ... rows`clause as well. PostgreSQL, however, only accepts- `offset`without the- `rows`keyword. The previously used- `limit/offset`syntax still works as shown in the MySQL example.- `SELECT * FROM sales ORDER BY sale_date DESC`- **OFFSET 10 FETCH NEXT 10 ROWS ONLY**
- SQL Server
- SQL Server does not have an “offset” extension for its proprietary - `top`clause but introduced the- `fetch first`extension with SQL Server 2012. The- `offset`clause is mandatory although the standard defines it as an optional addendum.- `SELECT * FROM sales ORDER BY sale_date DESC`- **OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY**

Besides the simplicity, another advantage of this method is that you just need the row offset to fetch an arbitrary page. Nevertheless, the database must count all rows from the beginning until it reaches the requested page. Figure 7.2 shows that the scanned index range becomes greater when fetching more pages.

## Figure 7.2 Access Using the Offset Method

This has two disadvantages: (1) the pages drift when inserting new sales because the numbering is always done from scratch; (2) the response time increases when browsing further back.

#### If you like this page, you might also like …

… to subscribe my **mailing lists**, get **free stickers**, buy **my book** or join a **training**.

The seek method avoids both problems because it uses the *values* of the previous page as a delimiter. That means it searches for the values that must *come behind* the last entry from the previous page. This can be expressed with a simple `where` clause. To put it the other way around: the seek method simply doesn’t select already shown values.

The next example shows the seek method. For the sake of demonstration, we will start with the assumption that there is only one sale per day. This makes the `SALE_DATE` a unique key. To select the sales that must come behind a particular date you must use a less than condition (`<`) because of the descending sort order. For an ascending order, you would have to use a greater than (`>`) condition. The `fetch first` clause is just used to limit the result to ten rows.

```
SELECT *
  FROM sales
 
```
**WHERE sale_date < ?**
 ORDER BY sale_date DESC
 FETCH FIRST 10 ROWS ONLYInstead of a row number, you use the last value of the previous page to specify the lower bound. This has a huge benefit in terms of performance because the database can use the `SALE_DATE < ?` condition for index access. That means that the database can truly skip the rows from the previous pages. On top of that, you will also get stable results if new rows are inserted.

Nevertheless, this method does not work if there is more than one sale per day—as shown in Figure 7.2—because using the last date from the first page (“yesterday”) skips *all* results from yesterday—not just the ones already shown on the first page. The problem is that the `order by` clause does not establish a deterministic row sequence. That is, however, prerequisite to using a simple range condition for the page breaks.

Without a deterministic `order by` clause, the database by definition does not deliver a deterministic row sequence. The only reason you *usually* get a consistent row sequence is that the database *usually* executes the query in the same way. Nevertheless, the database could in fact shuffle the rows having the same `SALE_DATE` and still fulfill the `order by` clause. In recent releases it might indeed happen that you get the result in a different order every time you run the query, not because the database shuffles the result intentionally but because the database might utilize parallel query execution. That means that the same execution plan can result in a different row sequence because the executing threads finish in a non-deterministic order.

#### Important

Paging requires a deterministic sort order.

Even if the functional specifications only require sorting “by date, latest first”, we as the developers must make sure the `order by` clause yields a deterministic row sequence. For this purpose, we might need to extend the `order by` clause with arbitrary columns just to make sure we get a deterministic row sequence. If the index that is used for the pipelined `order by` has additional columns, it is a good start to add them to the `order by` clause so we can continue using this index for the pipelined `order by`. If this still does not yield a deterministic sort order, just add any unique column(s) and extend the index accordingly.

In the following example, we extend the `order by` clause and the index with the primary key `SALE_ID` to get a deterministic row sequence. Furthermore, we must apply the “comes after” logic to both columns *together* to get the desired result:

`CREATE INDEX sl_dtid ON sales (sale_date, sale_id)````
SELECT *
  FROM sales
```
** WHERE (sale_date, sale_id) < (?, ?)**
 ORDER BY sale_date DESC, **sale_id DESC**
 FETCH FIRST 10 ROWS ONLYThe `where` clause uses the little-known “row values” syntax (see the box entitled “*SQL Row Values*”). It combines multiple values into a logical unit that is applicable to the regular comparison operators. As with scalar values, the less-than condition corresponds to “comes after” when sorting in descending order. That means the query considers only the sales that come after the given `SALE_DATE`, `SALE_ID` pair.

Even though the row values syntax is part of the SQL standard, only a few databases support it. SQL Server 2017 does not support row values at all. The Oracle database supports row values in principle, but cannot apply range operators on them (ORA-01796). MySQL evaluates row value expressions correctly but cannot use them as access predicate during an index access. Db2 (only LUW, since 10.1) and PostgreSQL (since 8.4), however, have a proper support of row value predicates *and* uses them to access the index if there is a corresponding index available.

Nevertheless it is possible to use an approximated variant of the seek method with databases that do not properly support the row values—even though the approximation is not as elegant and efficient as row values in PostgreSQL. For this approximation, we must use “regular” comparisons to express the required logic as shown in this Oracle example:

```
SELECT *
  FROM ( SELECT *
           FROM sales
          WHERE 
```
**sale_date <= ?
            AND NOT (sale_date = ? AND sale_id >= ?)**
          ORDER BY sale_date DESC, sale_id DESC
       )
 WHERE rownum <= 10The `where` clause consists of two parts. The first part considers the `SALE_DATE` only and uses a less than or equal to (`<=`) condition—it selects more rows as needed. This part of the `where` clause is simple enough so that all databases can use it to access the index. The second part of the `where` clause removes the excess rows that were already shown on the previous page. The box entitled “*Indexing Equivalent Logic*” explains why the `where` clause is expressed this way.

The execution plan shows that the database uses the first part of the `where` clause as access predicate.

```
---------------------------------------------------------------
|Id | Operation                      | Name    |  Rows | Cost |
---------------------------------------------------------------
| 0 | SELECT STATEMENT               |         |    10 |    4 |
|*1 |  COUNT STOPKEY                 |         |       |      |
| 2 |   VIEW                         |         |    10 |    4 |
| 3 |    TABLE ACCESS BY INDEX ROWID | SALES   | 50218 |    4 |
|*4 |     INDEX RANGE SCAN DESCENDING| SL_DTIT |     2 |    3 |
---------------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------
   1 - filter(ROWNUM<=10)
   
```
**4 - access(**"SALE_DATE"<=:SALE_DATE)
       **filter**("SALE_DATE"<>:SALE_DATE
           OR "SALE_ID"<TO_NUMBER(:SALE_ID))The access predicates on `SALE_DATE` enables the database to skip over the days that were fully shown on previous pages. The second part of the `where` clause is a filter predicate only. That means that the database inspects a few entries from the previous page again, but drops them immediately. Figure 7.3 shows the respective access path.

## Figure 7.3 Access Using the Seek Method

Figure 7.4 compares the performance characteristics of the offset and the seek methods. The accuracy of measurement is insufficient to see the difference on the left hand side of the chart, however the difference is clearly visible from about page 20 onwards.

## Figure 7.4 Scalability when Fetching the Next Page

`where` clause very carefully—you also cannot fetch arbitrary pages. Moreover you need to reverse all comparison and sort operations to change the browsing direction. Precisely these two functions—skipping pages and browsing backwards—are not needed when using an infinite scrolling mechanism for the user interface.
