# Types of Databases

---

## Table of Contents

1. [Relational Databases](#1-relational-databases)
2. [Object-Oriented Databases](#2-object-oriented-databases)
3. [NoSQL Databases](#3-nosql-databases)
4. [Hierarchical Databases](#4-hierarchical-databases)
5. [Network Databases](#5-network-databases)
6. [Quick Comparison](#6-quick-comparison)

---

## 1. Relational Databases

Relational databases are based on the **Relational Model**. Even though this model was designed back in the 1970s, it is still one of the most widely used approaches today. Relational databases are also known as **RDBMS (Relational Database Management Systems)**.

- They commonly use **SQL (Structured Query Language)** for operations like creating, reading, updating, and deleting data (this is often remembered as **CRUD**).
- Data is stored in **discrete tables**, and these tables can be **JOINed** together using fields known as **foreign keys**.

**Example:** A `Users` table containing information about all users can be joined to a `Purchases` table containing information about all purchases, using a common field like `user_id`.

```
 Users Table                    Purchases Table
 ┌─────────┬──────────┐         ┌────────────┬─────────┬────────┐
 │ user_id │ name     │         │ purchase_id│ user_id │ amount │
 ├─────────┼──────────┤         ├────────────┼─────────┼────────┤
 │   1     │ Alice    │  <----> │    101     │    1    │  500   │
 │   2     │ Bob      │  <----> │    102     │    2    │  250   │
 └─────────┴──────────┘         └────────────┴─────────┴────────┘
        (joined using the "user_id" foreign key)
```

**Popular examples:** MySQL, Microsoft SQL Server, Oracle.

### Advantages

- Ubiquitous — has a steady user base since the 1970s.
- Highly optimized for working with **structured data**.
- Provides a stronger guarantee of **data normalization**.
- Uses a well-known querying language (SQL).

### Disadvantages

- Scalability issues, especially with **horizontal scaling**.
- As data grows huge, the system tends to become more complex.

---

## 2. Object-Oriented Databases

The **object-oriented data model** is based on the object-oriented programming paradigm, which is widely used in modern software development.

- Key OOP concepts like **inheritance**, **object-identity**, and **encapsulation** (information hiding) — along with methods that provide an interface to objects — carry over into this data model.
- It also supports a rich type system, including **structured and collection types**.
- While inheritance and complex types are also present in the E-R model, it is **encapsulation and object-identity** that distinguish the object-oriented data model from the E-R model.
- In an object-oriented database, data is treated as an **object** — all related information comes packaged together in one instantly available object, instead of being spread across multiple tables.

**Example (conceptual):**

```
 Object: Car
 ┌───────────────────────────────┐
 │ attributes: color, model, year│
 │ methods: start(), stop()      │
 │ (all bundled into one object) │
 └───────────────────────────────┘
```

### Advantages

1. Data storage and retrieval is **easy and quick**.
2. Can handle **complex data relations** and a wider variety of data types compared to standard relational databases.
3. Relatively friendly for modeling **advanced real-world problems**.
4. Works well with the functionality of OOP and object-oriented languages.

### Disadvantages

1. High complexity can cause **performance issues** — read, write, update, and delete operations may slow down.
2. Not much community support, since it isn't as widely adopted as relational databases.
3. Does **not support views**, unlike relational databases.
4. Since a database can be very complex with multiple relations, maintaining relationships between objects can be tedious.

**Examples:** ObjectDB, GemStone.

---

## 3. NoSQL Databases

**NoSQL** (also read as "**not only SQL**") databases are **non-tabular** — they store data differently from how relational tables do. NoSQL databases come in several types depending on their data model: **document, key-value, wide-column, and graph**.

### Key Characteristics

- **Schema-free** — no rigid, predefined schema is required.
- Data structures are **flexible** and can adjust dynamically.
- Can handle **huge amounts of data** (big data).
- Most NoSQL databases are **open-source** and support **horizontal scaling**.
- Simply put: data is stored in some format **other than relational** (tables).

```
 Document Store Example (e.g., MongoDB)

 {
   "user_id": 1,
   "name": "Alice",
   "purchases": [
      { "item": "Laptop", "amount": 500 },
      { "item": "Mouse", "amount": 20 }
   ]
 }
```

> Note: Detailed coverage of NoSQL types (document, key-value, wide-column, graph) is available in the earlier lecture notes (LEC-15).

---

## 4. Hierarchical Databases

As the name suggests, the **hierarchical database model** works best for use cases where information naturally follows a **concrete hierarchy** — for example, several employees reporting to a single department in a company.

- The schema is defined by a **tree-like organization**: there is typically a root **"parent"** that links to various **subdirectory branches**, and each branch (child record) may further link to other branches.
- **Rule:** a parent record can have **several child records**, but each **child record can only have one parent record**.
- Data within records is stored as **fields**, and each field can hold only **one value**.
- Retrieving data requires **traversing the entire tree**, starting from the root node.
- Since disk storage systems are also inherently hierarchical, this model can also serve as a **physical model**.

```
                     Company
                        │
        ┌───────────────┼───────────────┐
        │               │               │
      Sales          Engineering       HR
        │               │
   ┌────┴────┐     ┌────┴────┐
 Emp A     Emp B  Emp C     Emp D
```

### Advantages

- **Ease of use** — the one-to-many organization makes traversal simple and fast.
- Ideal for use cases like website **drop-down menus** or **folder structures** in systems like Microsoft Windows OS.
- Since tables are separated from physical storage structures, information can be added or deleted **without affecting the whole database**.
- Most major programming languages offer built-in support for reading tree-structured databases.

### Disadvantages

- **Inflexible nature** — the one-to-many structure cannot describe relationships where a child node has **multiple parent nodes**.
- Requires **top-to-bottom sequential searching**, which is time-consuming.
- Often requires **repetitive storage** of data across multiple entities, leading to redundancy.

**Example:** IBM IMS.

---

## 5. Network Databases

The **network database model** is an extension of the hierarchical database model.

- Unlike hierarchical databases, **child records are free to associate with multiple parent records**.
- Data is organized in a **graph structure** rather than a strict tree.
- Can handle **complex relationships** more effectively than hierarchical databases.

```
        Parent A        Parent B
            \             /
             \           /
              \         /
               Child C
         (linked to multiple parents)
```

### Advantages

- Can represent **complex relations** (many-to-many) that hierarchical databases cannot.

### Disadvantages

- **Maintenance is tedious**.
- **M:N (many-to-many) links** may cause slow data retrieval.
- Not much web community support.

**Examples:** Integrated Data Store (IDS), IDMS (Integrated Database Management System), Raima Database Manager, TurboIMAGE.

---

## 6. Quick Comparison

| Database Type          | Data Structure        | Best Suited For                          | Key Limitation                          |
|-------------------------|------------------------|-------------------------------------------|-------------------------------------------|
| Relational               | Tables (rows/columns) | Structured data, standard business apps  | Horizontal scaling is harder              |
| Object-Oriented          | Objects                | Complex, real-world entities (OOP apps)  | Performance issues at high complexity     |
| NoSQL                    | Document/Key-Value/Wide-Column/Graph | Big data, flexible/unstructured data | Weaker consistency guarantees in some types |
| Hierarchical             | Tree structure         | Strict one-to-many relationships (e.g., folders) | Cannot model multiple parents         |
| Network                  | Graph structure        | Many-to-many relationships                | Tedious maintenance, slower retrieval     |

