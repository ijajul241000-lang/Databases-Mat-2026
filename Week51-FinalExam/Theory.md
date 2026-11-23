# Week 51 — Final Exam

> **Final exam week.** No new course content. The checklist below is the list of fundamentals to be able to explain and write. The optional mock paper is in the Exercises file.

## Epilogue: "The Journey's End"

Exam 1 covered Weeks 36–41. The final exam covers the whole course, including the design review from Week 49 and the SQL review from Week 50. Week 42 was that first exam. Week 43 was a holiday and has no topics of its own.

---

## Learning Objectives

By the end of this chapter you will be able to:

- See which kind of question is which: a short explanation, a SQL statement, or a design task
- Use the week-by-week checklist to find gaps
- Sit the optional mock paper under a time limit

---

## 1. Exam Guidance

### What to expect

Questions fall into three kinds:

| Kind | What you do | Share of the paper |
| --- | --- | --- |
| Short explanation | Define a concept and give an example | about 30% |
| SQL | Write statements against a schema you are given | about 50% |
| Design | Give cardinalities, tables, keys, or a normalization to 3NF | about 20% |

The shares are a guide to how to spend your revision time.

**Short explanations.** Answer in your own words and add one concrete example. "Referential integrity means a foreign key matches an existing key. `orders.customer_id` must be a real `customers.customer_id`."

**SQL.** You are given table and column names. Start from the clause order `SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY`. Use the names in the question. TrailShop questions use `products.name`, `stock_quantity`, and `product_categories`. Revenue is `quantity * unit_price`.

**Design.** State both directions of each cardinality. Put a 1:N foreign key on the "many" side. Give every M:N a junction table. Then check 3NF: each non-key column depends on the whole primary key and on nothing else.

### What this exam does not focus on

These were marked advanced in the weekly chapters, or they sit outside the course. Do not spend revision time on them:

- Set operations (`UNION`, `INTERSECT`, `EXCEPT`), `CASE`, `COALESCE`, `NULLIF`, `GREATEST`, and `LEAST`
- Recursive CTEs, materialized views, and window functions such as `RANK` and `ROW_NUMBER`
- Join algorithms, index types other than the B-tree from `CREATE INDEX`, and point-in-time recovery
- Column-level `GRANT`, `WITH GRANT OPTION`, and Boyce–Codd normal form
- Application code, ORM syntax, cloud-provider settings, and the wording of PostgreSQL error messages

You should still be able to say what a parameterized query does, and that an ORM does not make a concatenated raw statement safe. You are not asked to write ORM code.

---

## 2. Study Checklist

If you cannot explain an item and, where it asks for SQL, write it, reread that week's theory and the matching part of Week 49 or Week 50.

### Week 36 — Introduction

- [ ] Data is a raw fact. Information is that fact with a meaning.
- [ ] Five file-system problems: redundancy, isolation, integrity, security, concurrency.
- [ ] A database is the shared collection. A DBMS is the software that manages it.
- [ ] Three levels: external, conceptual, internal. Logical and physical data independence.

### Week 37 — The relational model

- [ ] Relation, tuple, attribute, domain.
- [ ] Candidate key, primary key, foreign key, composite key, surrogate and natural keys.
- [ ] `NULL` is an unknown value. The test is `IS NULL`.
- [ ] Entity integrity and referential integrity.
- [ ] "Cardinality" as the number of rows, distinct from relationship cardinality in Week 38.

### Week 38 — Conceptual modelling

- [ ] Entity, attribute, relationship. Strong and weak entities.
- [ ] Composite attributes are split. Multi-valued attributes become another entity.
- [ ] 1:1, 1:N, and M:N, checked in both directions. Optional and mandatory participation.
- [ ] M:N becomes a junction. Category–Product is `product_categories`. `products` has no `category_id`.
- [ ] A foreign key forces each child to have a parent. It does not force each parent to have a child.

### Week 39 — Logical design

- [ ] 1:N: foreign key on the "many" side, `NOT NULL` when that side is mandatory.
- [ ] 1:1: foreign key plus `UNIQUE` (`payments.order_id`).
- [ ] Types: `NUMERIC` for money, `VARCHAR` and `TEXT`, `INTEGER`, `TIMESTAMPTZ`, `SERIAL`.
- [ ] `PRIMARY KEY`, `NOT NULL`, `UNIQUE`, `CHECK`, `DEFAULT`, `REFERENCES`, `ON DELETE`, `ON UPDATE`.
- [ ] Create parent tables before the tables that reference them.

### Week 40 — SQL fundamentals

- [ ] `CREATE TABLE` with the constraints above.
- [ ] `INSERT` of one row and of several rows. `UPDATE` and `DELETE` with `WHERE`.
- [ ] A basic `SELECT`. String literals use single quotes.

### Week 41 — Querying

- [ ] `WHERE` with comparisons, `AND` / `OR` / `NOT`, `BETWEEN`, `IN`, `LIKE`, `ILIKE`, `IS NULL`.
- [ ] `ORDER BY`, `LIMIT`, `OFFSET`.
- [ ] `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`. Aggregates skip `NULL`. `COUNT(*)` counts rows.
- [ ] `GROUP BY`. Every non-aggregated selected column is in the `GROUP BY`.
- [ ] `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, and `CROSS` joins. The `ON` clause.
- [ ] Logical order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`.

### Week 44 — Further SQL, fundamentals only

- [ ] A join of three or more tables, including the path through `product_categories`.
- [ ] A self-join with two aliases.
- [ ] A left join that keeps unmatched rows, and `IS NULL` to keep only those rows.
- [ ] Scalar subquery, `IN`, `EXISTS`, `NOT EXISTS`, and a correlated subquery.
- [ ] `CREATE VIEW` and `DROP VIEW`.
- [ ] A CTE with `WITH`, including one CTE used by the main query.

### Week 45 — Normalization

- [ ] Insertion, update, and deletion anomalies.
- [ ] 1NF: atomic values, no repeating groups, a primary key.
- [ ] 2NF: no partial dependency. Partial dependencies need a composite key.
- [ ] 3NF: no transitive dependency.
- [ ] `order_items.unit_price` is the price charged on that order, not a copy of `products.price`.
- [ ] Reach 3NF first. A later, measured exception is a decision you can explain.

### Week 46 — Transactions

- [ ] `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `ROLLBACK TO SAVEPOINT`.
- [ ] ACID, including that a committed change is written before the commit is confirmed.
- [ ] A single statement is already its own transaction.
- [ ] Dirty read, non-repeatable read, phantom read, lost update. PostgreSQL does not allow a dirty read.
- [ ] `READ COMMITTED` is the default. `READ UNCOMMITTED` behaves as `READ COMMITTED`. PostgreSQL's `REPEATABLE READ` also prevents phantoms.
- [ ] `SELECT ... FOR UPDATE` locks the row you are about to change. A deadlock is two transactions waiting on each other; PostgreSQL aborts one.

### Week 47 — Indexes

- [ ] A sequential scan reads the table. An index maps values to rows. The index you create is a B-tree.
- [ ] `CREATE INDEX`. Primary keys and `UNIQUE` constraints are indexed. Foreign keys are not.
- [ ] A composite index serves a lookup that starts with its leftmost column.
- [ ] `EXPLAIN` shows the estimated plan. `EXPLAIN ANALYZE` runs the statement.
- [ ] Seq Scan, Index Scan, and Index Only Scan.
- [ ] Skip an index on a small table, a column that matches most rows, or a table that is written constantly and rarely read.

### Week 48 — Administration, security, and documents

- [ ] `CREATE ROLE`, `GRANT`, `REVOKE`. Grant only what the job needs. `USAGE` on the schema, then the table privilege.
- [ ] `pg_dump` and `pg_restore` for a logical backup. Restore once to prove the file works.
- [ ] SQL injection comes from concatenating input into the statement text. A parameterized query sends the value separately.
- [ ] Orders, payments, stock, and keys stay in relational tables. A varying specification sheet can be a document or a `JSONB` column.

---

## 3. A 15-minute self-check

Pick one item you marked as shaky and do it on paper, without the theory open.

1. Draw Customer, Order, Product, and Category with cardinalities. Mark `product_categories` and `order_items`.
2. Write the query for "categories with no products".
3. Normalize one flat table as far as 3NF, naming each dependency you remove.
4. Write the `BEGIN` block for an order and a stock decrease, and say what a crash before `COMMIT` leaves behind.

The Exercises file is a full mock paper if you want a timed run. Ninety minutes is a reasonable length.

---

## 4. After the Course

Window functions, stored procedures, triggers, and ORMs are useful next steps. They are not part of this exam. The PostgreSQL tutorial and the SQL reference you have used all term are the place to start when you want them.

---

## Summary

- Revise from the checklist, week by week. Weeks 49 and 50 already collect the same fundamentals in two shorter chapters.
- In a design answer, show both directions of the cardinality, the foreign key or the junction, and a 3NF check.
- In a SQL answer, use the column names you are given, filter groups with `HAVING`, and keep unmatched rows with a left join or `NOT EXISTS`.
- In a transaction or index answer, name the property or the plan line, then the statement that produces it.
