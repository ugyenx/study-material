# SQL & PostgreSQL --- Complete Revision Notes

> A practical revision guide covering everything learned so far, from
> database fundamentals through PostgreSQL schema design, intermediate
> SQL, normalization, transactions, indexes, and the Foodora database
> project.

------------------------------------------------------------------------

## Table of Contents

1.  [Database Fundamentals](#1-database-fundamentals)
2.  [Relational Database Concepts](#2-relational-database-concepts)
3.  [Keys](#3-keys)
4.  [Constraints](#4-constraints)
5.  [NULL](#5-null)
6.  [SQL Data Types](#6-sql-data-types)
7.  [SQL Command Categories](#7-sql-command-categories)
8.  [Database and Table Creation](#8-database-and-table-creation)
9.  [INSERT](#9-insert)
10. [SELECT](#10-select)
11. [Filtering with WHERE](#11-filtering-with-where)
12. [Sorting and Limiting](#12-sorting-and-limiting)
13. [UPDATE](#13-update)
14. [DELETE](#14-delete)
15. [ALTER TABLE](#15-alter-table)
16. [DROP](#16-drop)
17. [TRUNCATE](#17-truncate)
18. [Aggregate Functions](#18-aggregate-functions)
19. [GROUP BY](#19-group-by)
20. [HAVING](#20-having)
21. [SQL Execution Order](#21-sql-execution-order)
22. [Relationships](#22-relationships)
23. [Foreign Keys](#23-foreign-keys)
24. [JOINs](#24-joins)
25. [Many-to-Many Relationships](#25-many-to-many-relationships)
26. [Subqueries](#26-subqueries)
27. [CTEs](#27-ctes)
28. [Window Functions](#28-window-functions)
29. [Views](#29-views)
30. [Recursive CTEs](#30-recursive-ctes)
31. [Database Design](#31-database-design)
32. [Functional Dependencies](#32-functional-dependencies)
33. [Normalization](#33-normalization)
34. [1NF, 2NF, 3NF, BCNF](#34-1nf-2nf-3nf-bcnf)
35. [Transactions](#35-transactions)
36. [ACID](#36-acid)
37. [Concurrency and Isolation](#37-concurrency-and-isolation)
38. [Locks and Deadlocks](#38-locks-and-deadlocks)
39. [Indexes](#39-indexes)
40. [B-tree Index](#40-b-tree-index)
41. [Composite and Partial Indexes](#41-composite-and-partial-indexes)
42. [EXPLAIN and EXPLAIN ANALYZE](#42-explain-and-explain-analyze)
43. [PostgreSQL Practical Workflow](#43-postgresql-practical-workflow)
44. [Foodora Database Project](#44-foodora-database-project)
45. [Important Foodora Design
    Decisions](#45-important-foodora-design-decisions)
46. [Quick SQL Cheat Sheet](#46-quick-sql-cheat-sheet)
47. [Common Mistakes](#47-common-mistakes)

------------------------------------------------------------------------

# 1. Database Fundamentals

## What is a database?

A database is an organized collection of data that can be stored,
retrieved, modified, and managed efficiently.

Example:

``` text
Users
--------------------------------
id | name  | email
1  | Ugyen | ugyen@example.com
2  | Sonam | sonam@example.com
```

A database is more than just a collection of files. A database
management system provides controlled access, consistency, security,
transactions, and efficient querying.

## DBMS

**DBMS = Database Management System**

A DBMS is software used to create, store, retrieve, update, and manage
databases.

Examples:

-   PostgreSQL
-   MySQL
-   MariaDB
-   Oracle Database
-   Microsoft SQL Server
-   SQLite

## RDBMS

**RDBMS = Relational Database Management System**

An RDBMS stores data primarily in related tables.

Example:

``` text
users
  |
  | 1:N
  v
orders
```

PostgreSQL is an RDBMS.

------------------------------------------------------------------------

# 2. Relational Database Concepts

A relational database organizes data into tables.

## Table

A table represents an entity or concept.

Example:

``` text
users
---------------------------------
id | name | email
```

## Row

A row represents one record.

``` text
1 | Ugyen | ugyen@example.com
```

## Column

A column represents an attribute.

``` text
id
name
email
```

## Schema

A schema describes the structure of the database: tables, columns,
relationships, constraints, indexes, views, etc.

------------------------------------------------------------------------

# 3. Keys

## Primary Key

Uniquely identifies each row.

``` sql
id BIGINT PRIMARY KEY
```

Properties:

-   unique
-   cannot be NULL
-   one primary key constraint per table
-   may consist of multiple columns (composite primary key)

Example:

``` sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
);
```

## Foreign Key

References a key in another table.

``` sql
user_id BIGINT REFERENCES users(id)
```

It establishes a relationship and enforces referential integrity.

## UNIQUE Key

Prevents duplicate values.

``` sql
email VARCHAR(255) UNIQUE
```

Unlike a primary key, a table can have multiple UNIQUE constraints.

## Candidate Key

A column or combination of columns that can uniquely identify a row.

Example:

``` text
users
id       -> unique
email    -> unique
```

Both may be candidate keys, while `id` is selected as the primary key.

## Composite Key

A key consisting of multiple columns.

Example:

``` sql
PRIMARY KEY (student_id, course_id)
```

Useful when the combination is unique.

------------------------------------------------------------------------

# 4. Constraints

Constraints enforce rules on data.

## NOT NULL

Value must exist.

``` sql
name VARCHAR(100) NOT NULL
```

## UNIQUE

No duplicate values.

``` sql
email VARCHAR(255) UNIQUE
```

## PRIMARY KEY

Unique + NOT NULL identity.

``` sql
id BIGINT PRIMARY KEY
```

## FOREIGN KEY

Enforces relationship integrity.

``` sql
FOREIGN KEY (user_id)
REFERENCES users(id)
```

## CHECK

Enforces a condition.

``` sql
CHECK (price >= 0)
```

Example:

``` sql
CHECK (
    status IN ('pending', 'confirmed', 'cancelled')
)
```

## DEFAULT

Provides a value when one isn't supplied.

``` sql
created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
```

------------------------------------------------------------------------

# 5. NULL

`NULL` means **unknown, missing, or not applicable**.

It is not:

-   `0`
-   an empty string `''`
-   `FALSE`

Incorrect:

``` sql
WHERE phone = NULL;
```

Correct:

``` sql
WHERE phone IS NULL;
```

And:

``` sql
WHERE phone IS NOT NULL;
```

Because NULL represents an unknown value, normal comparisons with NULL
do not produce TRUE.

------------------------------------------------------------------------

# 6. SQL Data Types

Common PostgreSQL types:

## Integer

``` sql
INTEGER
BIGINT
```

Use `BIGINT` when IDs may become large.

## Decimal / Money

``` sql
NUMERIC(10, 2)
```

Example:

``` text
180.00
1250.50
```

For financial values, `NUMERIC` is preferable to floating-point types
because it provides exact decimal arithmetic.

## Text

``` sql
TEXT
VARCHAR(100)
VARCHAR(255)
```

## Boolean

``` sql
BOOLEAN
```

Values:

``` sql
TRUE
FALSE
```

## Date and Time

``` sql
DATE
TIME
TIMESTAMP
TIMESTAMPTZ
```

For applications spanning time zones, `TIMESTAMPTZ` is often preferable.

------------------------------------------------------------------------

# 7. SQL Command Categories

## DDL --- Data Definition Language

Changes database structure.

``` text
CREATE
ALTER
DROP
TRUNCATE
```

## DML --- Data Manipulation Language

Changes data.

``` text
INSERT
UPDATE
DELETE
```

## DQL --- Data Query Language

Retrieves data.

``` text
SELECT
```

## TCL --- Transaction Control Language

Controls transactions.

``` text
BEGIN
COMMIT
ROLLBACK
SAVEPOINT
```

## DCL --- Data Control Language

Controls permissions.

``` text
GRANT
REVOKE
```

------------------------------------------------------------------------

# 8. Database and Table Creation

Create a database:

``` sql
CREATE DATABASE foodora_db;
```

Connect to it using your PostgreSQL client.

Check the current database:

``` sql
SELECT current_database();
```

Create a table:

``` sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Identity columns

PostgreSQL can automatically generate IDs:

``` sql
id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
```

You normally don't provide the ID when inserting a new row.

------------------------------------------------------------------------

# 9. INSERT

Insert one row:

``` sql
INSERT INTO users (name, email, password_hash)
VALUES ('Ugyen', 'ugyen@example.com', 'hash');
```

Insert multiple rows:

``` sql
INSERT INTO users (name, email, password_hash)
VALUES
    ('Ugyen', 'ugyen@example.com', 'hash1'),
    ('Sonam', 'sonam@example.com', 'hash2');
```

Always specify columns explicitly when practical.

------------------------------------------------------------------------

# 10. SELECT

Select everything:

``` sql
SELECT * FROM users;
```

Select specific columns:

``` sql
SELECT name, email
FROM users;
```

Aliases:

``` sql
SELECT
    name AS customer_name,
    email AS customer_email
FROM users;
```

------------------------------------------------------------------------

# 11. Filtering with WHERE

``` sql
SELECT *
FROM users
WHERE id = 1;
```

Operators:

``` text
=       equal
<>      not equal
>       greater than
<       less than
>=      greater/equal
<=      less/equal
```

Logical operators:

``` sql
WHERE age >= 18 AND city = 'Thimphu';
```

``` sql
WHERE city = 'Thimphu' OR city = 'Paro';
```

``` sql
WHERE NOT status = 'cancelled';
```

## IN

``` sql
WHERE status IN ('pending', 'confirmed');
```

## BETWEEN

``` sql
WHERE price BETWEEN 100 AND 500;
```

## LIKE

``` sql
WHERE name LIKE 'U%';
```

`%` means any number of characters.

``` text
'U%'     -> starts with U
'%ing'   -> ends with ing
'%food%' -> contains food
```

PostgreSQL also provides case-insensitive `ILIKE`:

``` sql
WHERE name ILIKE '%chicken%';
```

------------------------------------------------------------------------

# 12. Sorting and Limiting

## ORDER BY

Ascending:

``` sql
SELECT *
FROM menu_items
ORDER BY price ASC;
```

Descending:

``` sql
SELECT *
FROM menu_items
ORDER BY price DESC;
```

## LIMIT

``` sql
SELECT *
FROM menu_items
LIMIT 5;
```

## OFFSET

``` sql
SELECT *
FROM menu_items
LIMIT 10 OFFSET 20;
```

Useful for pagination, although large OFFSET values can become
inefficient.

------------------------------------------------------------------------

# 13. UPDATE

Modify existing rows.

``` sql
UPDATE users
SET phone = '17123456'
WHERE id = 1;
```

Multiple columns:

``` sql
UPDATE users
SET
    name = 'Ugyen Dorji',
    phone = '17123456'
WHERE id = 1;
```

## IMPORTANT

Never casually run:

``` sql
UPDATE users
SET phone = '17123456';
```

Without `WHERE`, every row can be modified.

Always check the target rows first:

``` sql
SELECT *
FROM users
WHERE id = 1;
```

Then update.

------------------------------------------------------------------------

# 14. DELETE

`DELETE` removes rows from a table.

Delete one row:

``` sql
DELETE FROM users
WHERE id = 1;
```

Delete rows matching a condition:

``` sql
DELETE FROM users
WHERE status = 'inactive';
```

Delete all rows:

``` sql
DELETE FROM users;
```

## DELETE vs DROP vs TRUNCATE

This distinction is extremely important.

``` text
DELETE
  ↓
removes rows

TRUNCATE
  ↓
removes all rows quickly

DROP
  ↓
removes the table itself
```

`DELETE` can use a `WHERE` condition.

------------------------------------------------------------------------

# 15. ALTER TABLE

`ALTER TABLE` changes the structure of an existing table.

## Add a column

``` sql
ALTER TABLE users
ADD COLUMN date_of_birth DATE;
```

## Rename a column

``` sql
ALTER TABLE users
RENAME COLUMN phone TO phone_number;
```

## Change a column type

``` sql
ALTER TABLE users
ALTER COLUMN name TYPE VARCHAR(150);
```

Be careful: changing a type may fail or require conversion.

## Set NOT NULL

``` sql
ALTER TABLE users
ALTER COLUMN name SET NOT NULL;
```

## Remove NOT NULL

``` sql
ALTER TABLE users
ALTER COLUMN name DROP NOT NULL;
```

## Set a default

``` sql
ALTER TABLE users
ALTER COLUMN created_at
SET DEFAULT CURRENT_TIMESTAMP;
```

## Drop a default

``` sql
ALTER TABLE users
ALTER COLUMN created_at
DROP DEFAULT;
```

## Add a constraint

``` sql
ALTER TABLE menu_items
ADD CONSTRAINT positive_price
CHECK (price >= 0);
```

## Drop a constraint

``` sql
ALTER TABLE menu_items
DROP CONSTRAINT positive_price;
```

## Rename a table

``` sql
ALTER TABLE users
RENAME TO customers;
```

------------------------------------------------------------------------

# 16. DROP

`DROP` removes a database object.

Drop a table:

``` sql
DROP TABLE users;
```

The table structure and its data are removed.

## DROP TABLE IF EXISTS

Useful when a table may not exist:

``` sql
DROP TABLE IF EXISTS users;
```

## CASCADE

PostgreSQL can remove dependent objects:

``` sql
DROP TABLE users CASCADE;
```

Be extremely careful with `CASCADE`.

It can remove objects that depend on the table.

## RESTRICT

Prevent dropping when dependencies exist:

``` sql
DROP TABLE users RESTRICT;
```

This is effectively the safer behavior when dependencies are present.

------------------------------------------------------------------------

# 17. TRUNCATE

`TRUNCATE` removes **all rows** from a table quickly while keeping the
table structure.

``` sql
TRUNCATE TABLE users;
```

The table still exists.

Compare:

``` text
DELETE FROM users;
    ↓
rows removed
table remains

TRUNCATE TABLE users;
    ↓
all rows removed efficiently
table remains

DROP TABLE users;
    ↓
table + data removed
```

## Restart identity

If you want identity sequences to restart:

``` sql
TRUNCATE TABLE users RESTART IDENTITY;
```

## CASCADE

``` sql
TRUNCATE TABLE users CASCADE;
```

This can truncate related tables, so use carefully.

------------------------------------------------------------------------

# 18. Aggregate Functions

Aggregate functions calculate a result over multiple rows.

## COUNT

``` sql
SELECT COUNT(*)
FROM users;
```

## SUM

``` sql
SELECT SUM(price)
FROM menu_items;
```

## AVG

``` sql
SELECT AVG(price)
FROM menu_items;
```

## MIN

``` sql
SELECT MIN(price)
FROM menu_items;
```

## MAX

``` sql
SELECT MAX(price)
FROM menu_items;
```

Example:

``` sql
SELECT
    COUNT(*) AS total_items,
    AVG(price) AS average_price,
    MIN(price) AS cheapest,
    MAX(price) AS most_expensive
FROM menu_items;
```

------------------------------------------------------------------------

# 19. GROUP BY

`GROUP BY` groups rows before aggregation.

Example:

``` sql
SELECT
    restaurant_id,
    COUNT(*) AS total_categories
FROM categories
GROUP BY restaurant_id;
```

Concept:

``` text
Restaurant 1 → 3 categories
Restaurant 2 → 5 categories
Restaurant 3 → 2 categories
```

Another example:

``` sql
SELECT
    status,
    COUNT(*) AS total_orders
FROM orders
GROUP BY status;
```

------------------------------------------------------------------------

# 20. HAVING

`WHERE` filters rows **before grouping**.

`HAVING` filters groups **after grouping**.

Example:

``` sql
SELECT
    restaurant_id,
    COUNT(*) AS total_orders
FROM orders
GROUP BY restaurant_id
HAVING COUNT(*) > 10;
```

Think:

``` text
WHERE
↓
filter individual rows

GROUP BY
↓
create groups

HAVING
↓
filter groups
```

------------------------------------------------------------------------

# 21. SQL Execution Order

Although we usually write SQL as:

``` sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

The logical processing order is approximately:

``` text
FROM
↓
JOIN / ON
↓
WHERE
↓
GROUP BY
↓
HAVING
↓
SELECT
↓
DISTINCT
↓
ORDER BY
↓
LIMIT/OFFSET
```

Understanding this explains many SQL behaviors.

------------------------------------------------------------------------

# 22. Relationships

Common relationships:

## One-to-One

``` text
A 1 ───── 1 B
```

Example:

``` text
person → passport
```

## One-to-Many

``` text
A 1 ───── N B
```

Example:

``` text
user → orders
```

One user can have many orders.

## Many-to-Many

``` text
A M ───── N B
```

Example:

``` text
orders ↔ menu_items
```

An order contains many menu items, and a menu item can appear in many
orders.

This usually requires a junction table.

------------------------------------------------------------------------

# 23. Foreign Keys

A foreign key creates a relationship between tables.

``` sql
CREATE TABLE addresses (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,

    FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```

Relationship:

``` text
addresses.user_id
       │
       ▼
users.id
```

A foreign key prevents invalid references.

For example, if user 999 doesn't exist:

``` sql
INSERT INTO addresses (user_id)
VALUES (999);
```

PostgreSQL rejects it.

------------------------------------------------------------------------

# 24. JOINs

JOINs combine rows from related tables.

## INNER JOIN

Returns rows that match in both tables.

``` sql
SELECT
    u.name,
    o.id
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

`JOIN` normally means `INNER JOIN`.

## LEFT JOIN

Returns all rows from the left table, even if there is no match.

``` sql
SELECT
    u.name,
    o.id
FROM users u
LEFT JOIN orders o
    ON o.user_id = u.id;
```

Useful for questions like:

> Show all users, including users who have never ordered.

## RIGHT JOIN

All rows from the right table are preserved.

Less commonly used because the same query can usually be expressed with
a LEFT JOIN by reversing table order.

## FULL OUTER JOIN

Returns matching and non-matching rows from both sides.

``` sql
SELECT *
FROM A
FULL OUTER JOIN B
    ON A.id = B.id;
```

## CROSS JOIN

Produces every combination.

If:

``` text
A = 3 rows
B = 4 rows
```

then:

``` text
CROSS JOIN = 12 rows
```

Use carefully.

------------------------------------------------------------------------

# 25. Many-to-Many Relationships

Suppose:

``` text
orders M:N menu_items
```

You cannot normally put all menu item IDs into one column.

Bad:

``` text
order_id | menu_items
1        | 1,2,3
```

Instead create a junction table:

``` text
orders
  │
  │ 1:N
  ▼
order_items
  ▲
  │ N:1
  │
menu_items
```

Example:

``` sql
CREATE TABLE order_items (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id BIGINT NOT NULL,
    menu_item_id BIGINT NOT NULL,
    quantity INTEGER NOT NULL,
    price NUMERIC(10,2) NOT NULL,

    FOREIGN KEY (order_id)
        REFERENCES orders(id),

    FOREIGN KEY (menu_item_id)
        REFERENCES menu_items(id)
);
```

------------------------------------------------------------------------

# 26. Subqueries

A subquery is a query inside another query.

Example:

``` sql
SELECT name, price
FROM menu_items
WHERE price > (
    SELECT AVG(price)
    FROM menu_items
);
```

Meaning:

``` text
Find average price
       ↓
Find items more expensive than average
```

## IN subquery

``` sql
SELECT name
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
);
```

## EXISTS

``` sql
SELECT u.name
FROM users u
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.user_id = u.id
);
```

`EXISTS` checks whether at least one matching row exists.

------------------------------------------------------------------------

# 27. CTEs

CTE = Common Table Expression.

Syntax:

``` sql
WITH expensive_items AS (
    SELECT *
    FROM menu_items
    WHERE price > 200
)
SELECT *
FROM expensive_items;
```

CTEs make complex queries easier to read.

Example:

``` sql
WITH restaurant_orders AS (
    SELECT
        restaurant_id,
        COUNT(*) AS total_orders
    FROM orders
    GROUP BY restaurant_id
)
SELECT *
FROM restaurant_orders
WHERE total_orders > 10;
```

------------------------------------------------------------------------

# 28. Window Functions

Window functions calculate across related rows without collapsing them
into one row.

Example:

``` sql
SELECT
    name,
    price,
    AVG(price) OVER () AS average_price
FROM menu_items;
```

Every menu item remains visible, while the overall average is added.

## ROW_NUMBER

``` sql
SELECT
    name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS rank
FROM menu_items;
```

## PARTITION BY

``` sql
SELECT
    restaurant_id,
    name,
    price,
    ROW_NUMBER() OVER (
        PARTITION BY restaurant_id
        ORDER BY price DESC
    ) AS rank
FROM menu_items;
```

`PARTITION BY` creates independent windows.

------------------------------------------------------------------------

# 29. Views

A view is a stored query that behaves like a virtual table.

``` sql
CREATE VIEW restaurant_menu AS
SELECT
    r.name AS restaurant,
    c.name AS category,
    m.name AS menu_item,
    m.price
FROM restaurants r
JOIN categories c
    ON c.restaurant_id = r.id
JOIN menu_items m
    ON m.category_id = c.id;
```

Then:

``` sql
SELECT *
FROM restaurant_menu;
```

The underlying query runs when the view is queried.

Drop it:

``` sql
DROP VIEW restaurant_menu;
```

------------------------------------------------------------------------

# 30. Recursive CTEs

Recursive CTEs are useful for hierarchical data.

Example hierarchy:

``` text
CEO
 ├── Manager A
 │    ├── Employee 1
 │    └── Employee 2
 └── Manager B
      └── Employee 3
```

Basic structure:

``` sql
WITH RECURSIVE employee_tree AS (
    -- Anchor
    SELECT id, name, manager_id
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive part
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    JOIN employee_tree t
        ON e.manager_id = t.id
)
SELECT *
FROM employee_tree;
```

Recursive CTEs are useful for trees, organizational structures,
categories, dependency graphs, and similar data.

------------------------------------------------------------------------

# 31. Database Design

Good database design aims for:

-   consistency
-   minimal unnecessary duplication
-   clear relationships
-   data integrity
-   maintainability
-   appropriate performance

Before creating tables, identify:

1.  Entities
2.  Attributes
3.  Primary keys
4.  Relationships
5.  Cardinality
6.  Constraints
7.  Business rules

Example:

``` text
User
Order
Restaurant
Category
Menu Item
Payment
Driver
Vehicle
Delivery
```

------------------------------------------------------------------------

# 32. Functional Dependencies

A functional dependency means one attribute determines another.

Notation:

``` text
A → B
```

means:

> If you know A, you can determine B.

Example:

``` text
user_id → user_name
```

assuming each user ID identifies exactly one user.

In:

``` text
users
id | name | email
```

we expect:

``` text
id → name
id → email
```

Functional dependencies are important when designing normalized
databases.

------------------------------------------------------------------------

# 33. Normalization

Normalization organizes data to reduce:

-   redundancy
-   update anomalies
-   insertion anomalies
-   deletion anomalies

Example of bad design:

``` text
orders
--------------------------------------------------
order_id | customer_name | customer_phone | food
```

Customer information is repeated across many orders.

Better:

``` text
users
orders
order_items
menu_items
```

------------------------------------------------------------------------

# 34. 1NF, 2NF, 3NF, BCNF

## 1NF --- First Normal Form

Rules:

-   each cell contains a single atomic value
-   no repeating groups
-   no lists inside a column

Bad:

``` text
order_id | items
1        | Rice, Coke, Noodles
```

Better:

``` text
order_items
order_id | menu_item_id
1        | 1
1        | 2
1        | 3
```

## 2NF --- Second Normal Form

Must:

1.  be in 1NF
2.  have no partial dependency on part of a composite key

This mainly matters when a table has a composite primary key.

## 3NF --- Third Normal Form

Must:

1.  be in 2NF
2.  have no inappropriate transitive dependencies

Example problem:

``` text
student_id → department_id
department_id → department_name
```

Don't unnecessarily store:

``` text
student_id | department_id | department_name
```

Instead:

``` text
students
departments
```

## BCNF

A stronger form of 3NF.

For every functional dependency:

``` text
X → Y
```

`X` should be a superkey under BCNF's conditions.

------------------------------------------------------------------------

# 35. Transactions

A transaction is a group of operations treated as one logical unit.

Example:

``` text
Transfer ₹100
    │
    ├── subtract ₹100 from A
    │
    └── add ₹100 to B
```

Both should succeed or neither should.

Start:

``` sql
BEGIN;
```

Commit:

``` sql
COMMIT;
```

Undo:

``` sql
ROLLBACK;
```

Example:

``` sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If something fails:

``` sql
ROLLBACK;
```

------------------------------------------------------------------------

# 36. ACID

Transactions aim to provide ACID properties.

## Atomicity

All operations succeed or none do.

``` text
ALL or NOTHING
```

## Consistency

The database moves from one valid state to another valid state.

Constraints help enforce consistency.

## Isolation

Concurrent transactions should not improperly interfere with each other.

## Durability

Once committed, changes should survive failures according to the
database's durability guarantees.

------------------------------------------------------------------------

# 37. Concurrency and Isolation

Multiple users can access the database simultaneously.

Potential problems include:

## Dirty Read

Transaction B reads uncommitted changes from transaction A.

## Non-repeatable Read

A row read twice gives different results because another transaction
committed a change.

## Phantom Read

A repeated query returns a different set of rows because another
transaction inserted/deleted matching rows.

PostgreSQL supports transaction isolation levels including:

``` text
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

PostgreSQL's default isolation level is `READ COMMITTED`.

Higher isolation generally provides stronger guarantees but can increase
contention or serialization failures.

------------------------------------------------------------------------

# 38. Locks and Deadlocks

Locks coordinate concurrent access.

Example:

``` sql
SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

This can lock the selected row for update within a transaction.

## Deadlock

A deadlock occurs when transactions wait for each other.

Example:

``` text
Transaction A locks Row 1
Transaction B locks Row 2

A waits for Row 2
B waits for Row 1

        ↓

      DEADLOCK
```

Databases detect deadlocks and abort one transaction so the others can
proceed.

Application code should generally be prepared to retry transactions when
appropriate.

------------------------------------------------------------------------

# 39. Indexes

An index is a data structure that helps PostgreSQL find rows more
efficiently.

Without a suitable index, PostgreSQL may perform a sequential scan:

``` text
Scan Row 1
Scan Row 2
Scan Row 3
...
Scan Row 1,000,000
```

An index can allow PostgreSQL to locate matching rows much faster.

Create:

``` sql
CREATE INDEX idx_students_department_id
ON students(department_id);
```

Then queries such as:

``` sql
SELECT *
FROM students
WHERE department_id = 10;
```

may benefit from the index.

## Important

A foreign key does **not automatically create an index on the
referencing column in PostgreSQL**.

Therefore, frequently queried foreign-key columns may need explicit
indexes.

------------------------------------------------------------------------

# 40. B-tree Index

PostgreSQL commonly uses B-tree indexes.

Important correction:

> A B-tree is NOT simply a binary tree.

A B-tree is a **balanced multi-way search tree**.

It can have:

``` text
many keys
many children
```

per node.

It is useful for:

``` text
=
<
>
<=
>=
BETWEEN
ORDER BY
```

and many related operations.

------------------------------------------------------------------------

# 41. Composite and Partial Indexes

## Composite index

Index multiple columns:

``` sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

Column order matters.

This is especially useful for queries involving the leading columns:

``` sql
WHERE user_id = 10
```

or:

``` sql
WHERE user_id = 10
AND status = 'pending'
```

It is generally less useful for a query filtering only on:

``` sql
WHERE status = 'pending'
```

because `user_id` is the leading column.

## Partial index

Indexes only rows satisfying a condition.

Example:

``` sql
CREATE INDEX idx_active_menu_items
ON menu_items(category_id)
WHERE is_available = TRUE;
```

Useful when a subset of rows is queried frequently.

------------------------------------------------------------------------

# 42. EXPLAIN and EXPLAIN ANALYZE

## EXPLAIN

Shows the query execution plan.

``` sql
EXPLAIN
SELECT *
FROM students
WHERE department_id = 10;
```

You may see:

``` text
Seq Scan
```

or:

``` text
Index Scan
```

## EXPLAIN ANALYZE

Actually executes the query and provides runtime information.

``` sql
EXPLAIN ANALYZE
SELECT *
FROM students
WHERE department_id = 10;
```

Use it to investigate performance.

Important:

> An index is not automatically better for every query.

If a query needs a large percentage of the table, PostgreSQL may
correctly choose a sequential scan.

------------------------------------------------------------------------

# 43. PostgreSQL Practical Workflow

For the Foodora project, the working structure is:

``` text
foodora-database/
├── schema.sql
├── seed.sql
└── queries.sql
```

## schema.sql

Contains table definitions:

``` sql
CREATE TABLE ...
```

## seed.sql

Contains test/sample data:

``` sql
INSERT INTO ...
```

## queries.sql

Contains queries used for learning/testing:

``` sql
SELECT ...
```

Keep useful test queries and comment them rather than deleting them:

``` sql
-- SELECT current_database();
```

When using VS Code, execute the selected SQL statement rather than
accidentally executing an entire file containing many independent
statements.

Check the current database:

``` sql
SELECT current_database();
```

Expected:

``` text
foodora_db
```

------------------------------------------------------------------------

# 44. Foodora Database Project

Current project entities:

``` text
users
addresses
restaurants
categories
menu_items
orders
order_items
payments
drivers
vehicles
deliveries
```

Current relationship design:

``` text
users
 │
 ├─────────────── 1:N ──────────────── addresses
 │
 └─────────────── 1:N ──────────────── orders
                                         │
                                         │ 1:N
                                         ▼
                                    order_items
                                         │
                                         │ N:1
                                         ▼
                                    menu_items
                                         ▲
                                         │ 1:N
                                         │
                                    categories
                                         ▲
                                         │ N:1
                                         │
                                    restaurants
```

More precisely:

``` text
Restaurant 1:N Category
Category   1:N Menu Item
Restaurant 1:N Order
User       1:N Order
User       1:N Address
Order      1:N Order Item
Menu Item  1:N Order Item
Order      1:1 Payment
Driver     1:N Vehicle
Order      1:1 Delivery
Driver     1:N Delivery
Vehicle    1:N Delivery
```

------------------------------------------------------------------------

# 45. Important Foodora Design Decisions

## Users

``` sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

`email` is UNIQUE because two users should not normally share the same
login email.

Passwords should never be stored as plaintext. Store a secure password
hash.

------------------------------------------------------------------------

## Addresses

``` text
users 1:N addresses
```

A user may have:

``` text
Home
College
Office
```

An order references the selected address.

------------------------------------------------------------------------

## Restaurants

A restaurant has its own:

-   name
-   address
-   phone

------------------------------------------------------------------------

## Categories

``` text
restaurant_id → restaurants.id
```

This means categories belong to a specific restaurant.

Example:

``` text
Dragon Kitchen
 ├── Main Course
 ├── Rice & Noodles
 └── Drinks
```

------------------------------------------------------------------------

## Menu Items

``` text
category_id → categories.id
```

We intentionally did not add:

``` text
restaurant_id
```

to `menu_items`.

Why?

Because the restaurant can already be derived:

``` text
menu_items
   ↓
categories
   ↓
restaurants
```

Adding both can duplicate relationship information and create
inconsistency.

------------------------------------------------------------------------

## Orders

An order stores:

``` text
user_id
restaurant_id
address_id
created_at
status
total_amount
```

It records:

-   who ordered
-   which restaurant
-   delivery address
-   when
-   current status
-   total charged amount

------------------------------------------------------------------------

## Order Items

An order can contain multiple menu items.

`order_items` resolves:

``` text
orders M:N menu_items
```

It stores:

``` text
order_id
menu_item_id
quantity
price
```

### Why store price twice?

``` text
menu_items.price
```

means current menu price.

``` text
order_items.price
```

means historical price at purchase time.

Example:

``` text
Current menu price = ₹220

Old order price = ₹180
```

Without historical price storage, old orders could appear to change
price when the restaurant updates its menu.

------------------------------------------------------------------------

## Payments

Payment belongs to an order:

``` text
orders 1:1 payments
```

Conceptually:

``` text
orders
   │
   ▼
payments
```

The payment table should reference `order_id`, not `user_id`, when
representing payment for a particular order.

------------------------------------------------------------------------

## Drivers

A driver can have multiple vehicles over time:

``` text
driver 1:N vehicles
```

Example:

``` text
Driver
 ├── Car A
 └── Bike B
```

Business rules may later restrict how many vehicles can be active at
once.

------------------------------------------------------------------------

## Deliveries

Delivery is modeled separately from orders:

``` text
orders
   │
   ▼
deliveries
   ├── driver_id
   └── vehicle_id
```

This is useful because delivery is a distinct business event with its
own status and timestamps.

It also preserves information about which driver and vehicle handled an
order.

------------------------------------------------------------------------

# 46. Quick SQL Cheat Sheet

## Create

``` sql
CREATE TABLE table_name (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);
```

## Insert

``` sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2);
```

## Read

``` sql
SELECT column1, column2
FROM table_name;
```

## Filter

``` sql
SELECT *
FROM table_name
WHERE condition;
```

## Update

``` sql
UPDATE table_name
SET column1 = value
WHERE id = 1;
```

## Delete rows

``` sql
DELETE FROM table_name
WHERE id = 1;
```

## Add column

``` sql
ALTER TABLE table_name
ADD COLUMN new_column TEXT;
```

## Remove column

``` sql
ALTER TABLE table_name
DROP COLUMN new_column;
```

## Remove all rows

``` sql
TRUNCATE TABLE table_name;
```

## Remove table

``` sql
DROP TABLE table_name;
```

## Count

``` sql
SELECT COUNT(*)
FROM table_name;
```

## Group

``` sql
SELECT column, COUNT(*)
FROM table_name
GROUP BY column;
```

## Join

``` sql
SELECT *
FROM A
JOIN B
    ON A.id = B.a_id;
```

## Left join

``` sql
SELECT *
FROM A
LEFT JOIN B
    ON A.id = B.a_id;
```

## Subquery

``` sql
SELECT *
FROM A
WHERE value > (
    SELECT AVG(value)
    FROM A
);
```

## CTE

``` sql
WITH result AS (
    SELECT *
    FROM A
)
SELECT *
FROM result;
```

## Transaction

``` sql
BEGIN;

-- operations

COMMIT;
```

Undo:

``` sql
ROLLBACK;
```

## Index

``` sql
CREATE INDEX idx_name
ON table_name(column_name);
```

## Explain

``` sql
EXPLAIN ANALYZE
SELECT *
FROM table_name
WHERE column_name = value;
```

------------------------------------------------------------------------

# 47. Common Mistakes

## 1. Forgetting WHERE in UPDATE

Dangerous:

``` sql
UPDATE users
SET phone = '123';
```

This updates every row.

Better:

``` sql
UPDATE users
SET phone = '123'
WHERE id = 1;
```

------------------------------------------------------------------------

## 2. Forgetting WHERE in DELETE

Dangerous:

``` sql
DELETE FROM users;
```

This deletes all rows.

------------------------------------------------------------------------

## 3. Confusing DELETE, TRUNCATE and DROP

Remember:

``` text
DELETE
→ remove selected/all rows

TRUNCATE
→ quickly remove all rows

DROP
→ remove the table/object itself
```

------------------------------------------------------------------------

## 4. Using = NULL

Wrong:

``` sql
WHERE phone = NULL;
```

Correct:

``` sql
WHERE phone IS NULL;
```

------------------------------------------------------------------------

## 5. Using current menu price for historical orders

Wrong concept:

``` sql
menu_items.price
```

for old orders.

Correct:

``` sql
order_items.price
```

because it represents the historical purchase price.

------------------------------------------------------------------------

## 6. Adding unnecessary duplicated foreign keys

If:

``` text
menu_item
   ↓
category
   ↓
restaurant
```

then adding both:

``` text
menu_item.category_id
menu_item.restaurant_id
```

may duplicate the relationship unnecessarily.

------------------------------------------------------------------------

## 7. Forgetting the junction table for M:N

Don't store:

``` text
menu_items = '1,2,3'
```

inside an order.

Use:

``` text
order_items
```

instead.

------------------------------------------------------------------------

## 8. Thinking foreign keys automatically create indexes

In PostgreSQL, a foreign key does not automatically mean the referencing
column has an index.

For frequently queried relationships, create appropriate indexes
yourself.

------------------------------------------------------------------------

## 9. Thinking B-tree means binary tree

B-tree means **balanced multi-way tree**, not binary tree.

------------------------------------------------------------------------

## 10. Assuming indexes are always faster

Indexes have costs:

-   disk space
-   additional write overhead
-   maintenance
-   planner overhead

PostgreSQL chooses between plans based on estimated cost.

------------------------------------------------------------------------

# Final Mental Model

When designing a relational database, think in this order:

``` text
1. What are my entities?
        ↓
2. What attributes does each entity have?
        ↓
3. What uniquely identifies each entity?
        ↓
4. How are entities related?
        ↓
5. What are the cardinalities?
        ↓
6. Which relationships need foreign keys?
        ↓
7. What constraints enforce business rules?
        ↓
8. Is the design normalized appropriately?
        ↓
9. What queries will be common?
        ↓
10. Which indexes support those queries?
        ↓
11. How should transactions protect important operations?
```

For the Foodora project:

``` text
USER
 │
 ├── ADDRESS
 │
 └── ORDER
       │
       ├── ORDER_ITEM ── MENU_ITEM ── CATEGORY ── RESTAURANT
       │
       ├── PAYMENT
       │
       └── DELIVERY ── DRIVER ── VEHICLE
```

This diagram is the core mental model for the project.

------------------------------------------------------------------------

## What you should be able to explain before moving forward

You should be comfortable explaining:

-   DB vs DBMS vs RDBMS
-   tables, rows, columns, schemas
-   primary keys
-   foreign keys
-   UNIQUE, NOT NULL, CHECK, DEFAULT
-   NULL
-   common PostgreSQL data types
-   CRUD
-   INSERT / SELECT / UPDATE / DELETE
-   ALTER / DROP / TRUNCATE
-   WHERE / ORDER BY / LIMIT
-   aggregate functions
-   GROUP BY / HAVING
-   JOIN types
-   one-to-one, one-to-many, many-to-many
-   junction tables
-   subqueries
-   CTEs
-   window functions
-   views
-   recursive CTEs
-   functional dependencies
-   normalization
-   1NF / 2NF / 3NF / BCNF
-   transactions
-   ACID
-   isolation
-   locks and deadlocks
-   indexes
-   B-tree indexes
-   composite indexes
-   partial indexes
-   EXPLAIN / EXPLAIN ANALYZE
-   why historical data sometimes needs to be stored separately
-   how to design relationships without unnecessary duplication
