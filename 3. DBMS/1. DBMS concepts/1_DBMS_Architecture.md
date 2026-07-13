# DBMS Architecture

---

## Table of Contents

1. [Abstraction in DBMS](#1-abstraction-in-dbms)
2. [Three-Schema Architecture](#2-three-schema-architecture)
   - 2.1 [Physical Level](#21-physical-level)
   - 2.2 [Logical / Conceptual Level](#22-logical--conceptual-level)
   - 2.3 [View / External Level](#23-view--external-level)
3. [Instance and Schema](#3-instance-and-schema)
4. [Data Model](#4-data-model)
5. [Data Languages](#5-data-languages)
   - 5.1 [DDL – Data Definition Language](#51-ddl--data-definition-language)
   - 5.2 [DML – Data Manipulation Language](#52-dml--data-manipulation-language)
6. [DBMS Application Architecture](#6-dbms-application-architecture)
   - 6.1 [1-Tier Architecture](#61-1-tier-architecture)
   - 6.2 [2-Tier Architecture](#62-2-tier-architecture)
   - 6.3 [3-Tier Architecture](#63-3-tier-architecture)

---

## 1. Abstraction in DBMS

DBMS provides users with an **abstracted view of data** — the system hides details of how data is actually stored (which data structures are used, whether it's stored as tables, hashes, etc.). End users have no business knowing these details.

**Purpose:** To show data selectively to different users.

**Example:** Different departments in a company should only be able to see data relevant to their department, even though all data is stored at one place.

---

## 2. Three-Schema Architecture

The way abstraction is achieved in DBMS is through the **three-schema architecture** — it enables multiple users to access the same data with a personalized view while storing the underlying data only once.

---

### 2.1 Physical Level

- Describes **how data is stored** and which data structure is used.
- Talks about storage allocation, data compression, encryption, etc.
- Goal is to use algorithms which allow **fast access to data**.

---

### 2.2 Logical / Conceptual Level

- Describes the **design of the database at the logical level**.
- Defines what data is stored and what the relationships between that data are.

---

### 2.3 View / External Level

- Provides **different views to different end users**.
- At the external level, a database contains several schemas used to describe different views of the database.
- Also provides **security mechanisms** to prevent users from accessing certain parts of the DB.

---

## 3. Instance and Schema

**Instance** — what data is stored in the database at a particular point in time.

**Schema** — the overall design of the database (basically the logical level schema). It includes:

- Attributes of a table (columns)
- Consistency constraints — e.g., no attribute should be null, a field shouldn't be more than 20 characters, etc.
- Relationships between tables

---

## 4. Data Model

A **data model** provides a way to describe the design of a database at the logical level — how a table will look, what the relationships between information will be, consistency constraints, etc.

**Example:** ER Model

---

## 5. Data Languages

### 5.1 DDL – Data Definition Language

Used to **define the database schema**.

### 5.2 DML – Data Manipulation Language

Used to **do operations in a database** — to query and update data.

---

## 6. DBMS Application Architecture

### 6.1 1-Tier Architecture

- Client, server, and DB are all on the **same machine**.
- **Example:** Development phase of an application.

---

### 6.2 2-Tier Architecture

- Client machine invokes the DB at the server end through a query.
- Query statements are sent over a network to the server where the DB is running.

---

### 6.3 3-Tier Architecture

The application is divided into 3 logical components:

- The user application does **not communicate with the DB server directly**.
- The user app communicates with the **application server**, which in turn interacts with the **database server**.

**Advantages:**

- **Scalability** — distributed application servers
- **Data integrity** — the app server acts as a middle layer between user and DB, which prevents data corruption
- **Security** — client can't access the DB directly, hence more secure
