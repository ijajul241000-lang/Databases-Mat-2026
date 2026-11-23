# Week 50 — Final Review: SQL — Exercises (Optional)

> [!IMPORTANT]
> **_Optional Exam Preparation_**
> **No submission is required this week.** Write each answer before you open `Answers.md`. The tasks use the current TrailShop schema and the fundamentals from Weeks 40, 41, 44, 46, 47, and 48.

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
> -- Write your query here
>
>
> ```

### 1.2 Group and filter the groups

For each category, show `category_name`, the number of products, and the average price. Keep categories whose average price is above €100. Sort by average price, highest first. Join through `product_categories`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.3 Inner join

List each order's `order_id`, `status`, and the customer's `first_name` and `last_name`. Include only orders that have a customer, which in this schema is every order.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.4 Left join, including rows with no match

List every category name and the names of its products. Include categories that have no products. Those rows should show a null product name.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.5 Customers with no orders

List `customer_id`, `first_name`, and `last_name` for customers who have never ordered. Use a left join.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.6 Three or more tables

For every order line, show the customer's last name, the order id, the product name, the quantity, and the line total (`quantity * unit_price`).

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.7 A subquery

List the `name` and `price` of products that appear on at least one order line. Use `EXISTS` or `IN`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.8 A CTE

Using `WITH`, compute each customer's total spending from order lines, then return `first_name`, `last_name`, and `total_spent` for customers who have spent more than €200. Sort by `total_spent` descending.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 1.9 A view

Create a view `pending_orders` with `order_id`, `order_date`, `customer_id`, `first_name`, and `last_name` for orders whose status is `'pending'`.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
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
> -- Write your query here
>
>
> ```

### 1.11 An index and the plan

Orders are looked up by `customer_id`, and `orders` is large. Write `CREATE INDEX` for that lookup. Then say what `EXPLAIN` is likely to show before the index and after it, and name one reason you would not add this index on a table of a few dozen rows.

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
> *(Write your answer here.)*
>
>
>
>

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
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
> *(Write your answer here.)*
>
>
>
>

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
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
> *(Write your answer here.)*
>
>
>
>

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 2.4 Raise the price of product 10 by 10 percent

```sql
UPDATE products
SET price = price * 1.10;
```

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
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
> *(Write your answer here.)*
>
>
>
>

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

---

## 3. Short Written Questions

### 3.1 ACID

A checkout inserts an order, inserts two order lines, and decreases stock. Explain each ACID property for that checkout. In the durability sentence, state when the change is safe from a crash.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.2 Injection and roles

A shop page builds this statement by pasting the typed email into the SQL text:

```text
SELECT customer_id FROM customers WHERE email = '<typed email>'
```

Explain how this can be abused, what a parameterized query does instead, and why the database role used by the shop should not own every privilege.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.3 Tables and documents

TrailShop wants to store a specification sheet for each product. Tents have a capacity and a packed size. Boots have a size run and a sole type. The fields differ by product. Orders, payments, and stock stay as they are.

Say which of these facts belong in ordinary columns and foreign keys, and which fit a document or a `JSONB` column. Give one reason for each choice.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
