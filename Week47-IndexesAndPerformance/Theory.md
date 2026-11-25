[Week47_Exercises.md](https://github.com/user-attachments/files/35434730/Week47_Exercises.md)
# Week 47 — Exercises: Indexes and Query Performance

> 
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises walk you through measuring, creating, and evaluating indexes on your TrailShop database. You will use PostgreSQL's `EXPLAIN ANALYZE` to gather real execution data and make evidence-based decisions about indexing.

---

## 1. TrailShop Project Task

### Step 1 — Baseline Measurement

Before creating any indexes, run `EXPLAIN ANALYZE` on the following five queries. For each, record the **scan type**, **execution time (ms)**, and **rows removed by filter**.

`

--
Record your results in a table like this:

| Query | Scan Type | Exec Time (ms) | Rows Removed by Filter |
| ----- | --------- | -------------- | ---------------------- |
| A     |           |                |                        |
| B     |           |                |                        |
| C     |           |                |                        |
| D     |           |                |                        |
| E     |           |                |                        |

### Step 2 — Create Indexes

Now create the following indexes:

```sql
CREATE INDEX idx_product_name ON products (name);
CREATE INDEX idx_order_item_order_id ON order_items (order_id);
CREATE INDEX idx_order_item_product_id ON order_items (product_id);
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
```

### Step 3 — Re-measure

Because the project database is so small, the PostgreSQL optimizer will most likely default to Seq Scan even after creating indexes.

You can force PostgreSQL to use indexes by disabling the Seq scan using following commands:

```sql


Run the same five queries with `EXPLAIN ANALYZE` again. Fill in your comparison table:

| Query | Before Scan | Before Time (ms) | After Scan | After Time (ms) | Improvement Factor |
| ----- | ----------- | ---------------- | ---------- | --------------- | ------------------ |
| A     |             |                  |            |                 |                    |
| B     |             |                  |            |                 |                    |
| C     |             |                  |            |                 |                    |
| D     |             |                  |            |                 |                    |
| E     |             |                  |            |                 |                    |

### Step 4 — Recommendation

Write a paragraph (4–6 sentences) recommending which indexes to keep in production and why. Consider table size, query frequency, and write overhead. If any index showed minimal improvement, explain why you might drop it.

> [!NOTE]
> **_Your Answer_**
>
> Baseline measurements must be taken from my own PostgreSQL database. I will run the five EXPLAIN ANALYZE queries and copy the scan type, execution time, and rows removed by filter into the table.

---

## 2. Theory Review

Answer the following questions in your own words. See the corresponding theory sections for reference.

**Question 1** — What is a database index, and why is it needed? (See Theory Section 1)


> 
>
 The indexes on products.name, order_items.order_id, order_items.product_id, orders.customer_id, and product_categories.category_id are useful for the searches and joins in these queries. They can become more helpful when the tables get bigger and the queries are used often. However, indexes also use storage and make INSERT, UPDATE, and DELETE a little more expensive. If an index gives almost no benefit because the table is very small, I would consider removing it.

**Question 2** — Explain the difference between B-tree search at O(log n) and sequential scan at O(n). If a table has 1,000,000 rows, approximately how many comparisons does each approach need? (See Theory Section 2)

> [!NOTE]
> **_Your Answer_**
>
> An index is a data structure that helps PostgreSQL find rows faster. It is similar to the index of a book because PostgreSQL does not always have to check every row. Indexes are especially useful for large tables and common search queries.

**Question 3** — PostgreSQL automatically creates indexes on primary keys. Why doesn't it also auto-index foreign key columns? (See Theory Section 3)

> [!NOTE]
> **_Your Answer_**
>
> A B-tree search is O(log n), while a sequential scan is O(n). With 1,000,000 rows, a B-tree needs about log2(1,000,000), which is around 20 comparisons. A sequential scan may need to check up to about 1,000,000 rows. So the B-tree can be much faster when only a few rows are needed.

**Question 4** — What is the difference between an Index Scan and an Index Only Scan? When can PostgreSQL use Index Only Scan? (See Theory Section 4)

>
> Answer_
>
> PostgreSQL automatically creates an index for a primary key, but it does not automatically create one for every foreign key. Not every foreign key needs an index, and creating all of them would use extra storage and make writes slower. We can add an index when the foreign key is often used in searches or joins.

**Question 5** — Give three situations where you should NOT create an index. Explain each briefly. (See Theory Section 5)

> 
>  Answer
>
> An Index Scan uses an index to find the rows, but PostgreSQL may still need to read the table to get other columns. An Index Only Scan can get all required columns from the index itself. PostgreSQL can use it when the index contains all the columns needed by the query and the visibility information allows it.

**Question 6** — What does `EXPLAIN ANALYZE` show that plain `EXPLAIN` does not? Why does this matter? (See Theory Section 6)

> 
> Your Answer
>
> I would not create an index on a very small table because a sequential scan may already be fast. I would also avoid an index on a column with very few different values if many rows have the same value. Finally, I would avoid creating too many indexes on columns that change often because INSERT, UPDATE, and DELETE operations have to maintain the indexes.

**Question 7** — What is a partial index? Give one concrete example where a partial index on a TrailShop table would be useful. (See Theory Section 7)

> 
> 
>
> EXPLAIN shows the execution plan PostgreSQL expects to use. EXPLAIN ANALYZE actually runs the query and shows real information such as execution time and actual rows. This is useful because the real result can be different from PostgreSQL's estimates, so it helps us check if an index really improves performance.

**Question 8** — What is the leftmost prefix rule for multi-column indexes? If you create `CREATE INDEX idx ON orders (customer_id, order_date)`, which of the following queries can use the index and why?

```sql
-- a)
SELECT * FROM orders WHERE customer_id = 5;
-- b)
SELECT * FROM orders WHERE order_date = '2026-01-15';
-- c)
SELECT * FROM orders WHERE customer_id = 5 AND order_date = '2026-01-15';
```

(See Theory Section 8)

> 
> 
>
> A partial index is an index that contains only rows that match a condition. For example, if TrailShop often searches for pending orders, this index could be useful:

```

```

It is smaller than a full index on the table and can make searches for pending orders faster.

---

## 3. EXPLAIN Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Below are four `EXPLAIN ANALYZE` outputs. For each one, identify:

1. The scan type used
2. Estimated rows vs actual rows
3. Total execution time
4. Whether an index would help (and which column to index)

### Output A — Sequential Scan

```

```



- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would an index help? Which one?

> [!NOTE]
> **_Your Answer_**
>
> The leftmost prefix rule means that PostgreSQL can use the beginning of a multi-column index. For `CREATE INDEX idx ON orders (customer_id, order_date)`, query (a) can use the index because it uses `customer_id`, which is the first column. Query (b) normally cannot use this index effectively because it only uses `order_date`, the second column. Query (c) can use the index because it uses both columns starting with `customer_id`.

### Output B — Index Scan

```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Is the index effective here?

> 
> 
>
> Scan type: Sequential Scan on products.

Estimated vs actual rows: estimated 5 rows, actual 3 rows.

Execution time: 0.312 ms.

An index on `products(category_id)` could help when the table is large and this search is common. For the small table in this example, the sequential scan may still be faster.

### Output C — Bitmap Index Scan

```
Bitmap Heap Scan on order_items  (cost=4.32..35.76 rows=25 width=24) (actual time=0.045..0.098 rows=28 loops=1)
  Recheck Cond: (product_id = 15)
  Heap Blocks: exact=12
  ->  Bitmap Index Scan on idx_order_item_product_id  (cost=0.00..4.31 rows=25 width=0) (actual time=0.030..0.030 rows=28 loops=1)
        Index Cond: (product_id = 15)
Planning Time: 0.091 ms
Execution Time: 0.132 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Why did PostgreSQL choose a Bitmap scan instead of a plain Index Scan?

> [!NOTE]
> **_Your Answer_**
>
> Scan type: Index Scan using `idx_orders_customer_id`.

Estimated vs actual rows: estimated 1 row, actual 4 rows.

Execution time: 0.041 ms.

Yes, the index is effective here because PostgreSQL can use `customer_id` to find the matching orders without scanning the whole table.

### Output D — Hash Join

```
Hash Join  (cost=33.50..75.82 rows=120 width=52) (actual time=0.452..1.205 rows=115 loops=1)
  Hash Cond: (oi.product_id = p.id)
  ->  Seq Scan on order_items oi  (cost=0.00..30.40 rows=2040 width=16) (actual time=0.008..0.195 rows=2040 loops=1)
  ->  Hash  (cost=20.00..20.00 rows=500 width=40) (actual time=0.410..0.411 rows=500 loops=1)
        Buckets: 1024  Batches: 1  Memory Usage: 45kB
        ->  Seq Scan on products p  (cost=0.00..20.00 rows=500 width=40) (actual time=0.005..0.180 rows=500 loops=1)
Planning Time: 0.215 ms
Execution Time: 1.340 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would adding an index improve this join? Which column?

> [!NOTE]
> **_Your Answer_**
>
> Scan type: Bitmap Heap Scan with a Bitmap Index Scan underneath.

Estimated vs actual rows: estimated 25 rows, actual 28 rows.

Execution time: 0.132 ms.

PostgreSQL chose a bitmap scan because there are several matching rows. It can first collect the matching row locations with the index and then read the required table pages together. This can be more efficient than doing many separate index lookups.

---

## 4. Index Design Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, recommend the best index(es), explain why, and predict the scan type PostgreSQL will use after indexing.

**Query 1**

```sql
SELECT * FROM customers WHERE email = 'anna.korhonen@email.fi';
```

**Query 2**

```sql
SELECT * FROM orders WHERE customer_id = 5 ORDER BY order_date DESC;
```

**Query 3**

```sql
SELECT product_id, SUM(quantity) FROM order_items GROUP BY product_id;
```

**Query 4**

```sql
SELECT * FROM products WHERE price BETWEEN 50 AND 100;
```

**Query 5**

```sql
SELECT p.name, c.name AS category
FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
JOIN categories c ON c.category_id = pc.category_id
WHERE pc.category_id = 2;
```

**Query 6**

```sql
SELECT * FROM payments WHERE order_id = 88;
```

**Query 7**

```sql
SELECT * FROM orders WHERE status = 'shipped' AND order_date > '2026-06-01';
```

**Query 8**

```sql
SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;
```

**Query 9**

```sql
SELECT oi.quantity, p.name
FROM order_items oi
JOIN products p ON p.id = oi.product_id
WHERE oi.order_id = 100;
```

**Query 10**

```sql
SELECT * FROM products WHERE name LIKE 'Trail%';
```

For each query, fill in:

| #   | Recommended Index | Why It Helps | Expected Scan Type |
| --- | ----------------- | ------------ | ------------------ |
| 1 | `CREATE UNIQUE INDEX idx_customers_email ON customers (email);` | Finds a customer quickly by email. | Index Scan |
| 2 | `CREATE INDEX idx_orders_customer_order_date ON orders (customer_id, order_date DESC);` | Filters by customer and keeps the results in date order. | Index Scan |
| 3 | `CREATE INDEX idx_order_items_product_id ON order_items (product_id);` | Helps access order items by product. | Index Scan or Seq Scan |
| 4 | `CREATE INDEX idx_products_price ON products (price);` | Helps with the price range search. | Index Scan or Bitmap Scan |
| 5 | `CREATE INDEX idx_product_categories_category_id_product_id ON product_categories (category_id, product_id);` | Finds products in a category and provides product IDs for the join. | Index Scan |
| 6 | `CREATE INDEX idx_payments_order_id ON payments (order_id);` | Finds payments for one order quickly. | Index Scan |
| 7 | `CREATE INDEX idx_orders_status_order_date ON orders (status, order_date);` | Helps filter by status and then by date. | Index Scan or Bitmap Scan |
| 8 | `CREATE INDEX idx_orders_customer_id ON orders (customer_id);` | Helps access orders grouped by customer, although PostgreSQL may still use a sequential scan for aggregation. | Index Scan or Seq Scan |
| 9 | `CREATE INDEX idx_order_items_order_id ON order_items (order_id);` | Quickly finds the items belonging to order 100. | Index Scan |
| 10 | `CREATE INDEX idx_products_name ON products (name);` | Can help with a prefix search such as `LIKE 'Trail%'`. | Index Scan |

---

## 5. Performance Comparison Exercise

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, follow these steps:

1. Run `EXPLAIN ANALYZE` **without** an index — record the full output
2. Create the recommended index
3. Run `EXPLAIN ANALYZE` **again** — record the full output
4. Calculate the improvement factor (before time ÷ after time)
5. Write one sentence explaining why the improvement occurred (in the **Explanation** field under each comparison)

### Comparison 1

```sql
-- Query
SELECT * FROM order_items WHERE order_id = 55;

-- Index to create
CREATE INDEX idx_oi_order_id ON order_items (order_id);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> Scan type: Hash Join, with Sequential Scans on both input tables.

Estimated vs actual rows: estimated 120 rows, actual 115 rows.

Execution time: 1.340 ms.

An index on `order_items(product_id)` can help queries that filter or join using `product_id`. However, for this particular example PostgreSQL reads both whole tables and uses a hash join, so an index is not guaranteed to make this query faster.

### Comparison 2

```sql
-- Query
SELECT * FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
WHERE pc.category_id = 2 AND p.price < 80;

CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
CREATE INDEX idx_products_price ON products (price);

-- Index to create
CREATE INDEX idx_products_cat_price ON products (category_id, price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> Baseline measurements must be taken from the actual PostgreSQL database. After creating the index, run the same query again and copy the real scan type, execution time, and rows from PostgreSQL. The improvement factor is `Before execution time / After execution time`.

### Comparison 3

```sql
-- Query
SELECT * FROM customers WHERE email = 'test.user@example.com';

-- Index to create
CREATE UNIQUE INDEX idx_customers_email ON customers (email);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> The indexes can reduce the amount of data PostgreSQL needs to check when filtering by category and price. The exact improvement depends on the size and data distribution of the TrailShop database, so the real values should be copied from PostgreSQL.

### Comparison 4

```sql
-- Query
SELECT order_id, SUM(quantity * unit_price) AS total
FROM order_items
WHERE order_id IN (10, 20, 30, 40, 50)
GROUP BY order_id;

-- Index to create
CREATE INDEX idx_oi_order_id_covering ON order_items (order_id) INCLUDE (quantity, unit_price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> The unique email index can let PostgreSQL find the customer directly instead of checking every customer row. The exact execution time depends on the actual database, so the values should be copied from PostgreSQL.

### Comparison 5

```sql
-- Query
SELECT * FROM orders WHERE status = 'pending';

-- Index to create (partial index)
CREATE INDEX idx_orders_pending ON orders (status) WHERE status = 'pending';
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> The covering index contains `order_id`, `quantity`, and `unit_price`, so PostgreSQL may be able to get the needed values directly from the index. The exact performance depends on the actual database.

---

## Submission Checklist

**Required**

- [ ] Baseline measurements for all 5 project queries
- [ ] All 5 indexes created and re-measured
- [ ] Comparison table filled with before/after data
- [ ] Recommendation paragraph written
- [ ] All 8 theory questions answered

**Recommended practice**

- [ ] All 4 EXPLAIN outputs analyzed
- [ ] Index recommendations for all 10 queries
- [ ] All 5 performance comparisons completed with explanations
