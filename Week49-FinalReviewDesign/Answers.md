# Week 49 — Answers: Final Review — Database Design

> [!IMPORTANT]
> **_Model Answers_**
> Answers for the Week 49 exercises. Each question is repeated, and the model answer is in the same answer field. Write your own answer in `Exercises.md` before you read this file.

The TrailShop names used here are the current schema: `products` has no `category_id`. Categories are linked through `product_categories`. `order_items.unit_price` is the price charged on that order.

---

## Exercise 1 — Concept Questions

Answer in your own words. A short example is part of a complete answer.

### 1.1 Week 36 — Data and information

`189.50` appears in a TrailShop spreadsheet. Explain the difference between data and information, using this value as the data.

> [!NOTE]
> ***Your Answer***
>
> `189.50` is data: a raw fact with no meaning on its own. It becomes information when the context is added, for example "the Alpine Pro Hiking Boots cost €189.50". The database stores the fact once, in `products.price`, so a query can produce that information reliably.

### 1.2 Week 36 — Files and the three levels

Name two problems of storing TrailShop's data in separate files. Then name the three levels of the three-schema architecture and say what each level describes.

> [!NOTE]
> ***Your Answer***
>
> Any two of these problems, with a consequence: redundancy (the same price copied in several files, and the copies disagree), isolation (customers and orders cannot be combined without custom matching), integrity (a negative price is accepted), security (opening the file shows every column), concurrency (two saves overwrite each other).
>
> - **External:** what one user or application sees, such as the warehouse view of stock.
> - **Conceptual:** the full logical structure — all tables, columns, relationships, and constraints.
> - **Internal:** how PostgreSQL stores the data on disk, including indexes.

### 1.3 Week 37 — Keys

In `customers`, both `customer_id` and `email` uniquely identify a row. Explain candidate key, primary key, and foreign key. Say which of `customer_id` and `email` is the primary key in TrailShop, and give one foreign key that references `customers`.

> [!NOTE]
> ***Your Answer***
>
> A candidate key is a minimal set of columns that identifies each row. Both `customer_id` and `email` are candidate keys. The primary key is the candidate key chosen as the identifier: TrailShop chooses `customer_id`. `email` stays unique, so it remains a candidate key. A foreign key is a column whose values must match a key in another table. `orders.customer_id` is a foreign key referencing `customers.customer_id`.

### 1.4 Week 37 — Integrity and NULL

State entity integrity and referential integrity. Explain why a primary key cannot be `NULL`, and why `WHERE phone = NULL` does not find customers with no phone.

> [!NOTE]
> ***Your Answer***
>
> Entity integrity: the primary key is unique and no part of it may be `NULL`, because a missing identifier cannot distinguish the row. Referential integrity: a foreign-key value must equal an existing referenced key, or be `NULL` when the column allows it.
>
> `NULL` means "no value". It is not equal to any value, including another `NULL`, so `phone = NULL` matches no rows. The test is `phone IS NULL`.

### 1.5 Week 38 — Cardinality

For each sentence, give the cardinality (1:1, 1:N, or M:N) and say whether a junction table is required.

1. A customer places orders. An order is placed by one customer.
2. A product is filed under categories. A category contains products. A product may be in several categories.
3. An order is paid by one payment. A payment pays one order.

> [!NOTE]
> ***Your Answer***
>
> 1. **1:N** from customer to order. No junction. The foreign key `customer_id` goes on `orders`.
> 2. **M:N.** A junction is required: `product_categories(product_id, category_id)`.
> 3. **1:1.** No junction. `payments.order_id` references `orders` and is `UNIQUE`.

### 1.6 Week 38 — Participation

`orders.customer_id` is `NOT NULL`. A customer may have no orders. Which side of Customer–Order has mandatory participation, and which side has optional participation? Why can a foreign key not force every order to have at least one `order_items` row?

> [!NOTE]
> ***Your Answer***
>
> The order side is mandatory: every order has a customer, so `customer_id` is `NOT NULL`. The customer side is optional: a customer may have zero orders.
>
> `order_items.order_id` forces each line to belong to an order. It does not force an order to have a child row. "At least one line" needs an application check or a trigger.

### 1.7 Week 39 — Types and the foreign key

Choose a PostgreSQL type for each value, and say where the foreign key goes for "one customer places many orders": price, stock count, email, and the moment the customer registered.

> [!NOTE]
> ***Your Answer***
>
> - Price: `NUMERIC(10,2)`, because money must be exact.
> - Stock count: `INTEGER`.
> - Email: `VARCHAR` with a stated length, declared `UNIQUE`.
> - Registration moment: `TIMESTAMPTZ`.
>
> The foreign key goes on the "many" side: `orders.customer_id` references `customers.customer_id`.

### 1.8 Week 39 — Surrogate and natural keys

TrailShop's primary key for a customer is `customer_id`, while `email` is `UNIQUE`. Give one reason for that choice.

> [!NOTE]
> ***Your Answer***
>
> `customer_id` is a surrogate key. It does not change when the person changes email address, and it is not personal data. `email` remains a candidate key through the `UNIQUE` constraint, so two customers still cannot register the same address.

### 1.9 Week 45 — Anomalies and dependencies

A single table stores `order_id`, `customer_email`, `product_id`, and `product_name`, with primary key (`order_id`, `product_id`).

1. Name one insertion, one update, and one deletion anomaly.
2. Which columns are partial dependencies, and which normal form do they violate?
3. Why is `order_items.unit_price` in the real TrailShop schema allowed to repeat a price that also exists on `products`?

> [!NOTE]
> ***Your Answer***
>
> 1. **Insertion:** a new product cannot be stored until it appears on an order. **Update:** changing a product name means updating every order line, and a missed row leaves two names. **Deletion:** deleting the only order line for a product also deletes the product's name.
> 2. `customer_email` depends only on `order_id`, and `product_name` depends only on `product_id`. Both are partial dependencies. The table can be in 1NF and still violate 2NF.
> 3. `unit_price` is the price charged on that order. `products.price` is the current catalog price. They are different facts. The line keeps the historical price after the catalog price changes.

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
> | Entity | Primary key | Other attributes | Other uniqueness |
> | --- | --- | --- | --- |
> | User | `user_id` | email, username, subscription_tier, created_at | email, username |
> | Artist | `artist_id` | artist_name, bio | |
> | Genre | `genre_id` | genre_name | genre_name |
> | ArtistGenre | (`artist_id`, `genre_id`) | none | the pair itself |
> | Album | `album_id` | artist_id, title, release_date, cover_url | |
> | Song | `song_id` | album_id, title, duration_seconds, track_number | (`album_id`, `track_number`) |
> | Playlist | `playlist_id` | user_id, name, description | |
> | PlaylistSong | (`playlist_id`, `song_id`) | none, or a position if you store order | the pair itself |
> | UserFollow | (`user_id`, `artist_id`) | none | the pair itself |
> | Play | `play_id` | user_id, song_id, played_at | none — repeats are allowed |
>
> Artist–Genre, Playlist–Song, and User–Artist are M:N, so each has a junction.

### 2.2 Relationships and cardinalities

For each relationship, state the cardinality, which side is optional, and whether the foreign key sits on an existing table or in a junction.

> [!NOTE]
> ***Your Answer***
>
> | Relationship | Cardinality | Foreign key |
> | --- | --- | --- |
> | Artist — Album | 1:N, album side mandatory | `albums.artist_id` `NOT NULL` |
> | Album — Song | 1:N, song side mandatory | `songs.album_id` `NOT NULL` |
> | Artist — Genre | M:N, both directions optional until linked | junction `artist_genres` |
> | User — Playlist | 1:N, playlist side mandatory | `playlists.user_id` `NOT NULL` |
> | Playlist — Song | M:N | junction `playlist_songs` |
> | User — Artist (follows) | M:N | junction `user_follows` |
> | User — Play | 1:N | `plays.user_id` `NOT NULL` |
> | Song — Play | 1:N | `plays.song_id` `NOT NULL` |
>
> A user may follow nobody. An artist may have no album yet. A playlist may be empty: the junction does not force a playlist to contain a song.

### 2.3 CREATE TABLE statements

Write `CREATE TABLE` statements for every table, including the junction tables. Include primary keys, foreign keys, `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` where the requirements need them. Choose an `ON DELETE` action you can justify.

> [!NOTE]
> ***Your SQL***
>
> One consistent set of `ON DELETE` choices is shown. Another choice earns the credit when the reason matches the choice. `RESTRICT` is PostgreSQL's default when you omit `ON DELETE`.
>
> ```sql
> CREATE TABLE users (
>     user_id           SERIAL       PRIMARY KEY,
>     email             VARCHAR(254) NOT NULL UNIQUE,
>     username          VARCHAR(50)  NOT NULL UNIQUE,
>     subscription_tier VARCHAR(10)  NOT NULL DEFAULT 'free'
>                       CHECK (subscription_tier IN ('free', 'premium', 'family')),
>     created_at        TIMESTAMPTZ  NOT NULL DEFAULT NOW()
> );
>
> CREATE TABLE artists (
>     artist_id   SERIAL       PRIMARY KEY,
>     artist_name VARCHAR(200) NOT NULL,
>     bio         TEXT
> );
>
> CREATE TABLE genres (
>     genre_id   SERIAL      PRIMARY KEY,
>     genre_name VARCHAR(50) NOT NULL UNIQUE
> );
>
> CREATE TABLE artist_genres (
>     artist_id INTEGER NOT NULL
>               REFERENCES artists(artist_id)
>               ON DELETE CASCADE
>               ON UPDATE CASCADE,
>     genre_id  INTEGER NOT NULL
>               REFERENCES genres(genre_id)
>               ON DELETE CASCADE
>               ON UPDATE CASCADE,
>     PRIMARY KEY (artist_id, genre_id)
> );
>
> CREATE TABLE albums (
>     album_id     SERIAL       PRIMARY KEY,
>     artist_id    INTEGER      NOT NULL
>                  REFERENCES artists(artist_id)
>                  ON DELETE RESTRICT
>                  ON UPDATE CASCADE,
>     title        VARCHAR(200) NOT NULL,
>     release_date DATE,
>     cover_url    VARCHAR(500)
> );
>
> CREATE TABLE songs (
>     song_id          SERIAL       PRIMARY KEY,
>     album_id         INTEGER      NOT NULL
>                      REFERENCES albums(album_id)
>                      ON DELETE CASCADE
>                      ON UPDATE CASCADE,
>     title            VARCHAR(200) NOT NULL,
>     duration_seconds INTEGER      NOT NULL
>                      CHECK (duration_seconds > 0),
>     track_number     INTEGER      NOT NULL
>                      CHECK (track_number > 0),
>     UNIQUE (album_id, track_number)
> );
>
> CREATE TABLE playlists (
>     playlist_id SERIAL       PRIMARY KEY,
>     user_id     INTEGER      NOT NULL
>                 REFERENCES users(user_id)
>                 ON DELETE CASCADE
>                 ON UPDATE CASCADE,
>     name        VARCHAR(100) NOT NULL,
>     description TEXT
> );
>
> CREATE TABLE playlist_songs (
>     playlist_id INTEGER NOT NULL
>                 REFERENCES playlists(playlist_id)
>                 ON DELETE CASCADE
>                 ON UPDATE CASCADE,
>     song_id     INTEGER NOT NULL
>                 REFERENCES songs(song_id)
>                 ON DELETE CASCADE
>                 ON UPDATE CASCADE,
>     PRIMARY KEY (playlist_id, song_id)
> );
>
> CREATE TABLE user_follows (
>     user_id   INTEGER NOT NULL
>               REFERENCES users(user_id)
>               ON DELETE CASCADE
>               ON UPDATE CASCADE,
>     artist_id INTEGER NOT NULL
>               REFERENCES artists(artist_id)
>               ON DELETE CASCADE
>               ON UPDATE CASCADE,
>     PRIMARY KEY (user_id, artist_id)
> );
>
> CREATE TABLE plays (
>     play_id   SERIAL      PRIMARY KEY,
>     user_id   INTEGER     NOT NULL
>               REFERENCES users(user_id)
>               ON DELETE CASCADE
>               ON UPDATE CASCADE,
>     song_id   INTEGER     NOT NULL
>               REFERENCES songs(song_id)
>               ON DELETE RESTRICT
>               ON UPDATE CASCADE,
>     played_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
> );
> ```
>
> `albums.artist_id` uses `RESTRICT` so an artist with albums cannot disappear by accident. `songs.album_id` uses `CASCADE` because a song in this model belongs to that album. `plays` keeps a surrogate key because the same user can play the same song more than once, so (`user_id`, `song_id`) is not unique.

### 2.4 Third normal form

For each table, name the primary key and state why the table is in 3NF. Say what would be wrong with storing an artist's genres as one comma-separated column.

> [!NOTE]
> ***Your Answer***
>
> Every non-key column depends on the whole primary key of its own table. `artist_genres`, `playlist_songs`, and `user_follows` have no non-key columns. `songs.track_number` depends on the song, and the extra `UNIQUE (album_id, track_number)` stops two songs on the same album sharing a number. Nothing depends on a non-key column: the genre name lives on `genres`, not on the artist row.
>
> A comma-separated genre list in one column is not atomic, so the table would violate 1NF. It would also make "which artists are tagged Folk?" a string search, and renaming a genre would mean editing every artist.

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
> - **Insertion:** a new piece of gear, or a new customer who has not rented yet, cannot be stored, because every row is a slip.
> - **Update:** Anna's email is stored on every slip. Changing one copy and missing the other leaves two email addresses.
> - **Deletion:** if slip 502 is the only remaining record of some other gear, deleting that slip deletes the gear as well. Deleting both of Anna's slips also deletes Anna.

### 3.2 First normal form

Rewrite the data so the table is in 1NF. Show the rows, name the primary key, and say which 1NF rule the original `gear_list` column broke.

> [!NOTE]
> ***Your Answer***
>
> `gear_list` is not atomic: one cell holds several gear facts. 1NF gives one row per gear on a slip.
>
> **rental_lines** — primary key (`slip_id`, `gear_id`)
>
> | slip_id | slip_date | customer_id | customer_name | customer_email | gear_id | gear_name | days | daily_rate |
> | --- | --- | --- | --- | --- | --- | --- | --- | --- |
> | 501 | 2026-06-01 | 3 | Anna Virtanen | anna@mail.fi | 10 | Ridge Tent | 3 | 12.00 |
> | 501 | 2026-06-01 | 3 | Anna Virtanen | anna@mail.fi | 11 | Stove | 3 | 4.50 |
> | 502 | 2026-06-02 | 3 | Anna Virtanen | anna@mail.fi | 10 | Ridge Tent | 2 | 12.00 |
>
> The primary key is the pair, because a slip has many gear rows and a piece of gear appears on many slips.

### 3.3 Second normal form

Name every partial dependency in the 1NF table. Split the table into 2NF tables and show their columns and primary keys.

> [!NOTE]
> ***Your Answer***
>
> The primary key is (`slip_id`, `gear_id`).
>
> - `slip_date`, `customer_id`, `customer_name`, and `customer_email` depend only on `slip_id`.
> - `gear_name` and `daily_rate` depend only on `gear_id`.
> - `days` depends on the whole key.
>
> ```
> slips(slip_id PK, slip_date, customer_id, customer_name, customer_email)
> gear(gear_id PK, gear_name, daily_rate)
> rental_lines(slip_id PK/FK, gear_id PK/FK, days)
> ```

### 3.4 Third normal form

Name the transitive dependency that remains. Show the final 3NF tables. State the primary key of each table and where each foreign key goes.

> [!NOTE]
> ***Your Answer***
>
> On `slips`, `customer_name` and `customer_email` depend on `customer_id`, which is not the primary key. That is a transitive dependency.
>
> ```
> customers(customer_id PK, customer_name, customer_email)
> slips(slip_id PK, slip_date, customer_id FK)
> gear(gear_id PK, gear_name, daily_rate)
> rental_lines(slip_id PK/FK, gear_id PK/FK, days)
> ```
>
> `slips.customer_id` references `customers`. `rental_lines` references `slips` and `gear`. `days` stays on the junction because it depends on one gear on one slip. `daily_rate` stays on `gear` because the business rule says it is the current catalog rate.

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
> | Table | Primary key | Foreign keys | Cardinality |
> | --- | --- | --- | --- |
> | `categories` | `category_id` | none | the "one" side is reached through the junction |
> | `customers` | `customer_id` | none | 1:N toward `orders` |
> | `products` | `product_id` | none | no `category_id` column |
> | `product_categories` | (`product_id`, `category_id`) | both columns | Category M:N Product |
> | `orders` | `order_id` | `customer_id` → `customers` | N:1 toward customers |
> | `order_items` | (`order_id`, `product_id`) | both columns | Order M:N Product, with `quantity` and `unit_price` |
> | `payments` | `payment_id` | `order_id` → `orders`, and `UNIQUE` | Order 1:1 Payment |

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
> 1. A product price is a positive amount.
> 2. Two customers cannot register the same email address.
> 3. An order status is one of the five allowed values.
> 4. A customer who still has orders cannot be deleted.
> 5. Deleting an order deletes its lines. A line cannot exist without its order.
> 6. An order has at most one payment.

### 4.3 Normalization check

For `product_categories` and `order_items`, state the primary key, whether any non-key column has a partial or transitive dependency, and why `unit_price` is in 3NF.

> [!NOTE]
> ***Your Answer***
>
> `product_categories` has primary key (`product_id`, `category_id`) and no other columns, so there is no partial or transitive dependency.
>
> `order_items` has primary key (`order_id`, `product_id`). `quantity` and `unit_price` depend on that whole pair: how many of this product were bought on this order, and the price charged then. `unit_price` does not depend on `products.price`. It records a different fact, the historical price, so it is not a transitive copy of the catalog price.
