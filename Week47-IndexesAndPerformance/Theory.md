[Week47_Theory.md](https://github.com/user-attachments/files/35434965/Week47_Theory.md)
# Week 47 — Theory: Indexes and Query Performance

## Question 1 — What is a database index, and why is it needed?

A database index is a data structure that helps PostgreSQL find rows faster. It works like an index in a book: instead of checking every row, PostgreSQL can use the index to find the required data more quickly. Indexes are especially useful for large tables and frequently used search conditions. The disadvantage is that indexes use storage and can make INSERT, UPDATE, and DELETE operations slower.

## Question 2 — B-tree O(log n) vs sequential scan O(n)

A B-tree index can find a value in approximately O(log n) time, while a sequential scan checks rows one by one and is O(n).

For 1,000,000 rows:
- B-tree: about log2(1,000,000) ≈ 20 comparisons.
- Sequential scan: potentially up to 1,000,000 row checks.

Therefore, an index can be much faster when only a small number of rows are needed.

## Question 3 — Why are foreign key columns not automatically indexed?

PostgreSQL automatically creates an index for a primary key, but it does not automatically create an index for a foreign key. A foreign key does not always need an index, and automatically creating one for every foreign key would use extra storage and make writes slower. The database designer can create an index when the foreign key is frequently used for searches, joins, or other queries.

## Question 4 — Index Scan vs Index Only Scan

An Index Scan uses an index to find the required rows, but PostgreSQL may still need to read the table itself to get columns that are not available in the index.

An Index Only Scan can return the required data directly from the index without reading the table for every row. PostgreSQL can use it when the index contains all columns needed by the query and the visibility information allows PostgreSQL to avoid checking the table rows.

## Question 5 — Three situations where you should NOT create an index

1. **Very small tables:** A sequential scan can already be very fast, so an index may not provide a useful improvement.
2. **Columns with very few distinct values:** For example, a column containing only a few status values may not benefit much from a normal index if many rows match.
3. **Columns that change frequently:** Indexes need to be updated when indexed values change, so too many indexes can increase INSERT, UPDATE, and DELETE overhead.

## Question 6 — EXPLAIN ANALYZE vs EXPLAIN

`EXPLAIN` shows the execution plan PostgreSQL expects to use, including estimated costs and rows.

`EXPLAIN ANALYZE` actually runs the query and reports real execution information such as actual rows and execution time. This matters because the real result can be different from PostgreSQL's estimates. It helps us check whether an index or another query plan really improves performance.

## Question 7 — What is a partial index?

A partial index is an index that contains only rows matching a specified condition. This makes the index smaller and can make some queries faster.

For example, if TrailShop often searches for pending orders, we can create:

```sql
CREATE INDEX idx_orders_pending
ON orders (status)
WHERE status = 'pending';
```

This index stores only pending orders instead of every order.

## Question 8 — Leftmost prefix rule

For this index:

```sql
CREATE INDEX idx ON orders (customer_id, order_date);
```

the first column is `customer_id`, so PostgreSQL can use the index effectively when the query uses `customer_id`.

**a) Yes**

```sql
SELECT * FROM orders WHERE customer_id = 5;
```

The query uses the first column of the index, so the index can be used.

**b) Usually no**

```sql
SELECT * FROM orders WHERE order_date = '2026-01-15';
```

The query only uses the second column. Because `customer_id` is the leftmost column, this index is not normally useful for directly searching by `order_date` alone.

**c) Yes**

```sql
SELECT * FROM orders
WHERE customer_id = 5 AND order_date = '2026-01-15';
```

Both columns are used, starting with the leftmost column, so the index can be used very effectively.
