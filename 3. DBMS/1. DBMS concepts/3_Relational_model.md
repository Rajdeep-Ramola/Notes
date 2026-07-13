# Relational Model

---

## Table of Contents

1. [Relational Model Basics](#1-relational-model-basics)
   - 1.1 [Degree and Cardinality](#11-degree-and-cardinality)
   - 1.2 [Properties of a Table](#12-properties-of-a-table)
2. [Relational Model Keys](#2-relational-model-keys)
3. [Integrity Constraints](#3-integrity-constraints)
   - 3.1 [Domain Constraints](#31-domain-constraints)
   - 3.2 [Entity Constraints](#32-entity-constraints)
   - 3.3 [Referential Constraints](#33-referential-constraints)
   - 3.4 [Key Constraints](#34-key-constraints)
4. [Transforming ER Model to Relational Model](#4-transforming-er-model-to-relational-model)
   - 4.1 [ER Diagram Notations to Relations](#41-er-diagram-notations-to-relations)

---

## 1. Relational Model Basics

In the relational model, we represent an **entity as a table** and its **attributes as columns** of that table.

> **Note:** First create an ER diagram, then create a relational model.

---

### 1.1 Degree and Cardinality

- **Degree of table** — number of attributes (columns)
- **Cardinality** — number of rows

---

### 1.2 Properties of a Table

- The **name of a relation/table is unique** among all other tables.
- **Values of attributes must be atomic** — they cannot be broken down further.

---

## 2. Relational Model Keys

- **Super Key** — any combination of attributes present in a table which can uniquely identify each row.

- **Candidate Key** — minimum subset of super keys which can uniquely identify each row. It should not contain attributes with redundant values.

- **Primary Key** — selected from the candidate key set; has the least number of attributes.

- **Foreign Key** — used to establish a relation between 2 tables; the primary key of one table is added to another table where it acts as a foreign key.
  - The table whose primary key is being used is called the **referenced (parent) relation**.
  - The table which is using the primary key of the other table is the **referencing (child) relation**.

- **Composite Key** — a primary key formed using at least 2 attributes.

- **Compound Key** — a primary key formed using 2 foreign keys.

- **Surrogate Key** — a system-generated unique identifier used as a table's primary key.
  - It has no business meaning and is used to provide a consistent, unique row identifier regardless of changes in business data or table structure.

---

## 3. Integrity Constraints

Integrity constraints ensure the database doesn't become corrupted.

---

### 3.1 Domain Constraints

Restricts the value of attributes in a relation through **data types or logical conditions**.

- Example: `birth_year > 2020`

---

### 3.2 Entity Constraints

Every relation should have a **primary key**, and that primary key **should not be null**.

---

### 3.3 Referential Constraints

- **Insert Constraint** — we cannot add a row inside a child table if the value of the foreign key of that row is not present in the parent table.

- **Delete Constraint** — a value cannot be deleted from the parent table if it exists in the child table.
  - To solve this: use `ON DELETE CASCADE`.

> **Note:** Can a foreign key have a NULL value? Yes — if the corresponding value in the parent table has been deleted, we can assign the foreign key as NULL.

---

### 3.4 Key Constraints

- **NOT NULL** — by default a column can be null, but we can define a column to be NOT NULL.
- **UNIQUE** — values in that column will be unique.
- **DEFAULT** — defines the default value of a column.
- **CHECK** — limits the value range through a condition.
- **PRIMARY KEY** — uniquely identifies each row; primary key cannot be null.
- **FOREIGN KEY** — keeps the relationship between 2 tables.

---

## 4. Transforming ER Model to Relational Model

---

### 4.1 ER Diagram Notations to Relations

- **Strong Entity** — becomes an individual table with the same entity name; attributes become columns; entity's primary key is used as the relation's primary key.

- **Weak Entity** — a table is formed with all attributes of that entity; the primary key of the strong entity it depends on is added to the weak entity's table; the primary key of the weak entity's table is a **composite PK (FK + partial discriminator key)**.

- **Single-Valued Attributes** — represented as columns directly in the table.

- **Composite Attributes** — handled by creating separate attributes within the same relation.

- **Multivalued Attributes** — a new table is created for the multivalued attribute.
  - The primary key of the entity is used as a foreign key in the new table.
  - A new column is added to store the values of that multivalued attribute.
  - Primary key of the new table = **(FK + new column)**

- **Derived Attributes** — not considered in the table.

- **Generalization** — two methods:
  - **Method 1** — Create a table for the higher-level entity set; create tables for lower-level entity sets with their respective columns and a column that includes the primary key of the higher-level entity set.
  - **Method 2** — Instead of creating a parent entity set table, include the columns of the parent entity set in each of the lower-level entity set tables.
    - *Drawback:* May cause redundancy sometimes.

- **Aggregation** — make a table of the relationship where columns will be the primary keys of other entity tables (i.e., foreign keys).
