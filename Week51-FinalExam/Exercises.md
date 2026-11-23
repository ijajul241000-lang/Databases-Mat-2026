# Week 51 — Final Exam — Exercises (Optional)

> [!IMPORTANT]
> **_Exam Preparation_**
> **No submission is required.** This is one mock paper. Close the theory, set a timer for 90 minutes, and write your answers before you open `Answers.md`.

The paper uses the current TrailShop schema.

```
categories(category_id PK, category_name, description)
customers(customer_id PK, first_name, last_name, email, phone,
          street, city, postal_code, country, registered_at)
products(product_id PK, name, description, price, weight_kg,
         stock_quantity, created_at)
product_categories(product_id PK/FK, category_id PK/FK)
orders(order_id PK, customer_id FK, order_date, status)
order_items(order_id PK/FK, product_id PK/FK, quantity, unit_price)
payments(payment_id PK, order_id UNIQUE FK, amount, payment_method, payment_date)
```

`products` has no `category_id`. A line total is `quantity * unit_price`.

---

## Section A — Short Explanations

### A1. Data, information, and files (Week 36)

TrailShop's old `inventory.xlsx` contains the cell `12`. Explain the difference between data and information using that cell. Then name one problem of keeping orders in a separate file from products, and what a DBMS does about that problem.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A2. Keys and integrity (Week 37)

Explain candidate key, primary key, and foreign key using `customers` and `orders`. State entity integrity. Explain why `orders.customer_id` may not contain a customer id that is missing from `customers`.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A3. Cardinality and mapping (Weeks 38–39)

For each relationship, give the cardinality and say where the foreign key goes, or name the junction table.

1. A customer places orders. Each order belongs to one customer. A customer may have placed none.
2. A product belongs to any number of categories, and a category contains any number of products.
3. Each order has at most one payment, and each payment belongs to one order.

Also choose a PostgreSQL type for `products.price` and say why a floating-point type is a poor choice for money.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A4. Anomalies and normal form (Week 45)

This table has primary key (`order_id`, `product_id`):

```
order_id, order_date, customer_id, customer_email, product_id, product_name, quantity
```

1. Give one update anomaly.
2. Name the partial dependencies.
3. Show the tables in 2NF and then in 3NF. `customer_email` belongs to the customer, not to the order.
4. The real `order_items` table also has `unit_price`. Why is that column still in 3NF?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A5. ACID (Week 46)

A transaction inserts an order and then decreases `stock_quantity`. Define atomicity and durability for this transaction. What remains in the database if the server crashes after the stock update and before `COMMIT`?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A6. Indexes (Week 47)

Explain what an index is. Then describe one situation where adding an index makes the database slower for the work it actually does.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A7. Injection and privileges (Week 48)

The shop builds a statement by joining the typed product name into the SQL text. Explain the risk. State the defense you would use first. State one privilege the public shop role should not have.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### A8. Relational data and documents (Week 48)

TrailShop will store, for each product, a specification whose fields differ: tents have a capacity, boots have a size run. Orders and stock do not change shape.

Say where you would store the specification, and where you would store the order and the stock. Give one reason for each.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Section B — SQL

### B1. Filter and sort

List `name`, `price`, and `stock_quantity` for products that have fewer than 5 units in stock and whose name contains `tent`, in any letter case. Sort by `stock_quantity` ascending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### B2. Revenue by category

Show `category_name` and the total revenue of that category, using `SUM(quantity * unit_price)` from order lines. Include a product's revenue in each of its categories. Keep only categories with revenue above €1000. Sort by revenue descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### B3. Products nobody has ordered

List `product_id` and `name` for products that do not appear in `order_items`. Use `NOT EXISTS` or a `LEFT JOIN`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### B4. Create a table

Create `wishlists`. A row means one customer has saved one product. The same customer cannot save the same product twice. Record when it was saved. Choose foreign keys and `ON DELETE` actions you can justify in a comment.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### B5. A transaction

Write one transaction that records an order for customer `1` containing two units of product `10` at a unit price of `249.00`, and decreases that product's stock by 2. Commit only when the stock update is acceptable. If you need to undo the work, use `ROLLBACK`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Section C — Design and Normalization

### C1. Normalize to 3NF

A course register stores one row per student per course:

| course_id | course_title | instructor_id | instructor_name | student_id | student_name | grade |
| --- | --- | --- | --- | --- | --- | --- |
| 4 | Navigation | 9 | Liisa Korhonen | 21 | Anna Virtanen | 4 |
| 4 | Navigation | 9 | Liisa Korhonen | 22 | Mikko Lehto | 5 |
| 7 | First Aid | 9 | Liisa Korhonen | 21 | Anna Virtanen | 3 |

The primary key is (`course_id`, `student_id`).

- A course has one title and one instructor.
- An instructor has one name and can teach many courses.
- A student has one name and can take many courses.
- `grade` depends on this student taking this course.

Show the tables after 2NF and the tables after 3NF. Name the dependency you remove at each step.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### C2. A small schema

A trail club records members and hikes.

- A member has a name and a unique email.
- A hike has a title, a date, and a distance in kilometres.
- A member can join many hikes, and a hike has many members.
- For each member on a hike, store a role: `leader` or `walker`.

List the tables, primary keys, foreign keys, and the cardinality of Member–Hike. Add the constraint that matches the two roles.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Section D — Scenarios

### D1. A crash during checkout (Week 46)

Session A runs:

```sql
BEGIN;
INSERT INTO orders (customer_id) VALUES (1);
UPDATE products
SET stock_quantity = stock_quantity - 1
WHERE product_id = 10;
-- the server crashes here, before COMMIT
```

What is stored after PostgreSQL recovers? Name the ACID property. Then say what `SELECT ... FOR UPDATE` adds if a second session is selling the same product at the same time.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### D2. A slow lookup (Week 47)

`EXPLAIN ANALYZE` on this query shows a `Seq Scan` of `order_items`, about five million rows, with a filter on `product_id`:

```sql
SELECT order_id, quantity
FROM order_items
WHERE product_id = 10;
```

The primary key of `order_items` is (`order_id`, `product_id`).

Propose an index, write the statement, and say why the primary-key index does not serve this query. Name one cost the new index adds.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## If a Section Went Badly

Use this only to choose what to reread. The full checklist is in the Week 51 theory.

| If this was weak | Reread |
| --- | --- |
| A1 | Week 49 theory, Section 1 (Week 36) |
| A2, A3 | Week 49 theory, Sections 2–4 (Weeks 37–39) |
| A4, C1, C2 | Week 49 theory, Sections 5–7 (Week 45 and the checklist), and Exercise 3 |
| B1–B3 | Week 50 theory, Sections 2–4, and Exercises 1.1–1.7 |
| B4 | Week 50 theory, Section 1, and the SoundWave `CREATE TABLE` in Week 49 |
| B5, A5, D1 | Week 50 theory, Section 5, and Exercise 1.10 |
| A6, D2 | Week 50 theory, Section 6, and Exercise 1.11 |
| A7, A8 | Week 50 theory, Section 7, and Exercises 3.2–3.3 |
