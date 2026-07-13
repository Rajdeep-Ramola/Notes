# Partitioning & Sharding in DBMS (DB Optimisation)

> **Revision note:** This version corrects five conceptual blur-points from the original draft — most importantly, that *sharding is a special case of horizontal partitioning, not a synonym for it*. Each correction is called out inline in a **📌 Clarification** box at the point where the original draft was imprecise.

---

## Table of Contents

1. [The Problem: Why We Need DB Optimisation](#1-the-problem-why-we-need-db-optimisation)
2. [Database Optimisation Techniques](#2-database-optimisation-techniques)
   - 2.1 [Scale Up (Vertical Scaling)](#21-scale-up-vertical-scaling)
   - 2.2 [Replica Sets / Clustering](#22-replica-sets--clustering)
   - 2.3 [Partitioning (Scale Out / Horizontal Scaling)](#23-partitioning-scale-out--horizontal-scaling)
3. [What is Partitioning?](#3-what-is-partitioning)
4. [Types of Partitioning](#4-types-of-partitioning)
   - 4.1 [Vertical Partitioning](#41-vertical-partitioning)
   - 4.2 [Horizontal Partitioning](#42-horizontal-partitioning)
5. [The Partitioning Family Tree](#5-the-partitioning-family-tree)
6. [When is Partitioning Applied?](#6-when-is-partitioning-applied)
7. [Advantages of Partitioning](#7-advantages-of-partitioning)
8. [Distributed Database](#8-distributed-database)
9. [Sharding](#9-sharding)
   - 9.1 [What is Sharding](#91-what-is-sharding)
   - 9.2 [The Routing Layer](#92-the-routing-layer)
   - 9.3 [Pros of Sharding](#93-pros-of-sharding)
   - 9.4 [Cons of Sharding](#94-cons-of-sharding)
   - 9.5 [High Availability: Sharding Alone Isn't Enough](#95-high-availability-sharding-alone-isnt-enough)
10. [Quick Reference: Common Misconceptions](#10-quick-reference-common-misconceptions)

---

## 1. The Problem: Why We Need DB Optimisation

Two situations push a database to its limit:

- The **dataset becomes so huge** that managing and querying it becomes a tedious, slow task.
- The **number of incoming requests becomes so large** that a single DB server can't keep up, and the system's response time starts to rise.

The fix for both is to move toward a **distributed architecture** — achieved through DB optimisation techniques like **Clustering, Partitioning, and Sharding**, used alone or in combination (see [Section 8](#8-distributed-database) for why "in combination" matters).

---

## 2. Database Optimisation Techniques

Before jumping to partitioning, it helps to see the full menu of options and why partitioning wins out for very large systems.

### 2.1 Scale Up (Vertical Scaling)

- Simply upgrade the hardware of the existing DB server — more RAM, more CPU, faster disks.
- **Problem:** this gets **expensive** very quickly, and there's always a ceiling on how much a single machine can be upgraded.

### 2.2 Replica Sets / Clustering

- Commonly, we keep one **primary/master node** and multiple **replica nodes**.
- All writes/updates typically happen on the primary node first, and are then **propagated** to the replica nodes.
- **Problem:** this propagation isn't instant — it causes **replication delay/lag**, so replicas can briefly serve stale data.

> **📌 Clarification — not all clustering is primary-replica.** The classic primary→replica model is the most common teaching example, but it isn't the only clustering architecture. Some databases use **multi-master** clusters (more than one node accepts writes) or **shared-nothing** clusters (independent nodes with no shared primary). Treat primary-replica as *one* pattern, not the definition of clustering.

### 2.3 Partitioning (Scale Out / Horizontal Scaling)

- Instead of making one machine bigger (scale up), we **add more nodes/servers** (scale out).
- The database is broken into smaller pieces and spread across nodes — though, as clarified in [Section 3](#3-what-is-partitioning), those pieces don't *have* to leave a single machine to count as "partitioned."

> **Note:** Horizontally scaling relational databases is genuinely difficult because relations (foreign keys, joins) span across tables, and once those tables live on different machines, maintaining those relationships is challenging. Partitioning is precisely the technique that lets us take a large table and split it into structured, manageable slices — whether those slices stay on one server or move across many.

---

## 3. What is Partitioning?

A big problem is much easier to solve once it's broken into smaller sub-problems — partitioning applies this idea to databases.

- Partitioning **divides a big database table** (its data, metrics, and indexes) into smaller, manageable slices called **partitions**.
- Partitioned tables are used **directly by SQL queries without any alteration** — the query layer doesn't need to change.
- Once partitioned, operations act on the smaller partitioned slices instead of having to handle the entire giant table at once — this is what actually cuts down the complexity of managing large tables.

> **📌 Clarification — partitioning does *not* always mean "across separate servers."**
>
> The original claim — *"partitioning is the technique used to divide stored database objects across separate servers"* — overstates it. Partitioning can happen entirely **inside a single database server**.
>
> **Example:** an `Orders` table split by year —
> ```
> Orders (single DB instance)
> ├── Orders_2024
> ├── Orders_2025
> └── Orders_2026
> ```
> All three partitions can live on the exact same machine. A query filtering `WHERE order_date = '2025-05-01'` only has to scan `Orders_2025`, even though nothing left that one server.
>
> **Corrected definition:** *Partitioning is the process of dividing a table into smaller logical pieces. These partitions may reside on the same server or be distributed across multiple servers.* Splitting across multiple servers is a valid and common outcome of partitioning (and it's the case this document focuses on for scale-out) — it just isn't a requirement of the definition itself.

---

## 4. Types of Partitioning

Say we have a single large `Employee` table:

| EmployeeID | Name    | Department | Salary | Email                  | Address              |
|-----------:|---------|------------|-------:|-------------------------|-----------------------|
| 101        | Aditi   | Sales      | 55000  | aditi@company.com       | Delhi, India          |
| 102        | Rohan   | Engineering| 72000  | rohan@company.com       | Pune, India           |
| 103        | Meera   | Sales      | 58000  | meera@company.com       | Mumbai, India         |
| 104        | Karan   | Engineering| 69000  | karan@company.com       | Bangalore, India      |

### 4.1 Vertical Partitioning

- Slices the relation **vertically / column-wise** — different columns of the same table go into different partitions (which may sit on different servers, per the clarification in [Section 3](#3-what-is-partitioning)).
- Because a row's data is now split up, reconstructing a full row requires a **join on the partition key** (here, `EmployeeID`) across partitions.

**Example** — the `Employee` table above split into two vertical partitions:

**Partition A (core identity/work columns):**

| EmployeeID | Name    | Department | Salary |
|-----------:|---------|------------|-------:|
| 101        | Aditi   | Sales      | 55000  |
| 102        | Rohan   | Engineering| 72000  |
| 103        | Meera   | Sales      | 58000  |
| 104        | Karan   | Engineering| 69000  |

**Partition B (contact columns):**

| EmployeeID | Email                  | Address              |
|-----------:|-------------------------|-----------------------|
| 101        | aditi@company.com       | Delhi, India          |
| 102        | rohan@company.com       | Pune, India           |
| 103        | meera@company.com       | Mumbai, India         |
| 104        | karan@company.com       | Bangalore, India      |

To answer "give me Aditi's full record," the DB now has to query **both partitions** and stitch the row back together using `EmployeeID`.

### 4.2 Horizontal Partitioning

- Slices the relation **horizontally / row-wise** — different rows (tuples) of the same table go into different partitions.
- Each partition holds an **independent chunk of complete data tuples** — a row lives entirely in one partition, so no reconstruction is needed to read a single row.

**Example** — the same `Employee` table split by `Department`:

**Partition A (Sales rows):**

| EmployeeID | Name  | Department | Salary | Email              | Address       |
|-----------:|-------|------------|-------:|---------------------|---------------|
| 101        | Aditi | Sales      | 55000  | aditi@company.com   | Delhi, India  |
| 103        | Meera | Sales      | 58000  | meera@company.com   | Mumbai, India |

**Partition B (Engineering rows):**

| EmployeeID | Name  | Department  | Salary | Email              | Address          |
|-----------:|-------|-------------|-------:|---------------------|------------------|
| 102        | Rohan | Engineering | 72000  | rohan@company.com   | Pune, India      |
| 104        | Karan | Engineering | 69000  | karan@company.com   | Bangalore, India |

A query for "Rohan's full record" only ever needs to hit **Partition B**. This row-wise independence is the foundation both **local partitioning** (same server) and **Sharding** ([Section 9](#9-sharding)) are built on — the difference between those two is *where the partitions physically live*, covered next.

---

## 5. The Partitioning Family Tree

> **📌 Clarification — horizontal partitioning and sharding are not the same thing.** This is the single biggest conceptual fix in this revision, and it's a frequently-asked interview distinction.
>
> The original draft stated: *"Sharding is the technique used to implement Horizontal Partitioning in practice."* That's directionally right but oversimplified — it implies the two terms are interchangeable, and they aren't.
>
> **Precise statement:** *Sharding is a distributed form of horizontal partitioning, where the partitions are placed on different database instances/servers.* Horizontal partitioning is the broader category; sharding is one specific way of deploying it.

```
Partitioning
│
├── Vertical Partitioning
│
└── Horizontal Partitioning
     │
     ├── Local Partitioning   (partitions stay on the same DB instance)
     │
     └── Sharding             (partitions are spread across multiple DB instances)
```

**The rule of thumb:**
- Every shard **is** a horizontal partition.
- **Not** every horizontal partition **is** a shard.

If your `Orders_2024` / `Orders_2025` / `Orders_2026` partitions from [Section 3](#3-what-is-partitioning) all sit on one server, that's horizontal partitioning *without* sharding. The moment those partitions move onto separate servers with a routing mechanism in front of them, it becomes sharding.

---

## 6. When is Partitioning Applied?

Partitioning becomes necessary when:

1. The **dataset becomes so huge** that managing and dealing with it becomes a tedious task, even on a single server.
2. The **number of requests becomes large enough** that access latency starts climbing and the system's response time becomes too high.

Depending on severity, either problem can be solved with **local (single-server) partitioning** first; sharding becomes necessary once a single server's capacity — for either storage or throughput — is genuinely exhausted.

---

## 7. Advantages of Partitioning

| Advantage       | What it actually means |
|-----------------|--------------------------|
| **Parallelism** | Different partitions can be scanned/operated on concurrently instead of one process handling the entire table serially. *(When partitions are sharded across servers, this parallelism is achieved via a routing layer — see the clarification below.)* |
| **Availability** | Since data is spread across independent partitions, if one partition/server becomes unavailable, only that slice of data is affected — the rest of the database keeps serving requests. *(See [Section 9.5](#95-high-availability-sharding-alone-isnt-enough) for an important caveat on real production availability.)* |
| **Performance**  | Each partition only has to be searched through and operated on as **its own smaller slice** of the data instead of the entire dataset, so individual queries execute faster. |
| **Manageability**| Smaller partitions are easier to **back up, maintain, re-index, and administer** individually than one giant monolithic table. |
| **Reduced Cost** | Scaling out with sharded partitions (more, cheaper commodity servers) is generally more cost-effective than scaling up (buying one increasingly expensive, more powerful machine). |

> **📌 Clarification — a routing layer is a sharding concept, not a generic partitioning one.** The original draft attributed "requests are filtered through a routing layer" to partitioning in general. That's only accurate for **sharded** (cross-server) partitioning.
>
> For **local partitioning** on a single server — e.g., PostgreSQL native table partitioning — the database engine itself figures out which partition to scan. This is called **partition pruning**, and it requires no external routing component:
> ```sql
> SELECT * FROM orders
> WHERE order_date = '2025-05-01';
> -- The query planner automatically prunes to the Orders_2025 partition.
> ```
> Routing layers only enter the picture once partitions are spread across **separate DB instances** — i.e., once you're sharding. Real-world examples include Vitess, MongoDB's `mongos`, ProxySQL, and custom application-level shard routers.

---

## 8. Distributed Database

- A **distributed database** is a database whose data is stored across **multiple networked nodes**, while still **appearing as a single logical database** to the user.
- **Why is it needed?** — Refer back to [Section 6](#6-when-is-partitioning-applied): huge datasets and huge request volumes are what force us toward this kind of architecture.

> **📌 Clarification — a distributed database doesn't require clustering + partitioning + sharding all together.** The original draft called a distributed database *"the end product of applying DB optimisation techniques like Clustering, Partitioning, and Sharding together,"* which implies all three are necessary. They aren't — any one of them, alone, can already produce a distributed database:
>
> - **Replication only** — the same data copied across multiple nodes.
> - **Sharding only** — different data slices spread across multiple nodes.
> - **Partitioning only** — as long as the partitions are placed across servers rather than staying local.
> - Or **any combination** of the above.
>
> **Corrected definition:** *A distributed database is a database whose data is stored across multiple networked nodes while appearing as a single logical database.* Clustering, partitioning, and sharding are simply the common **techniques used to build** one — not a mandatory checklist.

---

## 9. Sharding

### 9.1 What is Sharding

- As established in [Section 5](#5-the-partitioning-family-tree), sharding is the **distributed** case of **horizontal partitioning** — it is a specific implementation strategy, not a different concept from horizontal partitioning.
- The core idea: instead of having **all the data sit on one DB instance**, we **split it up** across multiple instances, and introduce a **routing layer** on top so incoming requests can be **forwarded to the right instance** that actually holds the relevant data.

### 9.2 The Routing Layer

- Sharding specifically (unlike local partitioning — see [Section 7](#7-advantages-of-partitioning)) needs a **routing layer** — its job is to look at an incoming request and determine **which shard/instance** that request needs to be sent to.
- Continuing the horizontal-partitioning example from [Section 4.2](#42-horizontal-partitioning): if Partition A and Partition B were deployed as separate shards, a request for an Engineering employee's record would be routed straight to the **Engineering shard**, without touching the Sales shard at all.

### 9.3 Pros of Sharding

- **Scalability** — new shards (instances) can be added as data/traffic grows.
- **Fault isolation** — a failure in one shard doesn't automatically take down the entire database; the same reasoning as the Availability row in [Section 7](#7-advantages-of-partitioning), with the caveat in 9.5 below.

### 9.4 Cons of Sharding

- **Complexity** — someone has to build and maintain the **partition mapping** and the **routing layer** logic itself, which adds real engineering overhead to the system.
- **Non-uniformity** — data doesn't always distribute evenly across shards (e.g., one department/region grows much faster than others), which eventually creates the need for **re-sharding** (rebalancing data across shards).
- **Not well suited for analytical queries** — since the data is spread across different DB instances, a query that needs to aggregate/join across the *entire* dataset (e.g., `SELECT COUNT(*) FROM Employees;`) has to hit every shard and combine the partial results. This is known as the **Scatter-Gather problem**, and it's one of the major operational challenges of sharded systems.

### 9.5 High Availability: Sharding Alone Isn't Enough

> **📌 Clarification — the "if one server goes down, only that data is unavailable" claim needs a caveat.** This is true *only in the absence of replication*. It describes **fault isolation**, not full **high availability**.
>
> In real production systems, each shard is usually paired with its own replica(s), not left as a single point of failure:
> ```
> Shard 1                    Shard 2
> ├── Primary                ├── Primary
> └── Replica                └── Replica
> ```
> If a shard's primary node fails, a replica for *that shard* can take over, so the data it holds stays available. Without this, losing a shard's only node means that slice of data is genuinely gone from the system until it's restored.
>
> **Takeaway:** Sharding alone gives you *fault isolation* — a failure is contained to one shard instead of crashing the whole database. True **high availability** for that shard's data comes from combining sharding **with replication**.

---

## 10. Quick Reference: Common Misconceptions

| ❌ Imprecise statement | ✅ More accurate statement |
|---|---|
| "Partitioning always means splitting across separate servers." | Partitioning can happen on a single server (e.g., `Orders_2024`, `Orders_2025` on one instance) or across many. |
| "Sharding = Horizontal Partitioning." | Sharding is horizontal partitioning that's specifically distributed across multiple DB instances. Every shard is a horizontal partition; not every horizontal partition is a shard. |
| "Partitioning needs a routing layer." | Only *sharded* (cross-server) partitioning needs a routing layer. Single-server partitioning relies on the DB engine's own partition pruning. |
| "A distributed database = Clustering + Partitioning + Sharding, all together." | A distributed database just needs data spread across networked nodes that behave as one logical database — replication alone, sharding alone, or any combination can produce this. |
| "If one shard goes down, only its data is unavailable — full stop." | True only without replication. Production HA setups pair each shard with replicas so that a node failure doesn't mean data loss for that shard. |
| "Clustering always means one primary + multiple replicas." | That's the most common pattern, but multi-master and shared-nothing clusters also exist. |
