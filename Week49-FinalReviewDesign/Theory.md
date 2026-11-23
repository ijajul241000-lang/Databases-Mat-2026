# Week 49 — Final Review: Database Design

> **Exam review week.** No new concepts and no required work. Use this chapter and the optional exercises to practise the design fundamentals from Weeks 36, 37, 38, 39, and 45. SQL, transactions, indexes, and administration are reviewed in Week 50.

## Chapter 13: "The Design Review"

The TrailShop database is finished. This week you look back at the design decisions that produced it: why a database was needed, how the relational rules work, how the ER model became tables, and how you check that those tables are in third normal form.

---

## Learning Objectives

After this review you will be able to:

- Explain why a database is used instead of separate files, and name the three schema levels
- Identify keys, `NULL`, entity integrity, and referential integrity
- Read a short requirement and state entities, cardinalities, and participation
- Turn an ER model into tables, foreign keys, data types, and constraints
- Find insertion, update, and deletion anomalies and normalize a table to 3NF
- Apply a design checklist to the TrailShop schema

---

## How to Use This Chapter

Work through the five weeks below. Each section is the idea, one TrailShop example, and the mistake that most often loses marks. Then use the checklist in Section 7 on the exercises.

Weeks 40–41 and 44 (SQL), 46 (transactions), 47 (indexes), and 48 (administration, security, and NoSQL) are reviewed in Week 50. Week 42 was Exam 1. Week 43 was a holiday.

The later weekly chapters mark some topics as advanced. They are not the focus of this review: Boyce–Codd normal form beyond the statement that 3NF is enough for this course, and any design technique that was not in the learning objectives of Weeks 36–39 and 45.

---

## 1. Week 36 — Why Databases Exist

### The idea

**Data** is a raw fact. **Information** is that fact with a meaning you can use. `149.99` is data. "The Trail Runner X costs €149.99" is information.

A **database** is a shared, logically related collection of data, built for a purpose. A **DBMS** (PostgreSQL, in this course) is the software that stores that data and controls access to it. Applications talk to the DBMS. They do not open the data files themselves.

Separate files and spreadsheets fail in five ways:

| Problem | What goes wrong at TrailShop | What the DBMS does |
| --- | --- | --- |
| Redundancy and inconsistency | The same price is stored in three spreadsheets and the copies drift apart | Each fact is stored once |
| Isolation | Customers, orders, and categories live in different files and formats | Related tables can be queried together |
| Integrity | A blank name or a negative price is accepted | Constraints reject invalid rows |
| Security | Opening the file reveals every column | Roles grant access to specific actions |
| Concurrency | Two people save the same spreadsheet and one set of edits disappears | The DBMS coordinates simultaneous users |

The **three-schema architecture** separates three views of the same database:

| Level | What it describes | TrailShop example |
| --- | --- | --- |
| External | What one user or application sees | The warehouse view shows `stock_quantity`; the shop view shows `name` and `price` |
| Conceptual | The full logical structure: tables, columns, relationships, constraints | The TrailShop schema as a whole |
| Internal | How the data is stored on disk | Files, pages, and indexes, managed by PostgreSQL |

**Logical data independence** means you can change the conceptual schema, such as adding `weight_kg`, and applications that do not use the new column keep working. **Physical data independence** means you can change storage, such as adding an index, without rewriting those applications.

### TrailShop example

The founders could not answer "which category brought in the most revenue?" from separate spreadsheets. The database stores products, categories, and order lines once, and a query produces the information.

### Common mistake

Calling a spreadsheet a database because it has rows and columns. A table in a file has no DBMS enforcing integrity, concurrency, or a single shared structure.

---

## 2. Week 37 — The Relational Model

### The idea

| Relational term | Everyday word | TrailShop example |
| --- | --- | --- |
| Relation | Table | `products` |
| Tuple | Row | One product |
| Attribute | Column | `price` |
| Domain | The set of allowed values | Prices that are numbers with two decimal places, greater than 0 |
| Degree | Number of columns | |
| Cardinality (of a relation) | Number of rows | |

This "cardinality" is the number of rows. In Week 38, cardinality means how many entities participate in a relationship (1:1, 1:N, M:N). Both uses appear on the exam. Read the question to see which one it means.

**Keys**

- A **candidate key** is a minimal set of columns that uniquely identifies each row. `customer_id` and `email` can both be candidate keys of `customers`.
- The **primary key** is the candidate key chosen as the row's identifier. TrailShop uses `customer_id`.
- A **foreign key** is a column (or columns) whose values must match a candidate key in another table, usually the primary key. `orders.customer_id` references `customers.customer_id`.
- A **composite key** uses more than one column. `product_categories` is identified by (`product_id`, `category_id`).
- A **surrogate key** is an artificial identifier such as `SERIAL`. A **natural key** is a real-world identifier such as `email`.

**`NULL`** means "no value", not zero and not an empty string. `NULL` is not equal to anything, including another `NULL`. The test is `IS NULL` or `IS NOT NULL`.

**Integrity rules**

- **Entity integrity:** every row must be identifiable, so the primary key is unique and no part of it may be `NULL`.
- **Referential integrity:** a foreign-key value must match an existing referenced key, or be `NULL` when the column allows it. `orders.customer_id` is `NOT NULL`, so every order refers to a real customer.

### TrailShop example

`product_categories` stores pairs of keys. The primary key (`product_id`, `category_id`) stops the same product being filed under the same category twice. Each foreign key must point at a row that already exists in `products` or `categories`.

### Common mistake

Writing a condition that compares a column to `NULL` with `=`. The comparison does not match those rows. Use `IS NULL`.

---

## 3. Week 38 — Conceptual Modelling

### The idea

An ER model is a picture of the business before you write SQL.

- An **entity** is a thing you store data about. A **strong entity** has its own key (`customers`, `products`). A **weak entity** is identified only together with an owner. An order line is identified by the order plus the product.
- An **attribute** is a property of an entity. A **composite** attribute, such as an address, is split into `street`, `city`, `postal_code`, and `country`. A **multi-valued** attribute, such as several phone numbers, becomes another entity. A **derived** value, such as an order total, is calculated from stored columns rather than stored again.
- A **relationship** connects entities.
- **Cardinality** asks "how many?" in both directions. **Participation** asks "must it?" Total (mandatory) participation means every instance takes part. Partial (optional) participation means some instances do not.

Ask both directions with a real example:

1. One customer — how many orders? Many.
2. One order — how many customers? One.

That is **1:N** from customer to order. The customer side is optional (a customer may have no orders). The order side is mandatory (every order has a customer).

**M:N** cannot be stored as a single foreign key. It becomes an associative (junction) entity.

| Relationship | Cardinality | How TrailShop models it |
| --- | --- | --- |
| Customer — Order | 1:N | `orders.customer_id` |
| Category — Product | M:N | `product_categories` (`product_id`, `category_id`) |
| Order — Product | M:N | `order_items`, with `quantity` and `unit_price` on the junction |
| Order — Payment | 1:1 | `payments.order_id` is `UNIQUE` |

Crow's foot notation draws the "many" end as a fork and the "one" end as a single mark. An open circle means optional. A bar means mandatory. The exam accepts a clear diagram or a precise written description of the same facts.

A foreign key on the child records the parent's key. It enforces "each child has a parent". It does not, by itself, enforce "each parent has at least one child". TrailShop's business rule says an order should contain at least one item, and a product may have no category yet. "At least one item" needs an application check or a trigger.

### TrailShop example

A rain jacket can be both Clothing and Accessories. `products` therefore has no `category_id`. The link lives in `product_categories`.

### Common mistake

Drawing Category–Product as 1:N and putting `category_id` on `products`. That was an early sketch. From Week 38 onward one product belongs to many categories.

---

## 4. Week 39 — Logical Design

### The idea

Logical design turns the ER model into tables.

| ER construct | Relational result |
| --- | --- |
| Strong entity | A table. The key attribute becomes the primary key |
| Composite attribute | Separate columns (`street`, `city`, `postal_code`, `country`) |
| 1:N relationship | The primary key of the "one" side is a foreign key on the "many" side |
| Mandatory participation on the many side | That foreign key is `NOT NULL` |
| M:N relationship | A junction table whose primary key is the two foreign keys, unless extra columns need their own identifier |
| 1:1 relationship | A foreign key on one side, with `UNIQUE` so the link cannot be repeated |

**Data types used in TrailShop**

| Kind of value | Type | Why |
| --- | --- | --- |
| Identifier | `SERIAL` or `INTEGER` | `SERIAL` creates an integer sequence for surrogate keys |
| Money | `NUMERIC(10,2)` | Exact decimal. Floating-point types round money |
| Short text | `VARCHAR(n)` | Names, emails, status codes, with a stated maximum |
| Long text | `TEXT` | Descriptions |
| Whole count | `INTEGER` | `stock_quantity`, `quantity` |
| Weight | `NUMERIC(6,2)` | Kilograms, optional on `products` |
| Moment in time | `TIMESTAMPTZ` | `registered_at`, `order_date`, `created_at` |
| Yes/no | `BOOLEAN` | When the value really is only true or false |

**Constraints**

| Constraint | Role in TrailShop |
| --- | --- |
| `PRIMARY KEY` | Identifies the row |
| `NOT NULL` | The column must have a value |
| `UNIQUE` | No duplicates. `email` is unique. `payments.order_id` is unique, which makes Order–Payment 1:1 |
| `CHECK` | A condition. `price > 0`, `stock_quantity >= 0`, `status` in a fixed list |
| `DEFAULT` | A value when the insert omits the column. `stock_quantity` defaults to 0. `country` defaults to `'Finland'` |
| `REFERENCES` | The foreign key. `ON DELETE RESTRICT` blocks the parent delete. `ON DELETE CASCADE` deletes the dependent rows. `ON UPDATE CASCADE` follows a changed parent key |

**Surrogate and natural keys.** `email` can identify a customer, and it is declared `UNIQUE`. The primary key is still `customer_id`, because an email can change and it is personal data. The natural key remains a candidate key.

Create parent tables before the tables that reference them: `categories`, `customers`, and `products` first; then `product_categories`, `orders`, `order_items`, and `payments`.

### TrailShop example

```sql
CREATE TABLE products (
    product_id     SERIAL         PRIMARY KEY,
    name           VARCHAR(100)   NOT NULL,
    description    TEXT,
    price          NUMERIC(10,2)  NOT NULL
                   CONSTRAINT products_price_positive CHECK (price > 0),
    weight_kg      NUMERIC(6,2)
                   CONSTRAINT products_weight_positive CHECK (weight_kg > 0),
    stock_quantity INTEGER        NOT NULL DEFAULT 0
                   CONSTRAINT products_stock_non_negative CHECK (stock_quantity >= 0),
    created_at     TIMESTAMPTZ    NOT NULL DEFAULT NOW()
);
```

There is no `category_id` column. Categories are attached in `product_categories`.

### Common mistake

Storing a price in a floating-point column, or placing the foreign key on the "one" side of a 1:N relationship. The order belongs to one customer, so `customer_id` belongs on `orders`.

---

## 5. Week 45 — Normalization

### The idea

Normalization organizes tables so each fact is stored once. Redundancy produces three **anomalies**:

| Anomaly | What happens |
| --- | --- |
| Insertion | You cannot store a fact on its own. A new product cannot be recorded until somebody orders it, because the product columns live only on the order row |
| Update | The same fact is copied on many rows. Changing a customer's email means changing every copy, and a missed row leaves two emails |
| Deletion | Removing a row removes an unrelated fact. Deleting the only order for a product also deletes the product |

**Normal forms**

| Form | Rule |
| --- | --- |
| **1NF** | Values are atomic, there are no repeating groups (`phone1`, `phone2`, or a comma-separated list), and the table has a primary key |
| **2NF** | 1NF, and every non-key column depends on the **whole** primary key. A **partial dependency** is a non-key column that depends on only part of a composite key. A single-column primary key cannot have a partial dependency |
| **3NF** | 2NF, and no non-key column depends on another non-key column. That indirect link is a **transitive dependency** |

The short form: every non-key column depends on the key, the whole key, and nothing but the key.

**Worked dependencies**

- `order_items` with primary key (`order_id`, `product_id`): `quantity` depends on the whole key. `name` and the catalog `price` depend only on `product_id`, so they belong on `products`, not on the line.
- A table `products(product_id, name, category_id, category_name)` has a transitive dependency: `category_name` depends on `category_id`. The category name moves to `categories`. In TrailShop the link is M:N, so the product does not keep `category_id` either. The pair of keys moves to `product_categories`.

**When a repeated value is still correct.** `order_items.unit_price` is the price charged on that order. It is a different fact from `products.price`, which is today's catalog price. Keeping the snapshot is required history. It is not a 3NF violation.

**When not to normalize further.** Design the transactional schema to 3NF first. A later, measured performance problem, or a reporting database loaded in batches, may justify a controlled copy of a value. Reaching 3NF is enough for this course. Boyce–Codd normal form is a stricter form that this exam does not ask you to apply.

### TrailShop example

`product_categories` stores only the two keys. Both columns are in the primary key, so there is no non-key column that could depend on part of the key. `orders` stores `customer_id`, `order_date`, and `status`, each depending on `order_id`. The customer's name stays on `customers`.

### Common mistake

Removing `unit_price` from `order_items` in the name of normalization, or claiming a partial dependency in a table whose primary key is one column.

---

## 6. The Design Pipeline

Use this order when a question gives you a description and asks for a schema.

```
Requirements → ER model → Logical tables → 3NF check → CREATE TABLE
```

1. **Requirements.** List the things you must remember and the questions the database must answer.
2. **ER model.** Name entities, attributes, and both directions of each relationship. Resolve every M:N with a junction.
3. **Logical tables.** Apply the Week 39 rules. Choose types and constraints. Put each foreign key on the "many" side.
4. **3NF check.** For each table, name the primary key and confirm every other column depends on the whole key and nothing else. Keep historical snapshots, and say why.
5. **Implementation.** Write `CREATE TABLE` in dependency order, with `ON DELETE` and `ON UPDATE` actions you can justify.

Early mistakes are cheap. A missing entity in the ER model becomes a missing table, and the database cannot answer the question that entity existed to answer.

---

## 7. Design Checklist

Use this on any schema, including your own TrailShop database and the Week 49 exercises.

**Entities and cardinalities**

- Every thing the requirements mention is an entity, or is an attribute of one.
- Each relationship has a cardinality in both directions, checked with one concrete example.
- Every M:N relationship has a junction. `products` has no `category_id`.
- Participation is reflected in `NOT NULL` where the DBMS can enforce it.

**Keys and foreign keys**

- Every table has a primary key. No part of it is nullable.
- Foreign keys reference a real key, and parent tables can be created first.
- A 1:1 link is enforced with `UNIQUE` on the foreign key (`payments.order_id`).
- Junction tables use the two foreign keys as the primary key when the pair itself is the identity.

**Data types and constraints**

- Money is `NUMERIC`, not a floating-point type.
- Text lengths, checks, and defaults match a stated business rule.
- `ON DELETE` is a choice you can explain: `RESTRICT` when the history must survive, `CASCADE` when the child cannot exist without the parent.

**Third normal form**

- No repeating groups or lists in a cell.
- No partial dependencies on composite keys.
- No transitive dependencies.
- Any copied value is a different fact, such as `order_items.unit_price`, and is labelled as such.

---

## 8. TrailShop Schema at a Glance

This is the schema the rest of the course uses. Column names here are the ones to use in exam answers about TrailShop.

| Table | Primary key | Foreign keys and notes |
| --- | --- | --- |
| `categories` | `category_id` | `category_name` is `UNIQUE` |
| `customers` | `customer_id` | `email` is `UNIQUE`. Address is `street`, `city`, `postal_code`, `country` |
| `products` | `product_id` | `name`, `price`, `weight_kg`, `stock_quantity`. No category column |
| `product_categories` | (`product_id`, `category_id`) | Pure junction for Category M:N Product. Both foreign keys `ON DELETE CASCADE` |
| `orders` | `order_id` | `customer_id` `NOT NULL`, `ON DELETE RESTRICT`. `status` is checked against a list |
| `order_items` | (`order_id`, `product_id`) | `quantity`, `unit_price` (price at order time). Deleting an order cascades to its lines. Deleting a product is restricted while a line still references it |
| `payments` | `payment_id` | `order_id` is `NOT NULL UNIQUE`, so one payment per order |

---

## Key Terms

| Term | Meaning |
| --- | --- |
| Database | A shared collection of related data, designed for a purpose |
| DBMS | Software that defines, stores, and controls the database |
| Three-schema architecture | External, conceptual, and internal levels |
| Candidate key | A minimal identifier of each row |
| Primary key | The chosen candidate key |
| Foreign key | A value that must match a key in the referenced table |
| Entity integrity | The primary key is unique and not null |
| Referential integrity | A foreign key matches an existing key, or is null when allowed |
| Cardinality | How many instances of each entity a relationship allows |
| Participation | Whether every instance must take part |
| Junction table | The table that represents an M:N relationship |
| Partial dependency | A non-key column depends on part of a composite key |
| Transitive dependency | A non-key column depends on another non-key column |
| 1NF, 2NF, 3NF | Atomic values; then no partial dependencies; then no transitive dependencies |
| Anomaly | An insertion, update, or deletion problem caused by redundancy |

---

## Summary

- Weeks 36 and 37 give the reason for a database and the rules that make tables trustworthy: one place for each fact, keys, `NULL`, and the two integrity rules.
- Weeks 38 and 39 turn requirements into an ER model and then into tables. 1:N puts a foreign key on the many side. M:N becomes a junction. TrailShop's categories are M:N through `product_categories`.
- Week 45 checks the result. Reach 3NF, keep `order_items.unit_price` as the price that was charged, and explain any later copy of a value as a deliberate exception.
- The checklist in Section 7 is the order to follow in a design question: cardinalities, keys, types, constraints, then normal form.
