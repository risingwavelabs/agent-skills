---
title: Do not rely on ORDER BY in materialized views for ongoing ordering
impact: MEDIUM
impactDescription: ORDER BY in a materialized view is silently ignored for streaming results — downstream consumers relying on it receive unordered data
tags: materialized-view, order-by, ordering, streaming, serving
---

## Do not rely on ORDER BY in materialized views for ongoing ordering

`ORDER BY` in a `CREATE MATERIALIZED VIEW` statement is applied only to the initial snapshot at creation time. As new data streams in and the MV is incrementally updated, results are not re-sorted. Downstream consumers that assume the MV is always ordered will see unordered data.

**Incorrect (ORDER BY in MV — ordering not maintained for streaming updates):**
```sql
-- Bad: ORDER BY only applies to the initial snapshot
CREATE MATERIALIZED VIEW recent_orders AS
SELECT order_id, customer_id, total, created_at
FROM orders
ORDER BY created_at DESC;
-- After backfill, new rows are appended without re-sorting
```

**Correct (order at query time or use a sink):**
```sql
-- Good: create the MV without ORDER BY
CREATE MATERIALIZED VIEW recent_orders AS
SELECT order_id, customer_id, total, created_at
FROM orders;

-- Apply ORDER BY at query time when serving
SELECT order_id, customer_id, total, created_at
FROM recent_orders
ORDER BY created_at DESC
LIMIT 100;
```

**If ordered output to a sink is required**, use a windowed MV with `EMIT ON WINDOW CLOSE` and sort in the downstream consumer, or use a batch export job.

Reference: [CREATE MATERIALIZED VIEW](https://docs.risingwave.com/sql/commands/sql-create-mv)
