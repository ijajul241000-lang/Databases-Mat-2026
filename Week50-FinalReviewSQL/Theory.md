# Week 50 — Final Review: SQL

> **Exam review week.** No new concepts and no required work. This chapter reviews the SQL you are expected to write, plus the fundamentals of transactions, indexes, and administration from Weeks 46–48.

## Chapter 14: "The SQL Reference"

Week 49 reviewed how the TrailShop tables were designed. This week reviews how you query and protect them. Every example uses the current schema: categories and products meet in `product_categories`, and a line price is `order_items.unit_price`.

---

## Learning Objectives

After this review you will be able to:

- Write `CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE`, and a basic `SELECT`
- Filter, sort, limit, aggregate, group, and join rows, and explain the logical order of those clauses
- Write a multi-table join, an outer join that keeps unmatched rows, a subquery, a view, and a CTE
- Write a transaction with `COMMIT`, `ROLLBACK`, and `SAVEPOINT`, and name the ACID properties and the concurrency problems
- Create an index, read a sequential scan and an index scan, and say when an index is a poor idea
- Create a role, grant and revoke privileges, take a logical backup, and explain parameterized queries and the relational-versus-document choice

---

## How to Use This Chapter

Each section is the syntax to practise and one TrailShop example. The exercises that follow use the same table and column names.

These topics were marked advanced in the weekly chapters. They are not the focus of this review, and the exercises do not ask for them: set operations (`UNION`, `INTERSECT`, `EXCEPT`), `CASE`, `COALESCE`, `NULLIF`, recursive CTEs, materialized views, window functions, join algorithms, point-in-time recovery, column-level privileges, and index types other than the B-tree you create with `CREATE INDEX`.

---

## The schema you are querying

```
categories(category_id PK, category_name UNIQUE, description)
customers(customer_id PK, first_name, last_name, email UNIQUE, phone,
          street, city, postal_code, country, registered_at)
products(product_id PK, name, description, price, weight_kg,
         stock_quantity, created_at)
product_categories(product_id PK/FK, category_id PK/FK)
orders(order_id PK, customer_id FK, order_date, status, shipping columns)
order_items(order_id PK/FK, product_id PK/FK, quantity, unit_price)
payments(payment_id PK, order_id UNIQUE FK, amount, payment_method, payment_date)
```

The product column is `name`. Stock is `stock_quantity`. Categories are reached through `product_categories`.

---

## 1. Week 40 — Creating and Changing Data

### CREATE TABLE

Define parent tables before the tables that reference them.

```sql
CREATE TABLE orders (
    order_id    SERIAL       PRIMARY KEY,
    customer_id INTEGER      NOT NULL
                REFERENCES customers(customer_id)
                ON DELETE RESTRICT
                ON UPDATE CASCADE,
    order_date  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    status      VARCHAR(20)  NOT NULL DEFAULT 'pending'
                CHECK (status IN (
                    'pending', 'processing', 'shipped',
                    'delivered', 'cancelled'
                ))
);
```

Column constraints you should be able to use: `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `DEFAULT`, `CHECK`, and `REFERENCES`. A table-level constraint can be named: `CONSTRAINT orders_status_allowed CHECK (...)`.

### INSERT, UPDATE, DELETE

```sql
INSERT INTO products (name, price, stock_quantity)
VALUES ('Ridge Tent', 249.00, 8);

INSERT INTO products (name, price)
VALUES
    ('Stove', 49.00),
    ('Sleeping Bag', 89.00);

UPDATE products
SET price = price * 1.10
WHERE product_id = 10;

DELETE FROM products
WHERE product_id = 10;
```

`INSERT` lists the columns it fills. Omitted columns must have a default or allow `NULL`. `SERIAL` fills the primary key. `UPDATE` and `DELETE` change every row that matches `WHERE`. Without `WHERE` they change every row in the table.

A single statement is already its own transaction (autocommit). Open an explicit transaction, reviewed in Section 5, when several statements must succeed or fail together.

String literals use single quotes: `'pending'`. Double quotes name an identifier, such as a column that needs quoting. They do not make a string.

### A basic SELECT

```sql
SELECT name, price
FROM products
WHERE price > 100;
```

### Common mistake

Running `UPDATE` or `DELETE` before you have written, and read, the `WHERE` clause.

---

## 2. Week 41 — Queries

### Filtering, sorting, and limiting

```sql
SELECT name, price, stock_quantity
FROM products
WHERE price BETWEEN 50 AND 150
  AND stock_quantity > 0
  AND name ILIKE '%tent%'
ORDER BY price DESC
LIMIT 10 OFFSET 0;
```

| Operator | Use |
| --- | --- |
| `=`, `<>`, `<`, `>`, `<=`, `>=` | Compare values |
| `AND`, `OR`, `NOT` | Combine conditions. `AND` is applied before `OR`. Use parentheses when you mix them |
| `BETWEEN` | Inclusive range |
| `IN ( ... )` | Match any value in a list |
| `LIKE` | Pattern. `%` is any sequence, `_` is one character. `LIKE` is case-sensitive |
| `ILIKE` | The same patterns, case-insensitive. This is a PostgreSQL extension used in the course |
| `IS NULL`, `IS NOT NULL` | Test for a missing value |

`ORDER BY` accepts several columns. `LIMIT` stops after that many rows. `OFFSET` skips rows. A page of 10 rows, page numbering from 1, uses `LIMIT 10 OFFSET (page_number - 1) * 10`.

`DISTINCT` removes duplicate result rows. A join to a "many" table produces one result row per match, so a product in two categories appears twice unless the query aggregates or uses `DISTINCT`.

### Aggregates, GROUP BY, and HAVING

```sql
SELECT c.category_name,
       COUNT(*) AS product_count,
       AVG(p.price) AS avg_price
FROM categories c
JOIN product_categories pc ON pc.category_id = c.category_id
JOIN products p ON p.product_id = pc.product_id
GROUP BY c.category_id, c.category_name
HAVING AVG(p.price) > 100
ORDER BY avg_price DESC;
```

`COUNT`, `SUM`, `AVG`, `MIN`, and `MAX` compute one value from many rows. Aggregates skip `NULL`, except that `COUNT(*)` counts rows. Every column in the `SELECT` list that is not inside an aggregate belongs in the `GROUP BY`.

### Joins

| Join | Result |
| --- | --- |
| `INNER JOIN` | Only rows with a match on both sides |
| `LEFT JOIN` | Every row from the left table. Unmatched right-hand columns are `NULL` |
| `RIGHT JOIN` | Every row from the right table |
| `FULL OUTER JOIN` | Every row from both tables |
| `CROSS JOIN` | Every combination of the two tables |

```sql
SELECT c.first_name, c.last_name, o.order_id, o.status
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id;
```

The `ON` clause states how the rows match. Table aliases (`c`, `o`) keep multi-table queries readable. Joining three or more tables is a chain of the same pattern: customers to orders, orders to `order_items`, `order_items` to products.

### Common mistake

Using `INNER JOIN` when the question asks for rows with no match, such as categories with no products. An inner join drops those categories. A left join keeps them, and `WHERE p.product_id IS NULL` keeps only the unmatched ones.

---

## 3. Logical Query Order

The clauses are written in one order and applied in another:

```
FROM  →  WHERE  →  GROUP BY  →  HAVING  →  SELECT  →  ORDER BY  →  LIMIT
```

This order is why an aggregate cannot appear in `WHERE`. Grouping has not happened yet, so there is no group to aggregate. Filter rows in `WHERE`. Filter groups in `HAVING`.

```sql
-- Rows first, then groups
SELECT c.category_name, COUNT(*) AS product_count
FROM categories c
JOIN product_categories pc ON pc.category_id = c.category_id
JOIN products p ON p.product_id = pc.product_id
WHERE p.price > 50
GROUP BY c.category_id, c.category_name
HAVING COUNT(*) >= 3;
```

A column alias created in `SELECT` is not available in `WHERE`, because `WHERE` runs first. Repeat the expression, or filter later with `HAVING` when the value is an aggregate.

### Common mistake

`WHERE COUNT(*) > 5`. Move the condition to `HAVING COUNT(*) > 5`.

---

## 4. Week 44 — Joins, Subqueries, Views, and CTEs

This section is the fundamental part of Week 44. Set operations and conditional expressions were marked advanced and are not practised here.

### Multi-table joins and the unmatched-row pattern

```sql
SELECT c.category_name, p.name
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
ORDER BY c.category_name, p.name;
```

Both joins are left joins. A later inner join would drop the unmatched categories that the first left join had kept.

A **self-join** uses the same table twice under two aliases. It answers questions such as "which customers live in the same city?":

```sql
SELECT a.customer_id, b.customer_id, a.city
FROM customers a
JOIN customers b
  ON a.city = b.city
 AND a.customer_id < b.customer_id;
```

`a.customer_id < b.customer_id` keeps each pair once and leaves out a customer matched with themselves.

### Subqueries

A subquery is a query inside another query.

```sql
-- Scalar: one value, compared in WHERE
SELECT name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- IN: the product has at least one category
SELECT name
FROM products
WHERE product_id IN (SELECT product_id FROM product_categories);

-- EXISTS: the product appears on at least one order line
SELECT name
FROM products p
WHERE EXISTS (
    SELECT 1
    FROM order_items oi
    WHERE oi.product_id = p.product_id
);

-- NOT EXISTS: customers with no orders
SELECT c.first_name, c.last_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

A **correlated** subquery refers to a column from the outer query. The `EXISTS` example above is correlated: `oi.product_id = p.product_id` uses the outer `p`. A join is often clearer for the same question. Use the form you can explain.

### Views

A view is a stored `SELECT`. Querying the view runs that `SELECT`.

```sql
CREATE VIEW customer_orders AS
SELECT c.customer_id,
       c.first_name,
       c.last_name,
       o.order_id,
       o.order_date,
       o.status
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;

-- Later
SELECT * FROM customer_orders WHERE status = 'pending';

DROP VIEW customer_orders;
```

`CREATE OR REPLACE VIEW` changes the query when the column list stays compatible. A view of one table, without aggregates or joins, can be updatable. `WITH CHECK OPTION` rejects an update that would make the row fall outside the view's `WHERE`.

### Common table expressions

A CTE names a query with `WITH`, then uses that name. Several CTEs are separated by commas. The result is the same as a subquery in `FROM`, and the name makes the steps easier to read.

```sql
WITH customer_spend AS (
    SELECT o.customer_id,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY o.customer_id
)
SELECT c.first_name, c.last_name, s.total_spent
FROM customer_spend s
JOIN customers c ON c.customer_id = s.customer_id
WHERE s.total_spent > 200
ORDER BY s.total_spent DESC;
```

Revenue uses `quantity * unit_price` from the order line. It does not use today's `products.price`.

### Common mistake

Forgetting that `NOT EXISTS` keeps the outer rows that have no match. An inner join drops those rows before the outer query can test them.

---

## 5. Week 46 — Transactions and Concurrency

### One unit of work

A **transaction** is a group of statements that succeed together or leave the database unchanged. Placing an order is one unit of work: insert the order, insert the lines, and decrease stock.

```sql
BEGIN;

INSERT INTO orders (customer_id)
VALUES (1);

INSERT INTO order_items (order_id, product_id, quantity, unit_price)
VALUES (50, 10, 1, 249.00);

UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10;

COMMIT;
```

`ROLLBACK` undoes every change since `BEGIN`. `SAVEPOINT` marks a point inside the transaction. `ROLLBACK TO SAVEPOINT` undoes only what happened after that mark. The rest of the transaction can still be committed.

```sql
BEGIN;
INSERT INTO orders (customer_id) VALUES (1);
SAVEPOINT order_saved;
UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10;
-- The stock change was the wrong product. Keep the order, undo the update.
ROLLBACK TO SAVEPOINT order_saved;
COMMIT;
```

In a real session the new `order_id` comes from `INSERT ... RETURNING order_id`. The number `50` above stands in for that value.

### ACID

| Property | Meaning | If it fails |
| --- | --- | --- |
| Atomicity | All of the statements are kept, or none are | A crash after the stock update and before `COMMIT` leaves the order unrecorded and the stock unchanged |
| Consistency | The transaction moves the database from one valid state to another. Constraints hold | A line with `quantity` 0 is rejected by `CHECK` |
| Isolation | Concurrent transactions do not see each other's uncommitted work | Two sessions can overwrite each other's stock count |
| Durability | After `COMMIT` returns, the change survives a crash. PostgreSQL writes the change before it confirms the commit | A committed order is still there when the server starts again |

### Concurrency

| Problem | What the second session sees or does |
| --- | --- |
| Dirty read | A row another transaction has changed and not committed. PostgreSQL never allows this |
| Non-repeatable read | The same row read twice in one transaction has different committed values |
| Phantom read | A second read of the same condition returns a new committed row |
| Lost update | Two transactions read a stock value and each write their own result, so one decrease disappears |

**Isolation levels.** `READ COMMITTED` is PostgreSQL's default: a statement sees only data committed before that statement starts. Setting `READ UNCOMMITTED` in PostgreSQL still behaves as `READ COMMITTED`. `REPEATABLE READ` keeps one snapshot for the whole transaction, and in PostgreSQL it also prevents phantom reads. `SERIALIZABLE` is the strictest level.

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- statements
COMMIT;
```

A writer locks the row it changes until `COMMIT` or `ROLLBACK`. `SELECT ... FOR UPDATE` takes that lock when you intend to update the row, which stops a lost update of `stock_quantity`. A plain `SELECT` still reads while a row is locked, because PostgreSQL keeps more than one version of a row. That idea is MVCC: readers do not block writers, and writers do not block readers.

A **deadlock** is two transactions each waiting for a lock the other holds. PostgreSQL aborts one of them. The application retries. Locking rows in a consistent order, and keeping transactions short, avoids the usual case.

### Common mistake

Decreasing stock in a separate autocommit statement from the order insert. A failure between the two statements leaves an order with no stock change, or a stock change with no order.

---

## 6. Week 47 — Indexes and Plans

### What an index is

Without an index, PostgreSQL reads the table from the start until it has checked the relevant rows. That **sequential scan** grows with the number of rows. An **index** is a separate structure that maps values to the rows that hold them. The default index is a **B-tree**. A lookup in a B-tree grows much more slowly than a full read of the table: on the order of a few dozen steps for a million rows, rather than a million comparisons.

```sql
CREATE INDEX idx_orders_customer_id
    ON orders (customer_id);
```

PostgreSQL creates an index for a primary key and for a `UNIQUE` constraint. It does **not** create an index merely because a column is a foreign key. `orders.customer_id` needs the index above if you look up one customer's orders in a large table.

A composite index such as the primary key (`product_id`, `category_id`) supports a lookup that starts with `product_id`. A lookup that starts with `category_id` needs its own index. TrailShop already has that second index: `idx_product_categories_category_id`.

### Reading a plan

`EXPLAIN` shows the plan PostgreSQL expects to use, including the estimated cost and row count. `EXPLAIN ANALYZE` runs the statement and adds the actual time and row count. `EXPLAIN ANALYZE` on `INSERT`, `UPDATE`, or `DELETE` really changes the data.

| Plan line | Meaning |
| --- | --- |
| Seq Scan | The table is read from the start |
| Index Scan | The index finds the matching rows, then the table supplies the other columns |
| Index Only Scan | The index itself contains every column the query needs |

```sql
EXPLAIN ANALYZE
SELECT order_id, order_date
FROM orders
WHERE customer_id = 1;
```

Before the index, a large `orders` table shows a sequential scan with a filter on `customer_id`. After `idx_orders_customer_id`, the plan can show an index scan. On a tiny table PostgreSQL may still choose a sequential scan, because reading the whole table is cheaper than using the index.

### When to index, and when not to

Create an index for columns you filter, join, or order by in large tables, especially foreign keys and highly selective columns (a value that matches few rows).

Skip an index when the table is small, the column matches most of the rows (a yes/no column split evenly), or the table is written constantly and rarely read. Every insert, update, and delete must also update the index. A query that wraps the indexed column in a function, such as `WHERE LOWER(email) = ...`, cannot use an ordinary index on `email`.

### Common mistake

Indexing every column "to be safe". The extra indexes slow down writes, and many of them are never used for a read.

---

## 7. Week 48 — Access, Backups, Injection, and Documents

### Roles and privileges

A **role** is an identity PostgreSQL can grant privileges to. `CREATE USER` is a role with `LOGIN`.

```sql
CREATE ROLE ts_analyst LOGIN PASSWORD 'analyst_pass';

GRANT USAGE ON SCHEMA public TO ts_analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ts_analyst;

REVOKE UPDATE ON products FROM ts_analyst;
```

`USAGE` on the schema lets the role see objects in it. Table privileges include `SELECT`, `INSERT`, `UPDATE`, and `DELETE`. Grant only what the job needs. An analyst who reports on TrailShop receives `SELECT`, not the right to change stock.

`ALTER ROLE` changes a password or an attribute. `DROP ROLE` removes a role that no longer owns objects.

### Backup and restore

`pg_dump` writes a logical backup of one database. Custom format (`-Fc`) is the form to restore with `pg_restore`. `pg_dumpall` includes every database and the roles.

```bash
pg_dump -Fc -f trailshop.dump trailshop
pg_restore -d trailshop_restore trailshop.dump
```

A backup that has never been restored is an untested file. The fundamental practice is: dump, then restore into an empty database and check that the tables are there.

### SQL injection

Injection happens when a program builds a statement by gluing user input into the SQL text:

```text
SELECT * FROM customers WHERE email = 'USER INPUT'
```

If the input is `' OR '1'='1`, the condition becomes true for every row. The defense to learn first is a **parameterized query**: the SQL text stays fixed, and the value is sent separately.

```text
SELECT * FROM customers WHERE email = $1
```

The application binds the email to `$1` (or to `%s` in a Python driver). The value cannot change the shape of the statement. Least privilege limits the damage if a statement is ever abused: the shop role should not be a superuser. An ORM parameterizes its ordinary queries. A raw statement that concatenates input is still unsafe.

### Relational tables and documents

TrailShop's orders, payments, stock, and keys stay in relational tables. The DBMS enforces the foreign keys, the checks, and the transactions.

A document database stores a whole structure, often JSON, in one record. That fits data whose shape varies and is usually read together, such as a product specification sheet that differs for tents and for boots. PostgreSQL can keep that varying sheet in a `JSONB` column on `products` while the price, stock, and keys stay as ordinary columns.

Choose relational tables when you need constraints, joins, and transactions across customers, orders, and stock. Choose a document, or a `JSONB` column, when the extra fields differ from product to product and you mostly fetch the whole sheet with the product.

### Common mistake

Building a login or a search by concatenating the user's text into the SQL string. An order and its payment belong in related tables with foreign keys and `CHECK` constraints.

---

## Key Terms

| Term | Meaning |
| --- | --- |
| DDL / DML | Statements that define tables, and statements that change rows |
| Aggregate | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX` |
| `HAVING` | A filter applied after grouping |
| Inner join / left join | Matches only, or every left-hand row |
| Correlated subquery | A subquery that uses a column from the outer query |
| View | A named `SELECT` |
| CTE | A named subquery introduced with `WITH` |
| Transaction | A unit of work ended with `COMMIT` or `ROLLBACK` |
| ACID | Atomicity, consistency, isolation, durability |
| Isolation level | How much concurrent transactions can see. Default: `READ COMMITTED` |
| Index | A separate structure that maps values to rows |
| Seq Scan / Index Scan | A full read of the table, or a lookup through an index |
| Role | An identity that holds privileges |
| Parameterized query | SQL text with a placeholder, and the value sent separately |
| JSONB | A PostgreSQL column that stores a JSON document |

---

## Summary

- Weeks 40 and 41 are the statements you write most: create and change rows safely, then filter, group, and join. The clause order explains why aggregates belong in `HAVING`.
- Week 44 adds the patterns that cross many tables: outer joins that keep unmatched rows, subqueries, views, and CTEs. Category questions join through `product_categories`. Money questions use `order_items.unit_price`.
- Weeks 46 and 47 cover correctness under concurrency and the cost of a search: transactions and locks, then indexes and `EXPLAIN`.
- Week 48 covers who may do what, how you get the data back, how injection is stopped, and when a document is a better fit than another table.
