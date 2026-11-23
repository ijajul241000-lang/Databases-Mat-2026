# Week 50 — Final Review: SQL — Answers

> [!IMPORTANT]
> **_Model Answers_**
> Answers for the Week 50 exercises. Each question is repeated, and the model answer is in the same answer field. Write your own answer in `Exercises.md` before you read this file.

You can run the SQL against your TrailShop database. Read-only practice is enough for the queries. Run the `UPDATE`, transaction, index, and role statements only on a database you are willing to change, and roll back anything you do not want to keep.

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

`products` has no `category_id`. Revenue is `quantity * unit_price` on the order line.

---

## 1. Progressive SQL

### 1.1 Filter, sort, and limit

List the `name` and `price` of products priced from €50 to €150, highest price first. Return at most 10 rows.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT name, price
> FROM products
> WHERE price BETWEEN 50 AND 150
> ORDER BY price DESC
> LIMIT 10;
> ```
>
> `BETWEEN` includes both ends.

### 1.2 Group and filter the groups

For each category, show `category_name`, the number of products, and the average price. Keep categories whose average price is above €100. Sort by average price, highest first. Join through `product_categories`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name,
>        COUNT(*) AS product_count,
>        AVG(p.price) AS avg_price
> FROM categories c
> JOIN product_categories pc ON pc.category_id = c.category_id
> JOIN products p ON p.product_id = pc.product_id
> GROUP BY c.category_id, c.category_name
> HAVING AVG(p.price) > 100
> ORDER BY avg_price DESC;
> ```
>
> `WHERE` cannot hold `AVG`. The average exists only after `GROUP BY`, so the test belongs in `HAVING`.

### 1.3 Inner join

List each order's `order_id`, `status`, and the customer's `first_name` and `last_name`. Include only orders that have a customer, which in this schema is every order.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT o.order_id, o.status, c.first_name, c.last_name
> FROM orders o
> JOIN customers c ON c.customer_id = o.customer_id
> ORDER BY o.order_id;
> ```

### 1.4 Left join, including rows with no match

List every category name and the names of its products. Include categories that have no products. Those rows should show a null product name.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name, p.name AS product_name
> FROM categories c
> LEFT JOIN product_categories pc ON pc.category_id = c.category_id
> LEFT JOIN products p ON p.product_id = pc.product_id
> ORDER BY c.category_name, p.name;
> ```
>
> Both joins stay left joins. An inner join to `products` would drop the empty categories.

### 1.5 Customers with no orders

List `customer_id`, `first_name`, and `last_name` for customers who have never ordered. Use a left join.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.customer_id, c.first_name, c.last_name
> FROM customers c
> LEFT JOIN orders o ON o.customer_id = c.customer_id
> WHERE o.order_id IS NULL
> ORDER BY c.customer_id;
> ```

### 1.6 Three or more tables

For every order line, show the customer's last name, the order id, the product name, the quantity, and the line total (`quantity * unit_price`).

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.last_name,
>        o.order_id,
>        p.name AS product_name,
>        oi.quantity,
>        oi.quantity * oi.unit_price AS line_total
> FROM customers c
> JOIN orders o ON o.customer_id = c.customer_id
> JOIN order_items oi ON oi.order_id = o.order_id
> JOIN products p ON p.product_id = oi.product_id
> ORDER BY o.order_id, p.name;
> ```

### 1.7 A subquery

List the `name` and `price` of products that appear on at least one order line. Use `EXISTS` or `IN`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT p.name, p.price
> FROM products p
> WHERE EXISTS (
>     SELECT 1
>     FROM order_items oi
>     WHERE oi.product_id = p.product_id
> )
> ORDER BY p.name;
> ```
>
> `IN` is equally valid: `WHERE product_id IN (SELECT product_id FROM order_items)`.

### 1.8 A CTE

Using `WITH`, compute each customer's total spending from order lines, then return `first_name`, `last_name`, and `total_spent` for customers who have spent more than €200. Sort by `total_spent` descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> WITH customer_spend AS (
>     SELECT o.customer_id,
>            SUM(oi.quantity * oi.unit_price) AS total_spent
>     FROM orders o
>     JOIN order_items oi ON oi.order_id = o.order_id
>     GROUP BY o.customer_id
> )
> SELECT c.first_name, c.last_name, s.total_spent
> FROM customer_spend s
> JOIN customers c ON c.customer_id = s.customer_id
> WHERE s.total_spent > 200
> ORDER BY s.total_spent DESC;
> ```

### 1.9 A view

Create a view `pending_orders` with `order_id`, `order_date`, `customer_id`, `first_name`, and `last_name` for orders whose status is `'pending'`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE VIEW pending_orders AS
> SELECT o.order_id,
>        o.order_date,
>        o.customer_id,
>        c.first_name,
>        c.last_name
> FROM orders o
> JOIN customers c ON c.customer_id = o.customer_id
> WHERE o.status = 'pending';
> ```

### 1.10 A transaction with a savepoint

Write a transaction that:

1. Inserts an order for customer 1.
2. Sets a savepoint.
3. Decreases `stock_quantity` by 1 for product 10.
4. Rolls back to the savepoint, undoing the stock change and keeping the new order.
5. Commits.

Use `50` as the new `order_id` if you insert an order line, or omit the line and only insert the order. Say in a comment which statements survive the commit.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> BEGIN;
>
> INSERT INTO orders (customer_id)
> VALUES (1);
>
> SAVEPOINT order_saved;
>
> UPDATE products
> SET stock_quantity = stock_quantity - 1
> WHERE product_id = 10;
>
> -- Undo the stock change. The INSERT of the order remains.
> ROLLBACK TO SAVEPOINT order_saved;
>
> COMMIT;
> ```
>
> After `COMMIT`, the new order is stored and product 10's stock is unchanged. `ROLLBACK` without `TO SAVEPOINT` would have removed the order as well.

### 1.11 An index and the plan

Orders are looked up by `customer_id`, and `orders` is large. Write `CREATE INDEX` for that lookup. Then say what `EXPLAIN` is likely to show before the index and after it, and name one reason you would not add this index on a table of a few dozen rows.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> CREATE INDEX idx_orders_customer_id
>     ON orders (customer_id);
>
> EXPLAIN
> SELECT order_id, order_date
> FROM orders
> WHERE customer_id = 1;
> ```

> [!NOTE]
> ***Your Answer***
>
> A foreign key is not indexed automatically. On a large table, `EXPLAIN` before the index typically shows a `Seq Scan` with a filter on `customer_id`. After the index, the plan can show an `Index Scan`. On a few dozen rows, reading the whole table can be cheaper than using the index, so PostgreSQL may keep the sequential scan, and the index would still add work to every insert.

---

## 2. Find the Error

For each statement, name the error and write a corrected statement. Assume the author wanted the result described in the heading.

### 2.1 Products with no description

```sql
SELECT product_id, name
FROM products
WHERE description = NULL;
```

> [!NOTE]
> ***Your Answer***
>
> `NULL` is not equal to anything, so `= NULL` matches no rows.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT product_id, name
> FROM products
> WHERE description IS NULL;
> ```

### 2.2 Product count per category, showing the category name

```sql
SELECT c.category_name, p.name, COUNT(*) AS product_count
FROM categories c
JOIN product_categories pc ON pc.category_id = c.category_id
JOIN products p ON p.product_id = pc.product_id
GROUP BY c.category_name;
```

> [!NOTE]
> ***Your Answer***
>
> `p.name` is in the select list and is not aggregated, so it must appear in `GROUP BY`, or it must leave the select list. A count of products per category does not need the product name.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name, COUNT(*) AS product_count
> FROM categories c
> JOIN product_categories pc ON pc.category_id = c.category_id
> JOIN products p ON p.product_id = pc.product_id
> GROUP BY c.category_id, c.category_name;
> ```

### 2.3 Categories with more than five products

```sql
SELECT c.category_name
FROM categories c
JOIN product_categories pc ON pc.category_id = c.category_id
WHERE COUNT(*) > 5
GROUP BY c.category_name;
```

> [!NOTE]
> ***Your Answer***
>
> `WHERE` runs before grouping, so it cannot contain an aggregate.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name
> FROM categories c
> JOIN product_categories pc ON pc.category_id = c.category_id
> GROUP BY c.category_id, c.category_name
> HAVING COUNT(*) > 5;
> ```

### 2.4 Raise the price of product 10 by 10 percent

```sql
UPDATE products
SET price = price * 1.10;
```

> [!NOTE]
> ***Your Answer***
>
> There is no `WHERE` clause, so every product's price changes.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> UPDATE products
> SET price = price * 1.10
> WHERE product_id = 10;
> ```

### 2.5 Categories with no products

```sql
SELECT c.category_name
FROM categories c
JOIN product_categories pc ON pc.category_id = c.category_id
WHERE pc.product_id IS NULL;
```

> [!NOTE]
> ***Your Answer***
>
> `JOIN` is an inner join. Categories with no row in `product_categories` are dropped before `WHERE` can see them, so the `IS NULL` test never succeeds.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> SELECT c.category_name
> FROM categories c
> LEFT JOIN product_categories pc ON pc.category_id = c.category_id
> WHERE pc.product_id IS NULL;
> ```

---

## 3. Short Written Questions

### 3.1 ACID

A checkout inserts an order, inserts two order lines, and decreases stock. Explain each ACID property for that checkout. In the durability sentence, state when the change is safe from a crash.

> [!NOTE]
> ***Your Answer***
>
> - **Atomicity:** the order, both lines, and both stock decreases are stored together, or none of them are. A failure before `COMMIT` undoes the whole checkout.
> - **Consistency:** the resulting rows satisfy the constraints, including positive quantities and a real `customer_id`.
> - **Isolation:** another session does not see this checkout's rows before they are committed, and `SELECT ... FOR UPDATE` on the product rows stops a second checkout from losing a stock decrease.
> - **Durability:** once `COMMIT` has returned, the checkout survives a crash, because PostgreSQL writes the change before it confirms the commit.

### 3.2 Injection and roles

A shop page builds this statement by pasting the typed email into the SQL text:

```text
SELECT customer_id FROM customers WHERE email = '<typed email>'
```

Explain how this can be abused, what a parameterized query does instead, and why the database role used by the shop should not own every privilege.

> [!NOTE]
> ***Your Answer***
>
> The typed text becomes part of the statement. Input such as `' OR '1'='1` can make the condition true for every customer, or extend the statement. A parameterized query keeps the SQL fixed (`WHERE email = $1`) and sends the email as a value, so the value cannot add clauses.
>
> The shop role should receive only the privileges the page needs, for example `SELECT` on `customers` and `INSERT` on `orders` and `order_items`. A role that can drop tables or read every column turns one injected statement into a much larger loss.

### 3.3 Tables and documents

TrailShop wants to store a specification sheet for each product. Tents have a capacity and a packed size. Boots have a size run and a sole type. The fields differ by product. Orders, payments, and stock stay as they are.

Say which of these facts belong in ordinary columns and foreign keys, and which fit a document or a `JSONB` column. Give one reason for each choice.

> [!NOTE]
> ***Your Answer***
>
> Orders, order lines, payments, prices, stock, and the keys between them stay in relational tables. Those facts need foreign keys, `CHECK` constraints, and a transaction that updates stock and records the order together.
>
> The specification sheet fits a `JSONB` column on `products`, or a document beside the product. Its fields differ from product to product, and the shop usually reads the whole sheet with that product. The price and the stock stay as ordinary columns so they can be constrained and indexed.
