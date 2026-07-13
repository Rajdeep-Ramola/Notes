# SQL

---

## Table of Contents

1. [Introduction to SQL](#1-introduction-to-sql)
   - 1.1 [CRUD Operations](#11-crud-operations)
   - 1.2 [RDBMS](#12-rdbms)
   - 1.3 [SQL vs MySQL](#13-sql-vs-mysql)
2. [SQL Data Types](#2-sql-data-types)
   - 2.1 [Common Data Types](#21-common-data-types)
   - 2.2 [Signed and Unsigned Data Types](#22-signed-and-unsigned-data-types)
   - 2.3 [Advanced Data Types](#23-advanced-data-types)
3. [Types of SQL Commands](#3-types-of-sql-commands)
4. [Managing Databases (DDL)](#4-managing-databases-ddl)
5. [Data Retrieval Language (DRL / DQL)](#5-data-retrieval-language-drl--dql)
   - 5.1 [SELECT and DUAL Tables](#51-select-and-dual-tables)
   - 5.2 [WHERE](#52-where)
   - 5.3 [BETWEEN](#53-between)
   - 5.4 [IN](#54-in)
   - 5.5 [AND / OR / NOT](#55-and--or--not)
   - 5.6 [IS NULL](#56-is-null)
   - 5.7 [Pattern Searching / Wildcards](#57-pattern-searching--wildcards)
   - 5.8 [ORDER BY](#58-order-by)
   - 5.9 [GROUP BY](#59-group-by)
   - 5.10 [DISTINCT](#510-distinct)
   - 5.11 [GROUP BY ... HAVING](#511-group-by--having)
   - 5.12 [WHERE vs HAVING](#512-where-vs-having)
6. [Constraints (DDL)](#6-constraints-ddl)
   - 6.1 [Primary Key](#61-primary-key)
   - 6.2 [Foreign Key](#62-foreign-key)
   - 6.3 [Unique](#63-unique)
   - 6.4 [Check](#64-check)
   - 6.5 [Default](#65-default)
7. [ALTER Operations](#7-alter-operations)
8. [Data Manipulation Language (DML)](#8-data-manipulation-language-dml)
   - 8.1 [INSERT](#81-insert)
   - 8.2 [UPDATE](#82-update)
   - 8.3 [DELETE](#83-delete)
   - 8.4 [REPLACE](#84-replace)
9. [Joining Tables](#9-joining-tables)
   - 9.1 [INNER JOIN](#91-inner-join)
   - 9.2 [OUTER JOIN](#92-outer-join)
   - 9.3 [CROSS JOIN](#93-cross-join)
   - 9.4 [SELF JOIN](#94-self-join)
   - 9.5 [Join Without Using JOIN Keyword](#95-join-without-using-join-keyword)
10. [Set Operations](#10-set-operations)
    - 10.1 [UNION](#101-union)
    - 10.2 [INTERSECT](#102-intersect)
    - 10.3 [MINUS](#103-minus)
    - 10.4 [JOIN vs SET Operations](#104-join-vs-set-operations)
11. [Sub Queries](#11-sub-queries)
    - 11.1 [Types of Sub Queries by Clause](#111-types-of-sub-queries-by-clause)
    - 11.2 [Co-related Sub Queries](#112-co-related-sub-queries)
    - 11.3 [JOIN vs SUB-QUERIES](#113-join-vs-sub-queries)
12. [MySQL Views](#12-mysql-views)

---

## 1. Introduction to SQL

**SQL (Structured Query Language)** is the language used to access and manipulate data in a database.

- SQL is **not a database itself** — it is a query language used by RDBMS software to interact with the data.
- RDBMS examples that use SQL: **MySQL, Oracle, MS SQL Server, MS Access, IBM DB2**, etc.

---

### 1.1 CRUD Operations

SQL uses **CRUD operations** to communicate with the database:

| Operation | Meaning |
|---|---|
| **C**REATE | Execute `INSERT` statements to insert a new tuple into a relation |
| **R**EAD | Read data already present in the relation |
| **U**PDATE | Modify data already inserted into the relation |
| **D**ELETE | Delete a specific data point/tuple/row, or multiple rows |

---

### 1.2 RDBMS

**RDBMS (Relational Database Management System)** is software that lets us implement a designed relational model.

- Examples: MySQL, MS SQL, Oracle, IBM, etc.
- **Table / Relation** is the simplest form of data storage object in a relational database.
- MySQL is an **open-source RDBMS** and uses SQL for all CRUD operations.
- MySQL uses a **client–server model**, where the client is a CLI or frontend that consumes services provided by the MySQL server.

---

### 1.3 SQL vs MySQL

| SQL | MySQL |
|---|---|
| A Structured Query Language used to perform CRUD operations in a relational database | An RDBMS used to store, manage, and administrate a database, using SQL internally |

---

## 2. SQL Data Types

In an SQL database, data is stored in the form of **tables**, and each column has a specific data type.

### 2.1 Common Data Types

| Data Type | Description |
|---|---|
| `CHAR` | Fixed-length string (0–255). Occupies the full defined size even if actual data is smaller |
| `VARCHAR` | Variable-length string (0–255). Occupies space equal to the actual data size |
| `TINYTEXT` | String (0–255) |
| `TEXT` | String (0–65,535) |
| `BLOB` | Binary Large Object — used to store files (audio, video, etc.) in byte form (0–65,535) |
| `MEDIUMTEXT` | String (0–16,777,215) |
| `MEDIUMBLOB` | Binary data (0–16,777,215) |
| `LONGTEXT` | String (0–4,294,967,295) |
| `LONGBLOB` | Binary data (0–4,294,967,295) |
| `TINYINT` | Integer (-128 to 127) |
| `SMALLINT` | Integer (-32768 to 32767) |
| `MEDIUMINT` | Integer (-8388608 to 8388607) |
| `INT` | Integer (-2147483648 to 2147483647) |
| `BIGINT` | Integer (-9223372036854775808 to 9223372036854775807) |
| `FLOAT` | Decimal, precision up to 23 digits |
| `DOUBLE` | Decimal, precision 24–53 digits |
| `DECIMAL` | Double stored as a string |
| `DATE` | `YYYY-MM-DD` |
| `DATETIME` | `YYYY-MM-DD HH:MM:SS` — stores date **and** time |
| `TIMESTAMP` | `YYYYMMDDHHMMSS` |
| `TIME` | `HH:MM:SS` |
| `ENUM` | Categorical data — one of a set of preset values |
| `SET` | One **or many** of a set of preset values |
| `BOOLEAN` | 0 / 1 |
| `BIT` | `BIT(n)`, n up to 64 — stores values in bits |

**Notes:**
- Size ordering: `TINY` < `SMALL` < `MEDIUM` < `INT` < `BIGINT`
- Variable-length types like `VARCHAR` are generally preferred since they occupy space equal to the actual data size, instead of always reserving the maximum.

---

### 2.2 Signed and Unsigned Data Types

Signed/unsigned is generally used with numeric data types to control whether a column can store negative values.

- A **signed** column can store both positive and negative numbers.
- An **unsigned** column can only store non-negative (0 and positive) numbers, effectively doubling the positive range.

```sql
-- Signed column (default): -128 to 127
age TINYINT

-- Unsigned column: 0 to 255
age TINYINT UNSIGNED
```

---

### 2.3 Advanced Data Types

- **`JSON`** — used to store JSON-formatted data directly in a column.

> **Note:** Table schema can also be imported/exported from files (`.csv` or `.json`).

---

## 3. Types of SQL Commands

SQL commands are grouped into five categories:

1. **DDL (Data Definition Language)** — defines the relation schema.
   - `CREATE` — create table, DB, view
   - `ALTER` — modify table structure (change column datatype, add/remove columns)
   - `DROP` — delete table, DB, view
   - `TRUNCATE` — remove all tuples from a table
   - `RENAME` — rename DB, table, or column name

2. **DRL / DQL (Data Retrieval / Data Query Language)** — retrieves data from tables.
   - `SELECT`

3. **DML (Data Manipulation Language)** — performs modifications in the database.
   - `INSERT` — insert data into a relation
   - `UPDATE` — update relation data
   - `DELETE` — delete row(s) from a relation

4. **DCL (Data Control Language)** — grants or revokes authority from a user.
   - `GRANT` — give access privileges to the DB
   - `REVOKE` — revoke a user's access privileges

5. **TCL (Transaction Control Language)** — manages transactions in the DB.
   - `START TRANSACTION` — begin a transaction
   - `COMMIT` — apply all changes and end the transaction
   - `ROLLBACK` — discard changes and end the transaction
   - `SAVEPOINT` — a checkpoint within a group of transactions to roll back to

---

## 4. Managing Databases (DDL)

```sql
-- Create a database
CREATE DATABASE IF NOT EXISTS db_name;

-- Select which database subsequent commands (CREATE TABLE, etc.) run against
-- Also used to switch between databases
USE db_name;

-- Delete a database
DROP DATABASE IF EXISTS db_name;

-- List all databases on the server
SHOW DATABASES;

-- List tables in the selected database
SHOW TABLES;
```

---

## 5. Data Retrieval Language (DRL / DQL)

### 5.1 SELECT and DUAL Tables

**Syntax:**
```sql
SELECT <set of column names> FROM <table_name>;
```

- The order of execution is generally from **right to left** (`FROM` is conceptually resolved before `SELECT`).
- **Can `SELECT` be used without `FROM`?** Yes — using **DUAL tables**. Dual tables are dummy/inbuilt tables MySQL uses internally, letting a user perform simple actions without referencing a user-defined table.

```sql
SELECT 40 + 70;      -- returns 110, in an unnamed result table
SELECT now();        -- returns the current timestamp
SELECT ucase('abc'); -- converts text to uppercase
```

---

### 5.2 WHERE

Reduces rows based on a given condition.

```sql
SELECT * FROM customer WHERE age > 18;
```

---

### 5.3 BETWEEN

```sql
SELECT * FROM customer WHERE age BETWEEN 0 AND 100;
```
Both `0` and `100` are **inclusive**.

---

### 5.4 IN

Reduces the need for multiple `OR` conditions.

```sql
SELECT * FROM officers
WHERE officer_name IN ('Lakshay', 'Maharana Pratap', 'Deepika');
```

---

### 5.5 AND / OR / NOT

```sql
-- AND
SELECT * FROM customer WHERE cond1 AND cond2;

-- OR
SELECT * FROM customer WHERE cond1 OR cond2;

-- NOT
SELECT * FROM customer WHERE col_name NOT IN (1, 2, 3, 4);
```

---

### 5.6 IS NULL

Checks for entries that are null in the table.

```sql
SELECT * FROM customer WHERE prime_status IS NULL;
```

---

### 5.7 Pattern Searching / Wildcards

Uses `%` and `_`, similar to regex:

- `%` — any number of characters, from 0 to n (like `*` in regex)
- `_` — exactly one character

```sql
SELECT * FROM customer WHERE name LIKE '%p_';
-- name can have any number of characters before "p",
-- but exactly one character after it
```

---

### 5.8 ORDER BY

Sorts the data retrieved (optionally after a `WHERE` clause).

```sql
SELECT * FROM customer ORDER BY name DESC;
-- DESC = Descending, ASC = Ascending
```

---

### 5.9 GROUP BY

Collects data from multiple records and groups the result by one or more columns. Generally used with **aggregation functions**:

- `COUNT()`
- `SUM()`
- `AVG()`
- `MIN()`
- `MAX()`

```sql
SELECT department, AVG(salary)
FROM worker
GROUP BY department;
```

> All column names mentioned in the `SELECT` statement should also appear in `GROUP BY` for the query to execute successfully.

```sql
SELECT c1, c2, c3
FROM sample_table
WHERE cond
GROUP BY c1, c2, c3;
```

---

### 5.10 DISTINCT

Finds unique (non-repetitive) values in a table.

```sql
SELECT DISTINCT column_name FROM table_name;
```

`GROUP BY` can achieve the same result:

```sql
SELECT column_name FROM table_name GROUP BY column_name;
```

> SQL is smart enough to realize that `GROUP BY` used **without** any aggregation function effectively means "give me distinct values."

---

### 5.11 GROUP BY ... HAVING

`HAVING` filters the **groups** produced by `GROUP BY`, similar to how `WHERE` filters rows.

```sql
SELECT COUNT(cust_id), country
FROM customer
GROUP BY country
HAVING COUNT(cust_id) > 50;
```

---

### 5.12 WHERE vs HAVING

Both filter rows based on a condition, but:

| WHERE | HAVING |
|---|---|
| Filters rows from the table based on a condition | Filters rows from the **groups** based on a condition |
| Used **before** `GROUP BY` | Used **after** `GROUP BY` |
| Can be used with `SELECT`, `UPDATE`, `DELETE` | Used only with `SELECT`; requires `GROUP BY` to be present |

---

## 6. Constraints (DDL)

### 6.1 Primary Key

- A **Primary Key (PK)** is **not null**, **unique**, and there can be only **one PK per table**.

```sql
CREATE TABLE customer (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);
```

---

### 6.2 Foreign Key

- A **Foreign Key (FK)** refers to the Primary Key of another table.
- A relation can have any number of foreign keys.
- An attribute can be **both a PK and an FK** in a table.

```sql
CREATE TABLE ORDER (
    id INT PRIMARY KEY,
    delivery_date DATE,
    order_placed_date DATE,
    cust_id INT,
    FOREIGN KEY (cust_id) REFERENCES customer(id)
);
```

---

### 6.3 Unique

- Every value in a `UNIQUE` column must be unique, but the column **can be null**.
- A table can have multiple `UNIQUE` attributes.

```sql
CREATE TABLE customer (
    email VARCHAR(1024) UNIQUE
);
```

---

### 6.4 Check

Applies a conditional constraint while inserting a value into a column.

```sql
CREATE TABLE customer (
    age INT,
    CONSTRAINT age_check CHECK (age > 12)
);
```

> The constraint name (`age_check`) is optional — MySQL will auto-generate one if omitted.

---

### 6.5 Default

Sets a default value for a column.

```sql
CREATE TABLE account (
    saving_rate DOUBLE NOT NULL DEFAULT 4.25
);
```

---

## 7. ALTER Operations

`ALTER` operations change the schema of an existing table.

```sql
-- ADD: add new column(s)
ALTER TABLE customer ADD age INT NOT NULL;

-- MODIFY: change the datatype of an existing attribute
ALTER TABLE customer MODIFY name CHAR(1024);

-- CHANGE COLUMN: rename a column (and optionally its datatype)
ALTER TABLE customer CHANGE COLUMN name customer_name VARCHAR(1024);

-- DROP COLUMN: drop a column completely
ALTER TABLE customer DROP COLUMN middle_name;

-- RENAME: rename the table itself
ALTER TABLE customer RENAME TO customer_details;
```

---

## 8. Data Manipulation Language (DML)

### 8.1 INSERT

```sql
INSERT INTO table_name (col1, col2, col3)
VALUES (v1, v2, v3), (val1, val2, val3);
```

---

### 8.2 UPDATE

```sql
UPDATE table_name SET col1 = 1, col2 = 'abc' WHERE id = 1;

-- Updating multiple rows at once
UPDATE student SET standard = standard + 1;
```

**`ON UPDATE CASCADE`** — can be added to a foreign key constraint. Suppose the PK of one table is used as the FK of another table; if the PK of the first table is updated, `ON UPDATE CASCADE` automatically updates the matching FK in the second table.

---

### 8.3 DELETE

```sql
DELETE FROM table_name WHERE id = 1;

-- Delete all rows
DELETE FROM table_name;
```

**`DELETE CASCADE`** — handles what happens to child rows when a parent row is deleted (overcomes the referential-constraint problem).

```sql
CREATE TABLE ORDER (
    order_id INT PRIMARY KEY,
    delivery_date DATE,
    cust_id INT,
    FOREIGN KEY (cust_id) REFERENCES customer(id) ON DELETE CASCADE
);
```

**`ON DELETE SET NULL`** — when the parent's row (PK) is deleted, the FK in the child table is set to `NULL` instead of the row being deleted.

```sql
CREATE TABLE ORDER (
    order_id INT PRIMARY KEY,
    delivery_date DATE,
    cust_id INT,
    FOREIGN KEY (cust_id) REFERENCES customer(id) ON DELETE SET NULL
);
```

---

### 8.4 REPLACE

Used for a tuple that may already be present in a table:

- Behaves like `UPDATE` (using `WHERE` on the PK) if the row already exists.
- Behaves like `INSERT` if there is no duplicate data — a new tuple is inserted.

```sql
REPLACE INTO student (id, class) VALUES (4, 3);

REPLACE INTO table_name SET col1 = val1, col2 = val2;
```

---

## 9. Joining Tables

All RDBMSs are relational in nature — we refer to other tables to get meaningful outcomes, and **foreign keys** are used to reference them. To apply a join, there must be a common attribute between the tables involved.

### 9.1 INNER JOIN

Returns a resultant table with matching values from all the joined tables — conceptually like an intersection (**A ∩ B**).

```sql
SELECT column_list
FROM table1
INNER JOIN table2 ON condition1
INNER JOIN table3 ON condition2;
```

**Alias (`AS`)** — gives a temporary name to a table or column for the duration of a query, making it shorter and more readable.

```sql
SELECT col_name AS alias_name FROM table_name;
SELECT col_name1, col_name2 FROM table_name AS alias_name;
```

---

### 9.2 OUTER JOIN

**LEFT JOIN** — returns all data from the left table and the matched data from the right table; unmatched right-side columns return `NULL`.

```sql
SELECT columns FROM table1 LEFT JOIN table2 ON join_condition;
```

**RIGHT JOIN** — returns all data from the right table and the matched data from the left table.

```sql
SELECT columns FROM table1 RIGHT JOIN table2 ON join_condition;
```

**FULL JOIN** — returns all data with a match on either side. MySQL has no native `FULL JOIN`, so it is emulated with `LEFT JOIN UNION RIGHT JOIN`.

```sql
SELECT columns FROM table1 AS t1 LEFT JOIN table2 AS t2 ON t1.id = t2.id
UNION
SELECT columns FROM table1 AS t1 RIGHT JOIN table2 AS t2 ON t1.id = t2.id;
```

> `UNION ALL` can also be used — this keeps duplicate values, whereas plain `UNION` returns only unique rows.

---

### 9.3 CROSS JOIN

Returns the **Cartesian product** of the data in both tables — every possible combination of rows.

```sql
SELECT column_list FROM table1 CROSS JOIN table2;
```

If table1 has 10 rows and table2 has 5 rows, the result has 50 rows. Rarely used in practice.

---

### 9.4 SELF JOIN

Used to compare rows within the same table, i.e. a table joined to itself. Rarely used; emulated using `INNER JOIN` with table aliases.

```sql
SELECT columns FROM table AS t1
INNER JOIN table AS t2 ON t1.id = t2.id;
```

---

### 9.5 Join Without Using JOIN Keyword

```sql
SELECT * FROM table1, table2 WHERE condition;

-- Example
SELECT artist_name, album_name, year_recorded
FROM artist, album
WHERE artist.id = album.artist_id;
```

---

## 10. Set Operations

Set operations combine results **row-wise** (vertically) from two or more `SELECT` statements, whereas `JOIN` combines tables **column-wise** (horizontally) based on a matching condition. For set operations, the number and order of columns — and their data types — must match across the queries. Set operations always return **distinct** rows (except `UNION ALL`).

| JOIN | SET Operations |
|---|---|
| Combines multiple tables based on a matching condition | Combines the result sets of two or more `SELECT` statements |
| Column-wise combination | Row-wise combination |
| Data types of the two tables can differ | Data types of corresponding columns must be the same |
| Can generate duplicate or distinct rows | Always generates distinct rows |
| Number of selected columns from each table may differ | Number of selected columns must be the same from each query |

---

### 10.1 UNION

Combines two or more `SELECT` statements, keeping only unique rows. Column count and order must match.

```sql
SELECT * FROM table1
UNION
SELECT * FROM table2;
```

---

### 10.2 INTERSECT

Returns the values common to both tables. MySQL has no native `INTERSECT`, so it is emulated:

```sql
SELECT DISTINCT column_list FROM table1
INNER JOIN table2 USING (join_col);
```

---

### 10.3 MINUS

Returns the distinct rows from the first table that do **not** occur in the second table. Also emulated in MySQL:

```sql
SELECT column_list FROM table1
LEFT JOIN table2 ON condition
WHERE table2.column_name IS NULL;

-- Example
SELECT id FROM table1
LEFT JOIN table2 USING (id)
WHERE table2.id IS NULL;
```

---

### 10.4 JOIN vs SET Operations

- **JOIN** is generally faster and keeps the responsibility of calculation on the DBMS engine; choosing the optimal join can be tricky.
- **SET operations** are comparatively easier to understand and implement, though they place more of the calculation burden on the user's query design.

---

## 11. Sub Queries

A **sub query** (nested query) is a query inside another query, where the outer query generally depends on the result of the inner query. Sub queries are an **alternative to joins**.

```sql
SELECT column_list(s)
FROM table_name
WHERE column_name OPERATOR
    (SELECT column_list(s) FROM table_name [WHERE ...]);

-- Example
SELECT * FROM table1 WHERE col1 IN (SELECT col1 FROM table1);
```

---

### 11.1 Types of Sub Queries by Clause

Sub queries mainly exist in three clauses: **WHERE**, **FROM**, and **SELECT**.

**Sub query inside `FROM`** — requires an alias, since a `FROM` clause expects a table:

```sql
SELECT MAX(rating)
FROM (SELECT * FROM movie WHERE country = 'India') AS temp;
```

**Sub query inside `SELECT`:**

```sql
SELECT (SELECT column_list(s) FROM T1 WHERE condition), column_list(s)
FROM T2
WHERE condition;
```

**Derived sub query** — the entire result of the inner query becomes a new virtual table:

```sql
SELECT column_list(s)
FROM (SELECT column_list(s) FROM table_name WHERE [condition]) AS new_table_name;
```

---

### 11.2 Co-related Sub Queries

With a normal nested sub query, the inner `SELECT` runs first and executes **once**, returning values used by the outer query.

A **co-related sub query**, on the other hand, executes **once for every candidate row** considered by the outer query — the inner query refers to (is driven by) the outer query, rather than the other way around.

---

### 11.3 JOIN vs SUB-QUERIES

| JOINS | SUBQUERIES |
|---|---|
| Faster | Slower |
| Places the calculation burden on the DBMS | Places the responsibility of calculation more on the user |
| Choosing the optimal join can be difficult | Comparatively easy to understand and implement |

---

## 12. MySQL Views

- A **view** is a database object that holds no data of its own — its contents are based on an underlying base table (or tables). It has rows and columns similar to a real table.
- In MySQL, a view is a **virtual table** created by a query, typically joining one or more tables. It behaves like a base table but stores no data itself.
- The key difference from a table: a view is a **definition built on top of other tables (or views)**. If the underlying table changes, those changes are automatically reflected in the view.

```sql
-- Create a view
CREATE VIEW view_name AS
SELECT columns FROM tables [WHERE conditions];

-- Alter a view
ALTER VIEW view_name AS
SELECT columns FROM table WHERE conditions;

-- Drop a view
DROP VIEW IF EXISTS view_name;

-- Example: view built using a JOIN
CREATE VIEW Trainer AS
SELECT c.course_name, c.trainer, t.email
FROM courses c, contact t
WHERE c.id = t.id;
```
