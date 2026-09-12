---
title: "Materialized Views Are Obviously Useful"
author: Sophie Alpert
date: 2025-08-22
url: https://sophiebits.com/2025/08/22/materialized-views-are-obviously-useful
fetched: 2026-05-14
topics:
  - databases-and-data
---

# Materialized Views Are Obviously Useful

Sophie Alpert discusses the practical challenges of maintaining synchronized data across systems in application development, illustrated through a task-tracking application scenario.

## The Initial Problem

The author begins with a straightforward SQL query to count tasks per project:

```sql
select count(1) from tasks where project_id = $1
```

Performance issues emerge when the query requires scanning entire task indexes repeatedly.

## Attempted Solutions

**Caching approach:** Adding Redis caching with one-hour expiration initially seems efficient but creates accuracy problems. Users observe stale counts after creating or deleting tasks.

**Incremental updates:** The next strategy involves incrementing/decrementing cached counts rather than recalculating totals. This requires Lua scripts to safely handle missing Redis keys. However, complications arise when tasks move between projects, necessitating updates to multiple counters.

## Growing Complexity

The author describes accumulating issues: server crashes can cause database writes without corresponding cache updates, creating permanent inconsistencies. Solutions like Kafka or database transactions add significant infrastructure complexity. She notes that "correctness of my system today depends not only on the code being correct right now but also on my code having done the correct thing at every point in the past."

## The Proposed Alternative

Sophie advocates for incremental view maintenance technology — a database feature that automatically maintains materialized views. Users would declare:

```sql
create materialized view projects_task_count as
select project_id, count(1) as count
from tasks
group by project_id
```

The system would automatically handle all incremental updates through dataflow graph analysis, eliminating application-level synchronization logic.

## Conclusion

She predicts that within a decade, most database systems will incorporate this capability, making it a priority feature for database providers.
