# NoSQL Databases

---

## Table of Contents

1. [What is NoSQL](#1-what-is-nosql)
2. [History Behind NoSQL](#2-history-behind-nosql)
3. [NoSQL Databases Advantages](#3-nosql-databases-advantages)
   - 3.1 [Flexible Schema](#31-flexible-schema)
   - 3.2 [Horizontal Scaling](#32-horizontal-scaling)
   - 3.3 [High Availability](#33-high-availability)
   - 3.4 [Easy Insert and Read Operations](#34-easy-insert-and-read-operations)
   - 3.5 [Caching Mechanism](#35-caching-mechanism)
4. [When to Use NoSQL?](#4-when-to-use-nosql)
5. [NoSQL DB Misconceptions](#5-nosql-db-misconceptions)
   - 5.1 [Relationship Data is Best Suited for Relational Databases](#51-relationship-data-is-best-suited-for-relational-databases)
   - 5.2 [NoSQL Databases Don't Support ACID Transactions](#52-nosql-databases-dont-support-acid-transactions)
6. [Types of NoSQL Data Models](#6-types-of-nosql-data-models)
   - 6.1 [Key-Value Stores](#61-key-value-stores)
   - 6.2 [Column-Oriented / Columnar / Wide-Column Stores](#62-column-oriented--columnar--wide-column-stores)
   - 6.3 [Document Based Stores](#63-document-based-stores)
   - 6.4 [Graph Based Stores](#64-graph-based-stores)
7. [NoSQL Databases Dis-advantages](#7-nosql-databases-dis-advantages)
8. [SQL vs NoSQL](#8-sql-vs-nosql)

---

## 1. What is NoSQL

NoSQL databases (aka **"not only SQL"**) are non-tabular databases that store data differently than relational tables. They come in a variety of types based on their data model — the main types being **document, key-value, wide-column, and graph**. NoSQL databases provide flexible schemas and scale easily with large amounts of data and high user loads.

**Key characteristics:**

- **Schema-free** — no rigid, predefined schema is required.
- Data structures used are **not tabular**; they are more flexible and can adjust dynamically.
- Can handle **huge amounts of data** (big data).
- Most NoSQL databases are **open source** and support **horizontal scaling**.
- They simply store data in some format **other than relational**.

---

## 2. History Behind NoSQL

NoSQL databases emerged in the **late 2000s** as the cost of storage dramatically decreased. Gone were the days of needing to create a complex, difficult-to-manage data model just to avoid data duplication. Developers (rather than storage) were becoming the primary cost of software development, so NoSQL databases optimised for **developer productivity**.

- Data was becoming **more unstructured**, so structuring it (defining a schema in advance) became costly.
- NoSQL databases allowed developers to store huge amounts of **unstructured data**, giving a lot of flexibility.
- There was a growing need to **rapidly adapt to changing requirements**. Developers needed the ability to iterate quickly and make changes throughout their software stack — all the way down to the database. NoSQL gave them this flexibility.
- **Cloud computing** rose in popularity, and developers began using public clouds to host applications and data. They wanted the ability to **distribute data across multiple servers and regions** for resilience, to scale out instead of scale up, and to intelligently geo-place their data. Some NoSQL databases, like MongoDB, provide these capabilities.

---

## 3. NoSQL Databases Advantages

### 3.1 Flexible Schema

RDBMS has a pre-defined schema, which becomes an issue when we don't have all the data with us, or when we need to change the schema — it's a huge task to change a schema on the go. NoSQL databases avoid this rigidity.

### 3.2 Horizontal Scaling

- **Horizontal scaling** (scale-out) refers to bringing on additional nodes to share the load. This is difficult with relational databases due to the difficulty in spreading related data across nodes. With non-relational databases, this is simpler since collections are self-contained and not coupled relationally — this allows them to be distributed across nodes more easily, as queries don't have to "join" data together across nodes.
- Scaling horizontally is achieved through **Sharding** or **Replica-sets**.

### 3.3 High Availability

- NoSQL databases are highly available due to their **auto-replication** feature — whenever a failure happens, data replicates itself to the preceding consistent state.
- If a server fails, data can still be accessed from another server, since in a NoSQL database, data is stored across multiple servers.

### 3.4 Easy Insert and Read Operations

Queries in NoSQL databases can be **faster** than SQL databases. Why? Data in SQL databases is typically normalised, so queries for a single object or entity require joining data from multiple tables — as tables grow, joins become expensive. Data in NoSQL databases, however, is typically stored in a way that's optimised for queries.

> **Rule of thumb (MongoDB):** Data that is accessed together should be stored together.

Because of this, queries typically don't require joins and are very fast. However, **delete and update operations are comparatively difficult**.

### 3.5 Caching Mechanism

NoSQL databases support caching mechanisms, which makes them well suited for **cloud applications**.

---

## 4. When to Use NoSQL?

- Fast-paced **Agile development**
- Storage of **structured and semi-structured** data
- **Huge volumes** of data
- Requirements for a **scale-out architecture**
- Modern application paradigms like **micro-services** and **real-time streaming**

---

## 5. NoSQL DB Misconceptions

### 5.1 Relationship Data is Best Suited for Relational Databases

A common misconception is that NoSQL/non-relational databases don't store relationship data well. In reality, NoSQL databases **can** store relationship data — they just store it differently than relational databases do. In fact, compared with relational databases, many find modelling relationship data in NoSQL to be **easier**, because related data doesn't have to be split between tables. NoSQL data models allow related data to be **nested within a single data structure**.

### 5.2 NoSQL Databases Don't Support ACID Transactions

Another common misconception is that NoSQL databases don't support ACID transactions. In fact, some NoSQL databases — like **MongoDB** — do support ACID transactions.

---

## 6. Types of NoSQL Data Models

### 6.1 Key-Value Stores

The simplest type of NoSQL database is a **key-value store**. Every data element in the database is stored as a key-value pair consisting of an attribute name (the "key") and a value. In a sense, a key-value store is like a relational database with only two columns: the key/attribute name (e.g., `state`) and the value (e.g., `Alaska`).

**Example:**

| Key           | Value    |
|---------------|----------|
| state         | Alaska   |
| user_id       | 10234    |
| theme         | dark     |
| cart_item_1   | Laptop   |

- A key-value database associates a value (anything from a number or simple string to a complex object) with a key used to track that object. In its simplest form, it's like a dictionary/array/map object as found in most programming paradigms, but stored persistently and managed by a DBMS.
- Key-value databases use compact, efficient index structures to quickly and reliably locate a value by its key, making them ideal for systems that need to find and retrieve data in **constant time**.
- **Use cases:** shopping carts, user preferences, user profiles.
- **Examples:** Oracle NoSQL, Amazon DynamoDB, MongoDB (also supports key-value), Redis.

**Optimal scenarios for a key-value store:**

a) Real-time random data access — e.g., user session attributes in an online application such as gaming or finance.
b) Caching mechanism for frequently accessed data or configuration based on keys.
c) Applications designed around simple key-based queries.

### 6.2 Column-Oriented / Columnar / C-Store / Wide-Column

- Data is stored such that each row of a column sits next to other rows from that **same column**.
- While a relational database stores data in rows and reads data row by row, a column store is organised as a set of **columns**. This means when running analytics on a small number of columns, you can read those columns directly without consuming memory with unwanted data.
- Columns are often of the same type and benefit from more efficient **compression**, making reads even faster. Columnar databases can quickly aggregate the value of a given column (e.g., adding up total sales for the year).
- **Use cases:** analytics.
- **Examples:** Cassandra, RedShift, Snowflake.

### 6.3 Document Based Stores

- This DB stores data in **documents** similar to JSON (JavaScript Object Notation) objects. Each document contains pairs of fields and values. Values can be a variety of types, including strings, numbers, booleans, arrays, or objects.
- **Use cases:** e-commerce platforms, trading platforms, mobile app development across industries.
- Supports **ACID** properties, hence suitable for transactions.
- **Examples:** MongoDB, CouchDB.

**Example document (JSON-like):**

```json
{
  "user_id": 101,
  "name": "Riya Sharma",
  "orders": [
    { "item": "Laptop", "price": 55000 },
    { "item": "Mouse", "price": 500 }
  ]
}
```

### 6.4 Graph Based Stores

- A graph database focuses on the **relationship between data elements**. Each element is stored as a **node** (e.g., a person in a social media graph). The connections between elements are called **links or relationships**. In a graph database, connections are first-class elements of the database, stored directly — whereas in relational databases, links are implied using data to express relationships.
- Graph databases are optimised to capture and search connections between data elements, overcoming the overhead associated with `JOIN`ing multiple tables in SQL.
- Very few real-world business systems can survive solely on graph queries, so graph databases are usually run alongside other, more traditional databases.
- **Use cases:** fraud detection, social networks, knowledge graphs.
- **Examples:** Neo4j, Amazon Neptune.

---

## 7. NoSQL Databases Dis-advantages

- **Data Redundancy** — Since data models in NoSQL databases are typically optimised for queries and not for reducing data duplication, NoSQL databases can be larger than SQL databases. Storage is currently so cheap that most consider this a minor drawback, and some NoSQL databases also support compression to reduce the storage footprint.
- **Update & Delete operations are costly.**
- **No single NoSQL data model fulfils all application needs** — Depending on the NoSQL database type selected, you may not be able to achieve all your use cases in a single database. For example, graph databases are excellent for analysing relationships but may not provide what you need for everyday retrieval such as range queries. When selecting a NoSQL database, consider your use cases and whether a general-purpose database like MongoDB would be a better option.
- **Doesn't support ACID properties in general.**
- **Doesn't support data entry with consistency constraints.**

---

## 8. SQL vs NoSQL

| Aspect                    | SQL Databases                                                                 | NoSQL Databases                                                                                                                                                 |
|---------------------------|--------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Data Storage Model**    | Tables with fixed rows and columns                                            | Document: JSON documents · Key-value: key-value pairs · Wide-column: tables with rows and dynamic columns · Graph: nodes and edges                          |
| **Development History**   | Developed in the 1970s with a focus on reducing data duplication              | Developed in the late 2000s with a focus on scaling and enabling rapid application change driven by Agile and DevOps practices                               |
| **Examples**               | Oracle, MySQL, Microsoft SQL Server, PostgreSQL                               | Document: MongoDB, CouchDB · Key-value: Redis, DynamoDB · Wide-column: Cassandra, HBase · Graph: Neo4j, Amazon Neptune                                        |
| **Primary Purpose**        | General purpose                                                               | Document: general purpose · Key-value: large amounts of data with simple lookup queries · Wide-column: large amounts of data with predictable query patterns · Graph: analysing and traversing relationships between connected data |
| **Schemas**                 | Fixed                                                                          | Flexible                                                                                                                                                       |
| **Scaling**                | Vertical (scale-up)                                                            | Horizontal (scale-out across commodity servers)                                                                                                               |
| **ACID Properties**        | Supported                                                                      | Not supported, except in databases like MongoDB, etc.                                                                                                          |
| **JOINS**                  | Typically required                                                            | Typically not required                                                                                                                                        |
| **General Purpose**        | Data-to-object mapping requires object-relational mapping (ORM)               | Many do not require ORMs — MongoDB documents map directly to data structures in most popular programming languages                                            |
