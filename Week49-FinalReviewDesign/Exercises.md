# Week 49 — Exercises: Final Review — Database Design (Optional)

> [!IMPORTANT]
> **_Optional Exam Preparation_**
> **No submission is required this week.** These exercises practise the design fundamentals from Weeks 36, 37, 38, 39, and 45. Work in order. Write your answer before you open `Answers.md`.

The TrailShop names used here are the current schema: `products` has no `category_id`. Categories are linked through `product_categories`. `order_items.unit_price` is the price charged on that order.

---

## Exercise 1 — Concept Questions

Answer in your own words. A short example is part of a complete answer.

### 1.1 Week 36 — Data and information

`189.50` appears in a TrailShop spreadsheet. Explain the difference between data and information, using this value as the data.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.2 Week 36 — Files and the three levels

Name two problems of storing TrailShop's data in separate files. Then name the three levels of the three-schema architecture and say what each level describes.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.3 Week 37 — Keys

In `customers`, both `customer_id` and `email` uniquely identify a row. Explain candidate key, primary key, and foreign key. Say which of `customer_id` and `email` is the primary key in TrailShop, and give one foreign key that references `customers`.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.4 Week 37 — Integrity and NULL

State entity integrity and referential integrity. Explain why a primary key cannot be `NULL`, and why `WHERE phone = NULL` does not find customers with no phone.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.5 Week 38 — Cardinality

For each sentence, give the cardinality (1:1, 1:N, or M:N) and say whether a junction table is required.

1. A customer places orders. An order is placed by one customer.
2. A product is filed under categories. A category contains products. A product may be in several categories.
3. An order is paid by one payment. A payment pays one order.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.6 Week 38 — Participation

`orders.customer_id` is `NOT NULL`. A customer may have no orders. Which side of Customer–Order has mandatory participation, and which side has optional participation? Why can a foreign key not force every order to have at least one `order_items` row?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.7 Week 39 — Types and the foreign key

Choose a PostgreSQL type for each value, and say where the foreign key goes for "one customer places many orders": price, stock count, email, and the moment the customer registered.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.8 Week 39 — Surrogate and natural keys

TrailShop's primary key for a customer is `customer_id`, while `email` is `UNIQUE`. Give one reason for that choice.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 1.9 Week 45 — Anomalies and dependencies

A single table stores `order_id`, `customer_email`, `product_id`, and `product_name`, with primary key (`order_id`, `product_id`).

1. Name one insertion, one update, and one deletion anomaly.
2. Which columns are partial dependencies, and which normal form do they violate?
3. Why is `order_items.unit_price` in the real TrailShop schema allowed to repeat a price that also exists on `products`?

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 2 — Design Challenge: SoundWave

Design a database for **SoundWave**, a music streaming service. Use the same rules as TrailShop.

**Requirements**

- A **user** has an email, a username, and a subscription tier: `free`, `premium`, or `family`.
- An **artist** has a name and a bio. An artist is associated with one or more **genres**. A genre has a name and can apply to many artists.
- An **album** belongs to one artist and has a title, a release date, and a cover image URL.
- A **song** belongs to one album and has a title, a duration in seconds, and a track number. Track numbers are unique within an album.
- A **playlist** is created by one user and has a name and a description. A playlist contains many songs, and a song can appear in many playlists.
- A user can **follow** many artists, and an artist can be followed by many users.
- A **play** records that a user played a song at a particular time. The same user may play the same song many times.

### 2.1 Entities, attributes, and keys

List each entity, its attributes, its primary key, and any other candidate key.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 2.2 Relationships and cardinalities

For each relationship, state the cardinality, which side is optional, and whether the foreign key sits on an existing table or in a junction.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 2.3 CREATE TABLE statements

Write `CREATE TABLE` statements for every table, including the junction tables. Include primary keys, foreign keys, `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` where the requirements need them. Choose an `ON DELETE` action you can justify.

> [!NOTE]
> ***Your SQL***
>
> ```sql
> -- Write your query here
>
>
> ```

### 2.4 Third normal form

For each table, name the primary key and state why the table is in 3NF. Say what would be wrong with storing an artist's genres as one comma-separated column.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 3 — Normalization Drill: Rental Slips

TrailShop's rental desk kept this export. One slip can list several pieces of gear. The gear list was typed into a single cell as `gear_id:gear_name:days:daily_rate` values separated by commas.

**rental_slips**

| slip_id | slip_date | customer_id | customer_name | customer_email | gear_list |
| --- | --- | --- | --- | --- | --- |
| 501 | 2026-06-01 | 3 | Anna Virtanen | anna@mail.fi | `10:Ridge Tent:3:12.00, 11:Stove:3:4.50` |
| 502 | 2026-06-02 | 3 | Anna Virtanen | anna@mail.fi | `10:Ridge Tent:2:12.00` |

**Business rules**

- A slip belongs to one customer. A customer can have many slips.
- A slip rents one or more pieces of gear. The same gear can appear on many slips over time, once per slip.
- `days` is how long that gear was rented on that slip.
- `daily_rate` is the gear's current catalog rate. It is the same fact wherever that gear appears. (It is not a historical price. If it were the rate actually charged, it would stay on the slip line, as `unit_price` does on `order_items`.)
- The customer's name and email belong to the customer.

### 3.1 Anomalies

Using the table above, give one insertion anomaly, one update anomaly, and one deletion anomaly.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.2 First normal form

Rewrite the data so the table is in 1NF. Show the rows, name the primary key, and say which 1NF rule the original `gear_list` column broke.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.3 Second normal form

Name every partial dependency in the 1NF table. Split the table into 2NF tables and show their columns and primary keys.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 3.4 Third normal form

Name the transitive dependency that remains. Show the final 3NF tables. State the primary key of each table and where each foreign key goes.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

---

## Exercise 4 — TrailShop Self-Check

Use the checklist in Theory Section 7. You are checking the course schema, not inventing a new one.

### 4.1 Keys and relationships

Complete the table for the seven TrailShop tables.

| Table | Primary key | Foreign keys | Cardinality this table implements |
| --- | --- | --- | --- |
| `categories` | | | |
| `customers` | | | |
| `products` | | | |
| `product_categories` | | | |
| `orders` | | | |
| `order_items` | | | |
| `payments` | | | |

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 4.2 Constraints and business rules

Give the business rule enforced by each constraint.

1. `CHECK (price > 0)` on `products`
2. `UNIQUE (email)` on `customers`
3. `CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled'))` on `orders`
4. `ON DELETE RESTRICT` on `orders.customer_id`
5. `ON DELETE CASCADE` on `order_items.order_id`
6. `UNIQUE (order_id)` on `payments`

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>

### 4.3 Normalization check

For `product_categories` and `order_items`, state the primary key, whether any non-key column has a partial or transitive dependency, and why `unit_price` is in 3NF.

> [!NOTE]
> ***Your Answer***
>
> *(Write your answer here.)*
>
>
>
>
