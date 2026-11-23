# Week 51 — Final Exam — Answers

> [!IMPORTANT]
> **_Model Answers_**
> Answers for the Week 51 mock paper. Each question is repeated, and the model answer is in the same answer field. Write your own answer in `Exercises.md` before you read this file.

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
> `12` is data: a raw fact. It becomes information when you know what it measures, for example "12 units of the Ridge Tent are in stock".
>
> One file problem is enough. Isolation: orders and products in separate files cannot be combined without custom matching, while a DBMS stores both as related tables and answers the question with a query. Redundancy, integrity, security, or concurrency, each with a consequence and the DBMS response, is equally acceptable.

### A2. Keys and integrity (Week 37)

Explain candidate key, primary key, and foreign key using `customers` and `orders`. State entity integrity. Explain why `orders.customer_id` may not contain a customer id that is missing from `customers`.

> [!NOTE]
> ***Your Answer***
>
> `customer_id` and `email` are both candidate keys of `customers`: each identifies a row, and neither needs the other column to do it. The primary key is the candidate key that was chosen: `customer_id`. `orders.customer_id` is a foreign key because its values refer to `customers.customer_id`.
>
> Entity integrity: a primary key is unique and no part of it may be `NULL`.
>
> Referential integrity: `orders.customer_id` must equal an existing `customers.customer_id`. In this schema the column is also `NOT NULL`, so every order has a customer.

### A3. Cardinality and mapping (Weeks 38–39)

For each relationship, give the cardinality and say where the foreign key goes, or name the junction table.

1. A customer places orders. Each order belongs to one customer. A customer may have placed none.
2. A product belongs to any number of categories, and a category contains any number of products.
3. Each order has at most one payment, and each payment belongs to one order.

Also choose a PostgreSQL type for `products.price` and say why a floating-point type is a poor choice for money.

> [!NOTE]
> ***Your Answer***
>
> 1. **1:N** from customer to order. `orders.customer_id` references `customers` and is `NOT NULL`. The customer side is optional.
> 2. **M:N.** Junction table `product_categories(product_id, category_id)`. There is no `category_id` on `products`.
> 3. **1:1.** `payments.order_id` references `orders` and is `UNIQUE`.
>
> `products.price` is `NUMERIC(10,2)`. A floating-point type cannot represent every decimal exactly, so money rounding becomes unreliable.

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
> 1. Changing a product name means updating every order line for that product. A missed row leaves two names for one product. An update anomaly about the repeated email is also valid.
> 2. `order_date`, `customer_id`, and `customer_email` depend only on `order_id`. `product_name` depends only on `product_id`. `quantity` depends on the whole key.
> 3. Removing the partial dependencies gives 2NF:
>
> ```
> orders(order_id PK, order_date, customer_id, customer_email)
> products(product_id PK, product_name)
> order_items(order_id PK/FK, product_id PK/FK, quantity)
> ```
>
> `customer_email` still depends on `customer_id`, which is not the primary key of `orders`. Removing that transitive dependency gives 3NF:
>
> ```
> customers(customer_id PK, customer_email)
> orders(order_id PK, order_date, customer_id FK)
> products(product_id PK, product_name)
> order_items(order_id PK/FK, product_id PK/FK, quantity)
> ```
>
> 4. `unit_price` is the price charged on that order. It depends on the order line. `products.price` is the current catalog price. They are different facts, so `unit_price` stays on `order_items`.

### A5. ACID (Week 46)

A transaction inserts an order and then decreases `stock_quantity`. Define atomicity and durability for this transaction. What remains in the database if the server crashes after the stock update and before `COMMIT`?

> [!NOTE]
> ***Your Answer***
>
> Atomicity: both changes are kept, or neither is. Durability: after `COMMIT` returns, both changes survive a later crash, because PostgreSQL writes them before it confirms the commit.
>
> A crash before `COMMIT` means the transaction never committed. Recovery undoes it. The new order is gone and the stock is unchanged.

### A6. Indexes (Week 47)

Explain what an index is. Then describe one situation where adding an index makes the database slower for the work it actually does.

> [!NOTE]
> ***Your Answer***
>
> An index is a separate structure, by default a B-tree, that maps column values to the rows that contain them. A lookup can use the index instead of reading the whole table.
>
> An index slows a table that is inserted or updated constantly and rarely searched, such as a log of every page view. Every write must also update the index. A small table, or a column that matches most of the rows, is another valid case: the index costs writes and the planner still reads the table.

### A7. Injection and privileges (Week 48)

The shop builds a statement by joining the typed product name into the SQL text. Explain the risk. State the defense you would use first. State one privilege the public shop role should not have.

> [!NOTE]
> ***Your Answer***
>
> The typed name becomes part of the statement, so the user can change the query, for example by closing the quote and adding further SQL. The first defense is a parameterized query: the statement text is fixed and the name is sent as a bound value.
>
> The shop role should not be a superuser, and it should not have `DROP` or unrestricted `DELETE` on every table. `SELECT` on the catalog and `INSERT` on orders and order lines is the kind of grant that matches the job.

### A8. Relational data and documents (Week 48)

TrailShop will store, for each product, a specification whose fields differ: tents have a capacity, boots have a size run. Orders and stock do not change shape.

Say where you would store the specification, and where you would store the order and the stock. Give one reason for each.

> [!NOTE]
> ***Your Answer***
>
> The specification fits a `JSONB` column on `products`, or a document stored with the product. The fields differ by product, and the page reads the whole sheet at once.
>
> The order, its lines, and `stock_quantity` stay in relational tables. They need foreign keys, checks such as `quantity > 0`, and a transaction that records the sale and decreases stock together.

---

## Section B — SQL

### B1. Filter and sort

List `name`, `price`, and `stock_quantity` for products that have fewer than 5 units in stock and whose name contains `tent`, in any letter case. Sort by `stock_quantity` ascending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT name, price, stock_quantity
> FROM products
> WHERE stock_quantity < 5
>   AND name ILIKE '%tent%'
> ORDER BY stock_quantity;
> ```

### B2. Revenue by category

Show `category_name` and the total revenue of that category, using `SUM(quantity * unit_price)` from order lines. Include a product's revenue in each of its categories. Keep only categories with revenue above €1000. Sort by revenue descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name,
>        SUM(oi.quantity * oi.unit_price) AS revenue
> FROM categories c
> JOIN product_categories pc ON pc.category_id = c.category_id
> JOIN products p ON p.product_id = pc.product_id
> JOIN order_items oi ON oi.product_id = p.product_id
> GROUP BY c.category_id, c.category_name
> HAVING SUM(oi.quantity * oi.unit_price) > 1000
> ORDER BY revenue DESC;
> ```
>
> The path runs through `product_categories` because a product has no `category_id`. A product in two categories contributes its line revenue to both, which is what "each of its categories" asks for.

### B3. Products nobody has ordered

List `product_id` and `name` for products that do not appear in `order_items`. Use `NOT EXISTS` or a `LEFT JOIN`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT p.product_id, p.name
> FROM products p
> WHERE NOT EXISTS (
>     SELECT 1
>     FROM order_items oi
>     WHERE oi.product_id = p.product_id
> );
> ```
>
> The left-join form is:
>
> ```sql
> SELECT p.product_id, p.name
> FROM products p
> LEFT JOIN order_items oi ON oi.product_id = p.product_id
> WHERE oi.product_id IS NULL;
> ```

### B4. Create a table

Create `wishlists`. A row means one customer has saved one product. The same customer cannot save the same product twice. Record when it was saved. Choose foreign keys and `ON DELETE` actions you can justify in a comment.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE TABLE wishlists (
>     customer_id INTEGER     NOT NULL
>                 REFERENCES customers(customer_id)
>                 ON DELETE CASCADE
>                 ON UPDATE CASCADE,
>     product_id  INTEGER     NOT NULL
>                 REFERENCES products(product_id)
>                 ON DELETE CASCADE
>                 ON UPDATE CASCADE,
>     added_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
>     PRIMARY KEY (customer_id, product_id)
> );
> ```
>
> The composite primary key stops a duplicate pair. `ON DELETE CASCADE` removes a saved row when the customer or the product is deleted, because the wish cannot refer to a missing person or product. `ON DELETE RESTRICT` on `product_id` is a defensible alternative when a wish should block deletion of the product.

### B5. A transaction

Write one transaction that records an order for customer `1` containing two units of product `10` at a unit price of `249.00`, and decreases that product's stock by 2. Commit only when the stock update is acceptable. If you need to undo the work, use `ROLLBACK`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> BEGIN;
>
> INSERT INTO orders (customer_id)
> VALUES (1);
>
> -- In a live session, use RETURNING order_id and substitute that value.
> INSERT INTO order_items (order_id, product_id, quantity, unit_price)
> VALUES (50, 10, 2, 249.00);
>
> UPDATE products
> SET stock_quantity = stock_quantity - 2
> WHERE product_id = 10
>   AND stock_quantity >= 2;
>
> -- If the UPDATE changed 0 rows, the stock was too low:
> -- ROLLBACK;
> COMMIT;
> ```
>
> `SELECT stock_quantity FROM products WHERE product_id = 10 FOR UPDATE` before the update is a stronger form: it locks the product row so a second checkout cannot lose a stock decrease. Either the `WHERE stock_quantity >= 2` guard or `FOR UPDATE` shows the idea. All of the statements sit in one `BEGIN` block so a `ROLLBACK` removes the order as well as the stock change.

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
> The table is already in 1NF: each cell is atomic and the composite primary key is stated.
>
> **Partial dependencies (remove these to reach 2NF).** `course_title`, `instructor_id`, and `instructor_name` depend only on `course_id`. `student_name` depends only on `student_id`. `grade` depends on the whole key.
>
> ```
> courses(course_id PK, course_title, instructor_id, instructor_name)
> students(student_id PK, student_name)
> enrolments(course_id PK/FK, student_id PK/FK, grade)
> ```
>
> **Transitive dependency (remove this to reach 3NF).** On `courses`, `instructor_name` depends on `instructor_id`, which is not the primary key.
>
> ```
> instructors(instructor_id PK, instructor_name)
> courses(course_id PK, course_title, instructor_id FK)
> students(student_id PK, student_name)
> enrolments(course_id PK/FK, student_id PK/FK, grade)
> ```

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
> Member–Hike is **M:N**, so the role lives on a junction table.
>
> ```sql
> CREATE TABLE members (
>     member_id   SERIAL       PRIMARY KEY,
>     member_name VARCHAR(100) NOT NULL,
>     email       VARCHAR(254) NOT NULL UNIQUE
> );
>
> CREATE TABLE hikes (
>     hike_id     SERIAL        PRIMARY KEY,
>     title       VARCHAR(200)  NOT NULL,
>     hike_date   DATE          NOT NULL,
>     distance_km NUMERIC(5,1)  NOT NULL CHECK (distance_km > 0)
> );
>
> CREATE TABLE participations (
>     member_id INTEGER     NOT NULL
>               REFERENCES members(member_id)
>               ON DELETE CASCADE,
>     hike_id   INTEGER     NOT NULL
>               REFERENCES hikes(hike_id)
>               ON DELETE CASCADE,
>     role      VARCHAR(10) NOT NULL
>               CHECK (role IN ('leader', 'walker')),
>     PRIMARY KEY (member_id, hike_id)
> );
> ```
>
> The primary key of `participations` stops the same member being recorded twice on the same hike.

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
> Nothing from this transaction is stored. The insert and the stock update are undone because the transaction did not commit. That is atomicity.
>
> `SELECT ... FOR UPDATE` on the product row locks it until Session A commits or rolls back. Session B cannot update that stock on a stale value, so one of the two decreases is not lost. The lock is the defense against a lost update.

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
> CREATE INDEX idx_order_items_product_id
>     ON order_items (product_id);
> ```

> [!NOTE]
> ***Your Answer***
>
> The primary-key index starts with `order_id`. A lookup that only knows `product_id` cannot use that leftmost column, so the planner reads the table. The new index can turn the plan into an index scan.
>
> Every insert, update, and delete of `order_items` must also maintain the index, so writes become more expensive. That trade is worth it when this lookup is common and the table is large.
