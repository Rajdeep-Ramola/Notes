# Clustering in DBMS

---

## Table of Contents

1. [What is Database Clustering?](#1-what-is-database-clustering)
2. [Why is Clustering Needed?](#2-why-is-clustering-needed)
3. [Advantages of Clustering](#3-advantages-of-clustering)
   - 3.1 [Data Redundancy](#31-data-redundancy)
   - 3.2 [Load Balancing](#32-load-balancing)
   - 3.3 [High Availability](#33-high-availability)
4. [How Does Clustering Work?](#4-how-does-clustering-work)

---

## 1. What is Database Clustering?

**Database Clustering** (also called making **Replica-sets**) is the process of combining more than one server or instance to connect to a single database. In simple terms, it means **replicating the same dataset across different servers**.

- Database clustering, SQL server clustering, and SQL clustering are all closely associated terms, since **SQL** is the language commonly used to manage database information.

```
                ┌───────────────────────────┐
                │         Cluster           │
                │                            │
   Requests →   │  ┌─────────┐  ┌─────────┐  │
                │  │ Server 1│  │ Server 2│  │
                │  │ (Data)  │  │ (Data)  │  │
                │  └─────────┘  └─────────┘  │
                │        ┌─────────┐         │
                │        │ Server 3│         │
                │        │ (Data)  │         │
                │        └─────────┘         │
                └───────────────────────────┘
        (Same dataset replicated across multiple servers)
```

---

## 2. Why is Clustering Needed?

Sometimes a **single server is not adequate** to manage:

- The **amount of data** being stored, or
- The **number of requests** coming in.

In such cases, a **Data Cluster** is needed — the same dataset is replicated across different servers so that the load and data can be handled collectively rather than relying on just one machine.

---

## 3. Advantages of Clustering

### 3.1 Data Redundancy

Clustering helps achieve **data redundancy** by storing the same data on multiple servers.

> **Note:** This redundancy should not be confused with harmful repetition of data that leads to anomalies. The redundancy offered by clustering is **required and intentional**, made possible through **synchronization** across servers.

- If any one server fails, the data remains **available on the other servers**, ensuring nothing is lost.

```
 Server 1 (fails ✗)     Server 2 (active ✓)     Server 3 (active ✓)
      │                       │                        │
      X                    [Data]                   [Data]
                     (Data still accessible from Server 2 & 3)
```

### 3.2 Load Balancing

**Load balancing** (and scalability) doesn't come by default with a database — it has to be achieved through clustering, and depends on the setup used.

- Load balancing works by **allocating workload among the different servers** that are part of the cluster.
- This means **more users can be supported**, and if a sudden spike in traffic occurs, the cluster is better equipped to handle it.
- No single machine absorbs all the requests — this allows for **seamless scaling** as needed.
- Load balancing is directly linked to **high availability**: without it, one machine could get overworked, causing traffic to slow down and, in the worst case, drop to zero.

```
        Incoming Requests
               │
        ┌──────┼──────┐
        ▼      ▼       ▼
   Server 1  Server 2  Server 3
   (33%)     (33%)     (34%)
   (Workload is spread evenly instead of hitting one server)
```

### 3.3 High Availability

**Availability** refers to whether a database can be accessed. **High availability** refers to the amount of time a database remains available for use.

- The level of availability needed depends on factors like the **number of transactions** being run and **how often analytics** are performed on the data.
- Through clustering, extremely **high levels of availability** can be achieved — this is a result of load balancing and having extra (redundant) machines in the cluster.
- If one server shuts down, the database as a whole **remains available**, since other servers can continue to serve requests.

---

## 4. How Does Clustering Work?

In a cluster architecture, incoming requests are **split across multiple computers**, so that an individual user's request may be executed and served by more than one computer system working together.

- Clustering is made possible by the combined abilities of **load balancing** and **high availability**.
- If one node in the cluster **collapses (fails)**, the request is automatically handled by **another node**.
- As a result, there are **very few — or no — chances of a complete system failure**.

```
   User Request
        │
        ▼
   ┌─────────┐   node fails   ┌─────────┐
   │ Node 1  │ ─────✗───────▶ │ Node 2  │ ──▶ Request completed
   └─────────┘                └─────────┘
   (If Node 1 fails, Node 2 automatically picks up the request)
```

