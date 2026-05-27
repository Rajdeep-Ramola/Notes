# Scaling and Performance in Backend Engineering

## Overview

Scaling and performance are two of the most widely used buzzwords in the world of systems, backends, and infrastructure — and for good reason. They are equally important and expand into many different territories and domains. The definition of scaling and performance looks very different depending on whether you are talking about browser systems, frontends, or infrastructure.

In the context of **backend engineering**, we will start from the very beginning — from the primary question of what performance even means — and build up layer by layer. The goal is to develop an intuition for:

- How systems behave under load
- Where bottlenecks actually hide
- How to think clearly when everything seems to be falling apart

By the end, you will not just know a few scaling techniques. You will understand **how to think about scaling and performance** in a way that applies universally, regardless of the system you are working with.

---

## What is Performance?

### Defining "Fast"

Let's ask the most fundamental question first: **what do we actually mean when we say a system is fast?**

Consider a modern web application. A user visits the site, clicks a button, and the following chain of events occurs:

1. The browser (e.g. Chrome) sends a request over the internet to the server
2. The server receives the request and processes it
3. The server likely interacts with the database — to store or fetch some data
4. The server may also call an external API (e.g. calling Resend to send an email)
5. The server sends back a JSON response
6. The response travels back over the internet to the browser
7. The browser parses the JSON and renders the result on screen (e.g. a list of cards)

There is some amount of time that passes between the user clicking the button and the cards finally appearing on the screen. This total time — from the user's action to the system's visible response — is what we call **latency**.

---

## Latency

### What is Latency?

**Latency** is one of the most fundamental concepts in performance. It is the measurable, numerical way to express how "fast" or "slow" a system feels to its users. When someone says "your app is slow," they are — knowingly or not — talking about latency.

Latency gives us something concrete: instead of saying "the app feels sluggish," we can say "the P99 latency is 2 seconds." This transforms a vague feeling into a measurable engineering problem.

### Why Latency is Not a Single Number

Latency is not as simple as saying "500 milliseconds." It **varies from request to request**. In a real-world system:

- One request might complete in **50 milliseconds** — perhaps because it hit a CDN cache or an in-memory cache like Redis, retrieving the result from a much faster storage layer
- Another request might take **200 milliseconds** — because it had to travel all the way to the database server, or because the server was concurrently processing 50 other requests at the same time

Real-world traffic is unpredictable. Nothing in production is as neat and orderly as a whiteboard diagram.

---

## Why Averages Are Misleading

If we try to average the latencies of all requests — say 50 ms and 200 ms — we get **125 milliseconds**. But does this number tell you anything meaningful about how your system actually performs?

**Averages are useful in many contexts, but in performance engineering, they can be dangerously misleading.** Here is why:

Imagine you measured 1,000 requests and calculated an average latency of **100 milliseconds**. That sounds great. But hiding inside that number could be the following reality:

- **99% of requests** completed in under 50 milliseconds
- **1% of requests** took around **5 seconds**

That 1% represents real users in your system who are having a terrible experience. Now scale it up:

- If your system serves **1 million requests per day**, that 1% equals **10,000 requests**
- 10,000 users had to stare at a loading spinner for **5 seconds**

The average latency would tell you your system is performing well. It would never reveal the frustration of those 10,000 users. This is exactly why averages fail us when talking about performance.

---

## Percentiles: P50, P90, and P99

Instead of averages, backend engineers talk about **percentiles**. Understanding the terminology used in performance discussions is critical — it lets you understand different opinions across the ecosystem. Here are the three most important:

### P50 — The 50th Percentile

If your **P50 latency is 400 milliseconds**, it means **50% of your users experience 400 ms of latency or less**.

### P90 — The 90th Percentile

If your **P90 latency is 900 milliseconds**, it means **10% of your users experience 900 ms of latency or more**.

A useful mental shortcut: subtract the percentile number from 100 to find the percentage of affected users. For P90: 100 - 90 = 10%, so 10% of users experience latency at or above that number.

### P99 — The 99th Percentile

If your **P99 latency is 2 seconds**, it means **1% of your users experience 2 seconds of latency** (or equivalently, 99% of users get a response faster than 2 seconds).

Using the same shortcut: 100 - 99 = 1%, so 1% of users experience that latency level.

These three numbers — **P50, P90, and P99** — will come up constantly in any discussion of performance, optimization, or scaling. Get comfortable with them.

### Why Engineers Focus on P99 and P95

In practice, backend engineers focus most heavily on **P99 and P95 latencies**. The reason is not just about caring for unhappy users — though that matters too. The requests that end up with the highest latency are usually the ones with:

- The most complex business logic
- The most expensive database queries
- The most complex service coordination (calling external APIs, waiting for webhooks, sending emails, etc.)

More importantly, the users experiencing P99 and P95 latencies often represent **your most valuable customers** — the ones whose workflows trigger the most complex operations. For example:

- A user **browsing** your platform generates simple read queries
- A user **making a payment** generates complex database writes, business logic validations, and external service calls

That is why caring about P99 and P95 is not just empathetic — it is good business.

---

## Throughput

### What is Throughput?

**Throughput** is the other critical performance metric. While latency tells you how long an individual request takes from start to finish, **throughput tells you how many requests your system can handle in a given time period**.

Throughput is typically measured in **requests per second (RPS)** or **requests per minute (RPM)**.

### Why Throughput Matters

You can have an impressively low latency under light load, but that number means nothing if it collapses under real traffic. For example:

- At **10 requests per second**, latency might be a comfortable **150 milliseconds**
- At **1,000 requests per second**, that same system's latency might jump to **2 seconds**

Understanding throughput alongside latency helps you answer real-world questions:

- Can our system handle the traffic spike of **Black Friday**?
- If we run an email campaign and suddenly get a surge of new users, how many concurrent users can we support?
- How many requests per second can we sustain before we need to provision more resources?

### The Relationship Between Throughput and Latency

Latency and throughput are deeply connected — but not in an intuitive way. **As throughput increases, latency also increases.** At first this happens slowly, but after a certain point it grows dramatically.

This non-linear relationship is a core concept in performance engineering and something we will explore in detail in the next section.

---

## Utilization and the Latency Curve

### The Ice Cream Shop Analogy

Imagine an ice cream shop with one worker. On a quiet Sunday evening, you walk in, there is nobody else there, you order, and you get your ice cream immediately. This is **low utilization with low latency** — the worker has spare capacity, and your request is served instantly.

Now imagine a busy Tuesday lunch rush. The same worker is there, still taking exactly **2 minutes** to prepare each ice cream — their speed has not changed at all. But you are standing somewhere in a long queue. You wait for everyone ahead of you to be served first. Your **wait time has increased significantly**, even though the worker's execution rate has not changed at all.

This is exactly how backend servers behave:

- When CPU utilization is low, a request arrives, the CPU grabs it, processes it, and returns the response almost immediately
- As more requests arrive, each one waits for the requests ahead of it to complete — a **queue forms**
- The higher the utilization, the longer the queue, and therefore the longer each request waits

### What is Utilization?

**Utilization** is the percentage of your system's capacity that is currently in active use.

- **0% utilization** → the system is idle, doing nothing
- **100% utilization** → the system is completely maxed out, at the brink of collapse

### The Counterintuitive Latency Curve

Here is the counterintuitive part. You might expect that as utilization increases, latency increases **linearly** — double the load, double the wait. But that is not what happens.

In reality, as utilization approaches 100%, latency does not grow linearly — **it grows exponentially**. The curve starts gentle at low utilization, but as the system nears capacity, it bends sharply upward.

The highway analogy makes this intuitive:

- **50% capacity** — Cars have plenty of space to overtake and change lanes. Traffic flows smoothly. Travel times are predictable.
- **80% capacity** — Some slowdowns appear. Changing lanes requires more thought. Things are still mostly functional but less predictable.
- **90% capacity** — Traffic becomes highly unpredictable. Sometimes it flows if cars are perfectly aligned; sometimes a single impatient driver triggers a cascade of braking and honking that ripples backwards through the entire highway.
- **100% capacity** — No one can move. The highway is completely blocked.

### The Key Realization

**You cannot run your systems at 100% utilization and expect them to perform well.** You need headroom — spare capacity — to absorb traffic spikes.

In real-world production systems, most backends run at **60–80% utilization**, reserving the remaining 20% as a buffer for two reasons:

1. **Traffic spikes** — unexpected surges in requests can push utilization well above the average
2. **Bursty traffic** — real-world traffic does not arrive like a metronome. It comes in **bursts**: a flood of requests one minute, near silence the next, then another surge half an hour later

Even if your **average** traffic is comfortably within capacity (say, 40%), a burst can instantly spike you past 100%. Without that buffer, your system crashes.

> **The rule:** Always keep headroom. Always maintain a buffer. Never run at 100% utilization.

---

## Finding Bottlenecks: Measure, Don't Guess

### What is a Bottleneck?

When we say a system is slow, it means something **specific** in that system is causing the slowness. Finding that specific component is called **identifying the bottleneck**.

This sounds obvious, but in practice, backend engineers routinely skip this step. Instead of measuring to find the actual cause, they jump directly to well-known solutions:

- "Add caching — it's the answer to all latency problems"
- "Upgrade the database from Postgres 16 to Postgres 18"
- "Add more servers — horizontal scaling will fix it"

Sometimes these solutions help by coincidence. But more often, **you spend days or weeks implementing a solution to a problem you do not actually have**, while the actual bottleneck remains completely untouched — and your system stays slow.

### A Real-World Example

Say you have a `GET /products/:id` API that seems slow. Your first instinct: the database must be the problem. So you spend a week integrating Redis as a caching layer in front of your database, deploy it to production, and... the API is still slow.

Now you go deeper. You add granular timing logs throughout the code:

- When does the request arrive?
- When does the database query start and end?
- When does the cache lookup start and end?

What you discover:

- The **database query** takes only **10 milliseconds**
- The **Redis cache lookup** takes **5 milliseconds**
- But there is a **logging function** that writes to a remote logging service (like Elasticsearch) — and it runs **synchronously**, blocking the entire request until the log write completes. This takes **500 milliseconds**.

The database was never the problem. The synchronous logging call was the culprit — and because nobody measured before jumping to solutions, it went undetected for weeks.

**Unintuitive bottlenecks are everywhere:**

- A slow JSON serialization step
- An enormous response payload causing network transmission delay
- An external API call made inside a loop
- Synchronous operations that should be asynchronous

> **The rule: Never guess. Always measure.** Every time you encounter a performance problem, resist the urge to jump to a solution. Identify and measure each component of your request workflow first, then fix the thing that is actually causing the slowness.

---

## Profiling and Distributed Tracing

### Profiling

**Profiling** is the practice of measuring where your application actually spends its time. A profiler attaches to your running application and — during actual request processing — records samples of what is happening at every corner of your system:

- Which functions are executing?
- When did they start?
- When did they return?
- How long did each one take?

The output of a profiler can be overwhelming — thousands of functions, each accounting for some fraction of total time.

### Flame Graphs

A **flame graph** is the standard tool for making profiler output digestible. It shows the call stack over time:

- Functions that take more time appear **wider** in the graph — drawing your attention to them
- Functions called by other functions appear **stacked on top** of their callers

A quick visual scan of a flame graph can reveal where your application spends most of its time without reading raw output line by line.

The first time you look at a profiler output, you will almost certainly be surprised. You might expect your complex business logic to be the bottleneck — but the profiler shows that the real time sink is serializing the JSON response. You might blame database queries, only to find the actual culprit is an external API call made inside a loop.

### Limitations of Profilers

CPU profilers are excellent at measuring **CPU-bound tasks** — computations where the CPU is actively doing work (calculations, transformations, etc.).

However, most performance problems in typical backend applications are **I/O-bound** — they involve waiting for input/output operations:

- Database queries
- File reads/writes
- External API calls
- Message queue operations

CPU profilers are not well-suited for measuring I/O-bound latency. For those, we need a different tool.

### Distributed Tracing

**Distributed tracing** falls under the broader category of **observability**. It works by following a specific request as it flows through your entire system, recording timestamps at each stage:

- When did it enter the API handler?
- When did the database query start?
- When did the database query return?
- When did the external API call begin?
- When did the response go out?

After setting up distributed tracing, you can see something like:

> `PATCH /products/5` — 2 ms in business logic, **800 ms in the database query**

Now you know exactly where to focus your optimization effort. Distributed tracing is one of the most practical tools available for quickly identifying which component in your system is responsible for latency — especially in I/O-bound workflows where profilers fall short.

---

## Database Performance

Databases are typically among the first places backend engineers look when optimizing performance — and for good reason. Databases do a lot of hard work:

- Store data durably on disk so it survives reboots and crashes
- Handle concurrent reads and writes with locking and consistency guarantees
- Execute complex queries across millions or billions of rows

All of this takes time. But rather than blaming the database in general, it is more productive to look for specific, common patterns that cause avoidable performance problems.

---

## The N+1 Query Problem

### What is the N+1 Problem?

The N+1 query problem is one of the most notorious and common database performance issues. Here is a classic example to understand it:

Imagine a **React app** displaying a list of 20 blog posts, where each post shows its author's name. The author data is not included in the list API response, so the frontend:

1. Makes **1 API call** to fetch the list of 20 blog posts
2. For each post, makes **another API call** to fetch the author's details

That is **1 + 20 = 21 API calls** to render a single page. For 100 posts, it becomes 101 calls. For 1,000 posts, it is 1,001 calls. The number of network calls grows **linearly** with the number of items displayed — which is fundamentally wrong.

### Why This is a Problem

The most obvious issue is that the number of items to display and the number of network calls should not be linearly related. But even if you set that aside, there is a second, deeper problem:

**Every database query carries overhead.** Each query involves:

- A network round-trip from your application server to your database server
- Potentially a TCP connection setup (unless you are using connection pooling)
- The database parsing the query
- The database planning the query execution
- The database executing the query
- The database returning the results

Even if a single query is extremely fast at **5 milliseconds**, making **1,000 queries** for 1,000 items adds up to **5,000 milliseconds — 5 seconds**. Your users are staring at a loading spinner for 5 full seconds, which is unacceptable by any modern standard.

### The N+1 Problem at the Server Level

While the blog post example was framed from the frontend's perspective, the N+1 query problem **actually lives at the server level**. Your backend server is the "frontend" for your database — and the problem classically appears in ORM (Object-Relational Mapper) usage:

```python
# N+1 problem in ORM code
posts = db.select(posts).where(user_id=...)  # 1 query
for post in posts:
    author = db.select(author).where(user_id=post.author_id)  # N queries
```

This looks like normal, clean code in your programming language. The ORM abstracts away SQL, so you don't immediately think about how many queries are being executed behind the scenes. That abstraction is precisely why the N+1 problem is so common with ORMs — it does not look wrong until you inspect the generated SQL.

### The Solution: Bulk Fetching and Joins

The fix is to fetch all related data **in bulk**, rather than one item at a time:

1. Run **one query** to fetch all posts
2. Collect all author IDs from those posts into a single list
3. Run **one query** to fetch all authors for those IDs at once

Total: **2 queries** — regardless of whether you are displaying 20, 100, or 1,000 posts.

In SQL, this typically means using **JOINs** (INNER JOIN, LEFT JOIN, etc.).

In popular ORMs, there are built-in primitives for this:

| ORM | Bulk fetch mechanism |
|---|---|
| **Django** | `select_related()` (foreign keys), `prefetch_related()` (many-to-many) |
| **Ruby on Rails** | `includes()` |
| **TypeORM** (TypeScript) | `leftJoinAndSelect()` |
| **Prisma, Drizzle** | `include` / relation fetching |

The general principle across all ORMs: **instruct the ORM to prefetch all related data before entering any loop**, rather than fetching inside the loop one item at a time.

### Debugging N+1 with ORM Query Logging

Most modern ORMs offer an option to **print the raw SQL queries** being executed behind the scenes. Enabling this during development is highly recommended — it lets you see exactly how many queries are being fired and whether they match the pattern you intend.

> **Key takeaway:** Always be alert to N+1 query patterns when using ORMs and databases. Prefer joins and bulk fetch primitives over any pattern that fetches related data inside a loop.

---

# Scaling and Performance in Backend Engineering — Part 2

## Database Indexes

### What is an Index?

After the N+1 query problem, the second — and arguably most common — source of database performance problems is indexes, or more specifically, **the absence of indexes**.

The classic way to understand indexes is through the library analogy. Imagine a library with a million books spread across different shelves, sections, rooms, and floors — but with no catalog whatsoever. A reader walks in and asks for all books by the author John Green. As the librarian, you have only one option: walk through the entire library, examine every single book, and collect all the ones by John Green. This takes around three days. And if a second reader comes the next day asking for the same thing, your three-day adventure starts all over again.

This process — scanning every row of a table to find the ones you need — is called a **sequential scan**, also known as a **full table scan**. In a database, it does not take three days, but for a very large table it can easily take 1,000 milliseconds to 2 seconds — which, by modern performance standards, is unacceptably slow.

### How Indexes Solve This

Going back to the library: if you maintained a **catalog** organized alphabetically by author name, you could look up "John Green," find the exact shelf locations of all his books, walk directly to those shelves, and return the books in 2–3 minutes instead of 3 days.

That catalog is exactly what a **database index** is.

An index is a data structure — typically a **B-tree** (though hash indexes and others exist) — that maintains a **sorted copy of the values in a specific column**, along with pointers to the corresponding rows in the table. When you search by that column, the database uses the index to jump directly to the relevant rows instead of scanning every row one by one.

**The performance difference is dramatic:**

| Approach | Time for 1 million rows |
|---|---|
| Full table scan (no index) | ~4 seconds |
| Index scan | ~40–100 milliseconds |

### Indexes Are Not Free

Indexes are not a one-stop solution to all database performance problems. They come with real costs:

**1. Storage cost** — Each index stores a sorted copy of the indexed column's values along with row pointers. As your table grows, the index grows with it. More indexes mean more disk usage.

**2. Write overhead (the more critical cost)** — Every index on a table must be kept in sync with the underlying data. This means that every `INSERT`, `UPDATE`, or `DELETE` operation on that table must also update all of its indexes. If you create an index on every column of a table, every write operation will be significantly slower because it has to update each index data structure one by one.

> **The rule:** Be conservative and deliberate when creating indexes. Do not index every column. The write overhead of maintaining too many indexes can easily outweigh the read performance gains.

### Which Columns Should You Index?

Some indexes are obvious from the start:

- **Primary keys** are automatically indexed by the database (Postgres does this by default — you do not need to explicitly index the `id` field)
- **Foreign key columns** you frequently search or join on (e.g. `author_id` on a books table) are often good candidates to index upfront, since filtering by foreign key is a very common pattern

For less obvious cases, **do not guess — measure first**. Use your distributed tracing setup to identify which endpoints have high latency, find the specific database queries responsible, and only then decide whether an index is warranted. Indexes have overhead, so you want to be confident a column is frequently queried before indexing it.

### Composite Indexes

A **composite index** covers multiple columns together. For example, if you frequently query by both `user_id` and `created_at`, a composite index defined as `(user_id, created_at)` will optimize that query better than two separate single-column indexes.

**Important: column order matters.** A composite index on `(user_id, created_at)` will be used for queries that filter by `user_id` alone, or by both `user_id` and `created_at` together. But it will **not** be used for queries that filter by `created_at` alone. The leading column in the index definition determines which query patterns it can serve.

### Covering Indexes

A **covering index** is an index that includes all the columns a particular query needs, so the database can serve the entire result directly from the index without ever touching the underlying table.

For example, if a `departments` table has 100 columns but a frequently run query only needs `id` and `name`, you can create a covering index on `name` (and optionally include `id`). The database will serve that query entirely from the index, making it extremely fast. The trade-off is that the more columns you add to a covering index, the larger it grows — so this requires careful analysis.

### Using `EXPLAIN ANALYZE` to Diagnose Index Usage

Once you have identified a slow query through distributed tracing, how do you know exactly which column needs an index? Most databases provide the answer through a built-in tool.

In Postgres, prefix your query with `EXPLAIN ANALYZE`:

```sql
EXPLAIN ANALYZE SELECT * FROM books JOIN authors ON books.author_id = authors.id WHERE ...;
```

The database will show you its **query execution plan** — which tables it scanned, which indexes it used (or didn't), and how long each step took. Look for steps labeled `Seq Scan` (sequential / full table scan) on large tables — those are your candidates for indexing.

After adding an index, run `EXPLAIN ANALYZE` again. If the database is now using your new index, the label will change from `Seq Scan` to `Index Scan`, and the query should be noticeably faster.

> **Workflow summary:** Distributed tracing → identify slow query → `EXPLAIN ANALYZE` → find missing index → add index → verify with `EXPLAIN ANALYZE` again.

---

## Connection Pooling

### Why Database Connections Are Expensive

One realization that often surprises engineers as they start scaling is that **database connections are not cheap**. In local development, connections are so lightweight we never think about them. But at scale, the cost of connections becomes a serious performance concern.

Every time your application establishes a new connection to the database, the following steps happen:

1. A **TCP connection** is established (involving a three-way handshake)
2. **Authentication** is performed
3. **Encryption is negotiated** (private/public key exchange)
4. A **session state** is set up for data access
5. The database **allocates memory** for the connection (several megabytes)

All of these steps take time and consume resources — both on your server and on the database. If your application opens a new connection for every database operation and immediately closes it afterward, you pay this cost repeatedly, on every single HTTP request.

The second problem is that databases have a **hard limit on concurrent connections**. A typical Postgres database supports somewhere around 400–500 connections depending on its memory and CPU configuration. During a traffic spike — a big sale, a viral moment, a product launch — your backend can easily exhaust this limit when handling thousands of concurrent requests, causing the database to reject new connections and crash.

### What is Connection Pooling?

**Connection pooling** solves both problems. Instead of opening and closing a fresh connection for every database query, your application interacts with a **pool** — a middleman that maintains a set of long-lived, pre-established connections to the database.

Here is how it works:

1. When your server needs to make a database query, it **borrows a connection** from the pool
2. It uses that connection to execute the query
3. Once done, it **returns the connection to the pool** — it is not closed, just returned to the available set
4. The pool keeps the connection open for reuse by the next request

This eliminates the repeated cost of setting up connections, and it caps the total number of connections to the database at the pool's configured maximum — preventing connection exhaustion during traffic spikes.

### Internal vs. External Pooling

There are two types of connection pooling, and understanding the difference matters when you start scaling horizontally.

**Internal pooling** means the database driver itself maintains a connection pool within each server process. This is built into most modern database drivers and works well when you have a single server instance. The pool lives inside your application.

**The problem with internal pooling at scale:** When you horizontally scale and run multiple server instances (say, three instances each with a pool of 150 connections), the total number of connections to the database is `3 × 150 = 450`. If your database only supports 300 connections, a traffic spike that triggers another scaling event will push you past that limit and crash the database.

**External pooling** solves this by placing the pool outside your application as a standalone service that all server instances share. The most popular external pooler for Postgres is **PgBouncer** — a lightweight, open-source connection pooler.

With PgBouncer in place:

- All server instances (regardless of how many are spun up by autoscaling) connect to PgBouncer, not directly to the database
- PgBouncer maintains its own fixed pool (say, 250–300 connections) and forwards queries to the database
- The database never sees more connections than PgBouncer's configured limit, no matter how many application instances are running

> **In production, prefer an external pooler like PgBouncer**, especially when you are aware of traffic spike patterns or running in an autoscaling environment.

---

## Caching

### The Core Idea

If you have optimized your queries, added appropriate indexes, and configured connection pooling — and the database is still your bottleneck — the next logical step is **caching**.

The idea is beautifully simple: **store the result of any expensive operation** so that subsequent requests for the same data can be served from the stored result instead of repeating the expensive operation.

In practice, this usually means:

1. A request arrives
2. Check the cache (e.g. Redis) — if the result is already there, return it immediately (~50 ms)
3. If not, run the expensive database query (~800 ms), store the result in the cache, then return it

With a single caching layer, a response that previously took 800 ms can now take 50 ms for all subsequent requests — a dramatic improvement with minimal code change.

### Cache Invalidation

While the caching idea is simple, there is a famously difficult problem that comes with it: **cache invalidation** — keeping your cached data in sync with the actual data in your database.

There is a well-known saying in software engineering:

> *"There are only two hard problems in computer science: naming things and cache invalidation."*

This is not an exaggeration. Cache invalidation becomes genuinely difficult in multi-layer systems where cached data might live in Redis, CDNs, reverse proxies, and browser caches simultaneously. When the underlying data changes, you need to invalidate the right cache entries at the right time, at every layer.

There are two primary techniques for handling this:

#### 1. Time-Based Expiration (TTL)

Set a **Time To Live (TTL)** on each cache entry — a duration after which the entry automatically expires. The next request after expiry will bypass the cache and fetch fresh data from the database, then re-populate the cache.

**The challenge:** Choosing the optimal TTL is difficult. It varies per endpoint, per access pattern, and per business requirement. A TTL that is too short reduces the effectiveness of caching. A TTL that is too long (e.g. 7 days) risks serving stale data to users for days — which defeats the purpose of having a TTL at all.

#### 2. Event-Based Invalidation

Invalidate or delete a cache entry **explicitly at the moment the underlying data changes**. For example, whenever a user updates their profile, the handler that processes the update also deletes the cached version of that profile. The next `GET` request finds no cache entry and fetches fresh data from the database.

**The challenge:** You must remember to invalidate the cache at every code path that modifies the relevant data. If even one update path is missed, users may see stale data with no automatic recovery mechanism.

Both techniques have trade-offs. Many production systems use a combination — TTL as a safety net, with explicit invalidation on known write paths.

### Local vs. Distributed Caching

**Where** your cache lives is another important design decision.

**Local caching** stores data in an in-memory data structure (like a dictionary or map) inside each server process. It is extremely fast — essentially zero network latency. But when you run multiple server instances, each instance maintains its own independent local cache, creating **cache inconsistency**: an update processed by Server 1 is invisible to Server 2's local cache.

**Distributed caching** uses an external caching service — Redis, Memcached, or the modern alternative **Valkey** — that all server instances share. This eliminates inconsistency since everyone reads from and writes to the same cache. The trade-off is a **network round-trip** to reach the cache, which adds latency — typically around 50 ms even on an internal network, compared to 2–3 ms for an in-memory lookup.

**Tiered caching** combines both approaches: a small local cache for the "hottest" data (most frequently accessed), backed by a distributed cache for everything else. A request first checks the local cache, then falls back to the distributed cache, and optionally promotes the result back into the local cache based on access frequency.

### Caching Patterns

The question of *when* to populate and invalidate the cache is answered by a **caching pattern**. There are three primary patterns:

#### Cache-Aside (Lazy Loading)

This is the most commonly used pattern — often implemented without engineers even knowing it has a name.

1. Request arrives → check the cache
2. **Cache hit:** return the cached result immediately
3. **Cache miss:** query the database, store the result in the cache, return the result
4. On write operations: explicitly delete or invalidate the affected cache entry

This is also called **lazy loading** because data is only loaded into the cache when it is first requested, not proactively. It is the most intuitive pattern and works well for the vast majority of use cases.

#### Write-Through

Every write operation updates **both the database and the cache simultaneously** before returning a success response.

- **Advantage:** There are never cache misses for data that has been written — the cache is always up to date
- **Disadvantage:** Write operations take slightly longer because they must complete both the database write and the cache write before responding

#### Write-Behind (Write-Back)

Write operations update **only the cache** and return a success response immediately. The database is then updated **asynchronously** in the background.

- **Advantage:** Write latency is minimized — since cache writes are extremely fast, the response is returned almost instantly
- **Disadvantage:** If the asynchronous database write fails, the cache and database are now in an **inconsistent state**, which is a serious data integrity risk

### Cache Hit Ratio

The **cache hit ratio** (or cache hit rate) is the metric that tells you how effective your caching implementation actually is. It measures the percentage of requests that are successfully served from the cache versus those that have to fall through to the database.

A **90% cache hit ratio** means 90% of requests are served from the cache; only 10% reach the database. That is excellent.

A **20% cache hit ratio** means your caching layer is largely ineffective — something in your caching algorithm or invalidation strategy needs to be fixed.

The three main factors that affect your cache hit ratio are:

**1. TTL (Time To Live)** — A longer TTL means cached entries live longer and have more chances to be hit. But a very long TTL increases the risk of serving stale data.

**2. Cache size** — The larger your cache's memory allocation, the more data it can hold and the more hits it can serve. Evictions due to insufficient memory directly reduce the hit ratio.

**3. Data access patterns** — If you do not understand which endpoints are accessed most frequently, and at what times, your caching strategy will not align with actual user behavior. A deep understanding of your users' access patterns is prerequisite to designing an effective caching strategy.

---

## Scaling

### The Core Problem

Your application runs on servers. As your traffic grows and your user base expands, those servers — and the resources they hold — become insufficient. You continually need more capacity. There are two fundamental approaches to getting it: **vertical scaling** and **horizontal scaling**.

---

## Vertical Scaling (Scaling Up)

**Vertical scaling** means replacing your existing server with a more powerful one — giving it more resources without changing the count of servers. Think of it as making your single server taller and stronger.

The hardware upgrades involved are straightforward:

- **CPU** — add more cores (2 → 4 → 8 → 16, etc.)
- **Memory (RAM)** — increase primary memory (2 GB → 4 GB → 8 GB → 32 GB, etc.)
- **Storage** — expand disk capacity and upgrade to faster storage (SSDs, NVMe SSDs)
- **Network card** — upgrade to modern cards offering 10 Gbps+ throughput

### Advantages of Vertical Scaling

**Simplicity** is the biggest advantage. Your architecture does not change at all. You do not need to think about statelessness, load balancing, distributed state, or inter-server communication. You upgrade the machine, and your application automatically benefits:

- A server with twice as many CPUs can handle roughly twice as many concurrent requests
- A server with twice as much RAM can hold twice as much data in local cache

**Economics** also often favor vertical scaling in the early stages. A single powerful server typically costs less than two servers of equivalent combined power, and you avoid the operational overhead of managing multiple machines — security patches, backups, maintenance, load balancer configuration, and so on.

### Disadvantages of Vertical Scaling

**1. Hard limits** — No matter how much you spend, there is always a ceiling on how powerful a single machine can be. Cloud providers offer very large instances, but at some point you will hit the biggest available machine — 32 cores, 96 GB RAM, whatever the maximum is — and if you still need more capacity, you have nowhere to go vertically.

**2. Single point of failure** — One powerful server is still one server. If it crashes — due to a dependency failure, an OS bug, a hardware fault, or any number of reasons servers crash for — your entire platform goes down for the duration of the outage. You can mitigate this with standby servers and automatic failover, but the fundamental risk remains.

**3. No geographic distribution** — A single server located in the US cannot reduce latency for users in India. If 40% of your user base is geographically distant from your server, that 40% will always experience higher latency — and there is nothing you can do about it as long as you are limited to one machine.

---

## Horizontal Scaling (Scaling Out)

**Horizontal scaling** takes a fundamentally different approach. Instead of making one server more powerful, you **add more instances** of the same server and have them work together to serve your growing traffic. Instead of one beast of a machine, you have multiple medium-sized machines operating in parallel.

### Advantages of Horizontal Scaling

**No hard limit** — You can keep adding instances indefinitely. There is no ceiling on the number of servers you can configure to work together. If one server handles 1,000 requests per second, five servers handle 5,000. Need more? Add more.

**Redundancy** — If one instance goes down, traffic is redistributed among the remaining instances. Your service stays up. The failure of any single server has no impact on users, because other servers are there to absorb the load.

**Geographic distribution** — You can deploy instances in different regions around the world. Combined with a load balancer that routes each user to the nearest server, you can minimize latency for your entire global user base — something impossible with a single vertical server.

### Disadvantages of Horizontal Scaling

The disadvantages of horizontal scaling are rooted in **complexity** — the exact complexity that vertical scaling avoids entirely.

The moment you add a second server instance, a cascade of new questions appears:

- **How do you distribute requests?** You need a **load balancer** — an entirely new component in your infrastructure — along with a decision about which load balancing algorithm to use
- **How do you keep servers in sync?** If a user updates their name on Server 1, how does Server 2 know about it?
- **What happens when the network between servers fails?** Servers that cannot communicate may start making conflicting or redundant decisions
- **How do you detect that a server has gone down?** And at what point do you stop routing traffic to it — and resume routing once it recovers?

All of these questions have answers. The engineering community has solved them. But the solutions involve **trade-offs**, because this is the domain of **distributed systems** — and distributed systems do not eliminate problems. They transform one set of problems into a different set of problems, and the goal is to choose the set of problems that is more manageable given your specific context.

> **The fundamental trade-off:** Vertical scaling offers simplicity at the cost of hard limits and a single point of failure. Horizontal scaling removes those limits and adds redundancy, but introduces the full complexity of distributed systems. Choosing between them — or combining them — depends on your scale, your team's capabilities, and the specific problems you are willing to take on.

---