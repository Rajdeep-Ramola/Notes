# Scaling Backend Systems

## Introduction & Recap

This chapter is a continuation of the discussion on different types of scaling. The previous part covered the basics of horizontal scaling. If you are reading this for the first time, it is recommended to go through the first part for better context. This chapter picks up from where that discussion ended.

---

## Statelessness: The Key to Horizontal Scaling

A very important property when talking about horizontal scaling is **statelessness**. It is the key enabler — the core property that makes horizontal scaling possible in the first place.

### What Is Statelessness?

When we talk about horizontal scaling, we mean adding multiple instances of the same backend application instead of increasing the capacity of a single machine. So in a horizontally scaled setup, you have multiple servers all running the same code.

When your application is deployed and internet traffic arrives, all requests from users — whether through a frontend or directly to the backend — are distributed among all these instances.

The reason horizontal scaling is possible at all is because of **statelessness**, which means:

> **No single server instance holds any data or information that is exclusive to it.**

When we say a server is **stateful**, we mean it holds some information — about a client, about a user — that is stored only on that particular server and is not accessible by other instances.

### Why Stateful Servers Break Horizontal Scaling

Imagine you have four instances — A, B, C, and D. If instance D holds some data that instances A, B, and C do not have access to, horizontal scaling will not work as expected. You will start seeing weird errors pop up across your entire stack.

This is what was meant when we said:

> Horizontal scaling requires planning your entire stack from the ground up.

Unlike vertical scaling — where you just go to your infrastructure dashboard and add more CPU cores and RAM to a single machine — horizontal scaling affects your code, your infrastructure, and your entire architecture.

### The Core Rule of Statelessness

In a horizontally scaled setup, **it should not matter which server handles a particular request — the result should always be the same.**

Even if you remove one of the servers entirely (say, instance B), the remaining instances A, C, and D should exhibit exactly the same behavior as if B were still there.

This also means: **no instance should hold any piece of information that is not accessible by all other instances.** Any data that needs to be persisted must live outside all the servers — in a centralized location that every instance can access.

---

## Practical Examples of Statelessness

### Sessions

In a typical authentication flow, a user enters their email and password. The server authenticates the user, creates a session, stores the session ID in the user's browser as a cookie, and also saves the session somewhere on the server so future requests can be verified.

This is **stateful authentication**. Here is what goes wrong in a horizontal scaling setup:

- The user authenticates and their request hits **instance A**.
- Instance A creates a session and stores it in its own **in-memory data structure** (like an array in RAM).
- The session ID cookie is saved in the user's browser.
- The next request from the same user hits **instance B** (because of load balancing).
- Instance B has no access to the session information stored in instance A's memory.
- Instance B throws a **401 Unauthorized** error, creating a confusing and frustrating experience for the user.

**The fix:** Instead of storing the session in instance A's memory, it should be stored in a centralized **in-memory database** like **Redis**. Since all instances can access the same Redis instance, the next request — regardless of which server it lands on — can successfully verify the session.

### File Storage

Same problem applies to file uploads:

- A user uploads a file and the request lands on **instance A**.
- Instance A saves the file in its own local storage (SSD or disk).
- The next request — perhaps to retrieve the file — lands on **instance C**.
- Instance C has no access to instance A's local file system and throws an error.

**The fix:** Instead of saving the file locally, instance A should upload it to a centralized **object storage** service like **AWS S3**, **Cloudflare R2**, or an on-premise equivalent. Since object storage is accessible by all instances, any server can retrieve the file on demand.

### The General Rule

This pattern keeps repeating. Once you commit to horizontal scaling, the thumb rule is:

- **No piece of information, no file, nothing should be saved inside a particular instance.**
- Everything must be centralized and accessible by all instances.

Practical mappings:
- In-memory data (sessions, cache): use **Redis**
- File uploads: use **S3 or any blob/object storage**
- Databases: use a **centralized database** like PostgreSQL, MySQL, or a managed RDS — not something like SQLite with the DB file saved locally on the server

> Always think about the statelessness property of your whole architecture. Every decision you make at the code level should ensure that no piece of state is tied to a single instance.

---

## Load Balancers

### Why Load Balancers Are Necessary

In a horizontal scaling setup with multiple server instances, a critical question arises:

> When a user makes a request, how do we decide which instance to send it to? And where exactly in the stack is that decision made?

This is exactly the role of a **load balancer**. Without a load balancer, a horizontal scaling setup is almost impossible to operate. The entire architecture starts with thinking about — and setting up — a load balancer.

### How a Load Balancer Works

The concept is straightforward:

- All requests from all clients (users on browsers, mobile apps, etc.) are sent to the **load balancer** instead of directly to the servers.
- The load balancer uses some internal logic — an algorithm — to decide which server instance should handle the request.
- It forwards the request to that instance.
- The server processes the request and sends the response back to the load balancer.
- The load balancer returns the response to the client over the same HTTP connection.

The load balancer is essentially a **middleman** — it receives requests from the internet and routes them to the right server instance. The important part is the routing logic itself, which is what we call the **load balancer algorithm**.

---

## Load Balancer Algorithms

### 1. Round Robin

The simplest algorithm is **round robin**. It sends requests to servers in a rotating order:

- Request 1 → Server A
- Request 2 → Server B
- Request 3 → Server C
- Request 4 → Server A (cycle repeats)

**When does round robin work well?**

Round robin works best when:
- All incoming requests are of a similar type (similar cost, similar operations — database reads, cache lookups, etc.)
- All server instances have the same capacity (e.g., 4 GB RAM, 2 CPU cores each)

In this kind of uniform setup, round robin is effective and can handle a large amount of traffic with minimal complexity.

**When does round robin fall short?**

Consider two types of requests:

- **Request Type 1 (lightweight):** A simple database read — something like `SELECT * FROM users WHERE id = 5`. Resolves in around 200–300 milliseconds on average.
- **Request Type 2 (expensive):** A complex operation that involves:
  - Making an HTTP call to an external service (like Elasticsearch or an email provider)
  - Performing a write operation on a large database table with many rows and multiple indices

  Every write operation requires updating all indices, making it slow. This entire operation takes around **2 seconds** to complete.

Since the round robin algorithm sends requests in a mindless rotating order, it has no awareness of request type or cost. If a burst of expensive requests all end up going to the same server, that server can become overwhelmed and eventually crash.

---

### 2. Weighted Round Robin

A variation of round robin that accounts for different server capacities.

For example:
- Server A: 8 GB RAM, 4-core CPU
- Server B: 4 GB RAM, 2-core CPU
- Server C: 4 GB RAM, 2-core CPU

You can configure the load balancer to send **twice as many requests to Server A** as to B or C. So for every 8 incoming requests, the distribution might look like:

- 2 requests → Server A
- 1 request → Server B
- 1 request → Server C
- 2 requests → Server A (again)
- ...and so on

This ensures that the more capable server handles proportionally more traffic. However, even weighted round robin **cannot make intelligent decisions about request type** — it still doesn't know which requests are expensive and which are cheap.

---

### 3. Least Connections

A smarter algorithm that routes requests based on current server load — specifically, the **number of active connections** each server is currently handling.

**How it works:**

In HTTP (specifically HTTP/1.1), a client sends a request and waits for a response. While it is waiting — while the server is doing its work (making API calls, running database queries, etc.) — that is called an **active connection**. The connection remains open until the server sends a response and closes it.

Expensive operations keep connections active for longer. So the number of active connections on a server is a reasonable proxy for how busy that server currently is.

**Example walkthrough with two request types:**

- At startup, all servers (A, B, C) have **0 active connections**.
- Request 1 (type: expensive, ~2 seconds) → Server A (chosen arbitrarily since all are empty)
- Request 2 (type: lightweight, ~200ms) → Server B (fewest active connections)
- Request 3 (type: lightweight) → Server C (also 0 connections)
- Request 4 arrives → By now, B and C may already be done (200ms requests resolve quickly). Server A is still busy with its 2-second request. The load balancer checks: B or C have 0 connections, A has 1. The request goes to B or C — **not A**.

This way, Server A is protected from receiving more work while it is already dealing with a heavy request. The algorithm distributes load **intelligently based on real-time server state**.

**Weighted Least Connections** is the same algorithm but with capacity weights applied, so a higher-capacity server still receives proportionally more requests even under this smarter routing logic.

---

### 4. Least Response Time

The load balancer tracks how fast each server is responding. Servers returning responses quickly receive more requests. Servers that are already struggling and returning slow responses receive fewer requests. This prevents further overloading of a server that is already under stress.

---

### 5. Resource-Based Algorithms

These algorithms check actual **CPU and RAM usage** across server instances. Servers with lower resource utilization receive more traffic, while overloaded servers are given a break. This is one of the most accurate approaches to intelligent traffic distribution.

---

## Health Checks

### The Problem: What Happens When a Server Goes Down?

Imagine you have a load balancer routing traffic to servers A, B, and C. Server A crashes due to excessive load or a misconfiguration. If the load balancer is using round robin, it will keep sending requests to A — every request that lands on A will fail with a **502 or 503** error. Users whose requests land on A get errors, while users landing on B or C are fine. But since round robin cycles through all servers, users who were previously served by B might get routed to A on their next request and also start seeing errors. This can quickly become a widespread and very frustrating user experience.

### The Solution: Health Checks

Load balancers solve this with a simple but effective technique called **health checks**.

**How health checks work:**

- While the load balancer is routing actual user traffic, it simultaneously sends its own **test requests** to all server instances — typically every second.
- These test requests hit a simple endpoint (usually a `GET` request) that requires no heavy processing.
- If the server is healthy, it responds with a **200 OK** immediately.
- These test requests are lightweight enough that they add virtually no load to the server instances.

**What happens when a server fails:**

- If Server A goes down and the next test request gets no response — or gets a non-200 response like a 502 — the load balancer **immediately blacklists** that instance.
- From that point forward, all user traffic is routed only to B and C.
- The load balancer continues sending test requests to A every second.
- The moment A comes back online and starts returning **200 OK** responses again, the load balancer **removes it from the blacklist** and begins routing user traffic to it once more.

This technique ensures that failing servers are automatically taken out of rotation without any manual intervention, and are automatically reintroduced once they recover.

> Health checks are how load balancers maintain the reliability and availability of a horizontally scaled system.

---

## Database Scaling

### The Challenge With Scaling Databases

Scaling your application code becomes relatively straightforward once you externalize state — as covered in the statelessness section. You can keep adding more server instances behind a load balancer and capacity scales linearly.

But there is one part of the architecture that is harder to scale: **the database**.

Databases are a **stateful component**. They hold large amounts of data in files on disk, and that data must remain consistent. Unlike application servers — which you can duplicate freely because they hold no unique state — database instances cannot simply be copied without careful coordination.

If you have multiple database instances, they must always return the same data for the same query. Making that coordination work is what makes database scaling a uniquely difficult problem.

---

## Read Replicas

### What Are Read Replicas?

Read replicas are a widely used, well-proven architecture for scaling database read operations across multiple instances.

The setup works like this:

- You have **one primary (master) database instance** that handles all write operations — `INSERT`, `UPDATE`, `DELETE`.
- You have **multiple replica (secondary) instances** that are copies of the primary and handle **only read operations** — `SELECT` queries.
- Replicas cannot accept writes — that is the number one rule.

### Geographic Distribution

Replicas are typically spread across different regions of the world. For example:
- Primary database in the **US**
- Read replicas in **India**, **China**, and **Japan**

Users in those regions communicate with backends deployed in the same region, which in turn read from the nearest replica. This reduces latency significantly compared to routing all reads back to a single instance in the US.

### Two Key Benefits

**1. Reduced load on the primary instance:**

For most SaaS applications, roughly **70–90% of all database requests are read requests** — simple `SELECT` queries fetching data. With read replicas:
- 70–90% of requests go to the replicas (reads)
- Only 30% or fewer go to the primary (writes)

The primary database goes from handling 100% of all queries to handling only 30% — a massive reduction in resource utilization.

**2. Lower latency:**

Since replicas can be placed in the same region as your users, read queries travel a much shorter distance. Response times improve dramatically compared to a single globally centralized database.

---

### The Consistency Problem

Read replicas come with a significant trade-off: **consistency**.

**Example scenario:**

1. A user updates their name in their profile from "A" to "AB".
2. This is a write operation — it goes to the **primary database** in the US.
3. The primary now holds the updated name "AB".
4. The user immediately refreshes the page — this triggers a **read request**, which goes to the **read replica in India**.
5. The replica has **not yet received the updated data** from the primary, because replication takes time.
6. The replica returns the old name "A".
7. Even though the save was successful, the user sees their old name — a confusing experience.

The time it takes for updated data to travel from the primary to the replicas is called the **replication lag**. Due to the physical distance between servers (e.g., US to India), replication lag can easily be **200–300 milliseconds** — the speed of light through undersea fiber optic cables is a physical limit that cannot be overcome.

### Solutions to the Consistency Problem

Several approaches exist to deal with replication lag:

**1. Intelligent request routing:**
After a write operation (e.g., updating the users table), route subsequent read requests for that same entity to the **primary instance** instead of a replica, until replication is confirmed complete.

**2. Blocking reads until replication is done:**
Track the average replication lag (e.g., 200–250 ms) and make read queries wait until replication is confirmed before responding to the client.

**3. Planned frontend latency:**
Architect the frontend so it does not immediately fire a `GET` request after a successful save. Instead, introduce a small intentional delay of around 300 milliseconds before re-fetching the data — enough time for replication to complete.

The right solution depends on the trade-offs you are willing to accept. This is a recurring theme in distributed systems: **there is always a trade-off**, and it is almost always about consistency.

> Modern managed database providers — like AWS RDS, Google Cloud SQL, Neon, PlanetScale — make setting up read replicas straightforward. It often takes just a few configuration toggles in a UI, and they handle robust consistency solutions internally. As a backend engineer, you may not need to implement this at the infrastructure level, but you must understand these concepts to configure your database correctly.

---

## Sharding (Partitioning)

### The Problem Sharding Solves

Even with read replicas, you may eventually hit a point where a single primary database struggles under the volume of data and write operations. Imagine an e-commerce application with billions of rows in the `orders` table. A query like:

```sql
SELECT * FROM orders WHERE user_id = 5;
```

...on a table with hundreds of billions of rows is extremely slow — even with proper indexing on `user_id` — simply because the volume of data is so massive.

Sharding solves two problems simultaneously:
1. **Query latency** — fewer rows per instance means faster queries
2. **Write throughput** — multiple database instances can handle more write requests per second

### What Is Sharding?

**Sharding** (also called **partitioning**) means **dividing a database table across multiple physical database instances**, rather than keeping all the data in one place.

**Simple example:**

Imagine your `orders` table has 10 billion rows (represented as 1–10 for simplicity):

- **Shard 1** (first database instance): rows 1–5 (orders from January to June)
- **Shard 2** (second database instance): rows 6–10 (orders from July to December)

The **order date** becomes the **sharding key** — the criterion used to decide which shard a particular piece of data belongs to.

### Benefits of Sharding

- **Two physical instances** means double the capacity for handling requests per second.
- Each instance now holds **5 billion rows instead of 10 billion** — query latency drops because there is simply less data to scan.

You can shard even more granularly if needed — one shard per month, for example — further reducing per-shard data volume and query time.

### How Routing Works

In your backend's database routing layer, before making a database query, you determine which shard holds the relevant data and route the request to that specific shard. The shard processes the query and returns the result.

### Choosing a Sharding Key

Deciding on the right sharding key is one of the trickiest parts of implementing sharding. The goal is to choose a key that:
- Distributes data evenly across shards (avoids one shard becoming much larger than others)
- Aligns with how you most frequently query the data

In the orders example, order date is an intuitive choice if you often query by time range. For other use cases, `user_id`, `region`, or other high-cardinality fields might be better choices.

---

## Distributed Databases: The Modern Approach

As of 2025, the current trend in database scaling is the use of **distributed databases** — purpose-built systems that handle sharding, replication, and distributed transaction coordination automatically. Some well-known examples include:

- **PlanetScale** — MySQL-based, built on Vitess
- **Neon** — PostgreSQL-based, written in Rust, serverless architecture
- **CockroachDB** — Distributed SQL database
- **YugabyteDB** — Another distributed SQL option

These databases handle all the complexities that arise with scaling — sharding, replication, distributed transactions, and more — so that you as a backend engineer do not have to manage them manually.

### What This Means in Practice

In a realistic scenario, you will:

1. Choose a managed database provider based on features, pricing, and team preference (AWS RDS, Neon, PlanetScale, etc.)
2. Sign up, create a database, and receive a connection URL with credentials
3. Plug those credentials into your backend and start interacting with the database

You will not manually implement replication or sharding. But you **do need to understand these concepts** so that you can:
- Go into your database provider's console and configure it correctly
- Decide how frequently to take database backups
- Choose which regions to place read replicas in
- Plan your sharding strategy if needed

> **The actual activity** of replication, sharding, and distributing replicas is handled by the provider. **Understanding what these terms mean and what trade-offs they involve** is what you need as a backend engineer.

---

## Summary

- **Statelessness** is the foundational property that makes horizontal scaling possible. Every piece of data — sessions, files, cache — must be stored in a centralized, shared location rather than inside individual server instances.
- **Load balancers** sit in front of all server instances and route incoming requests using algorithms like round robin, weighted round robin, least connections, least response time, and resource-based routing.
- **Health checks** allow load balancers to automatically detect failed server instances and stop routing traffic to them, restoring them once they recover.
- **Read replicas** scale database reads by distributing SELECT queries across multiple instances spread across regions, while keeping all writes on a single primary. The main trade-off is **replication lag**, which can cause temporary consistency issues.
- **Sharding** scales databases by physically dividing a table across multiple instances using a sharding key. It reduces both query latency (fewer rows per shard) and increases overall write throughput.
- **Modern distributed databases** like PlanetScale, Neon, CockroachDB, and YugabyteDB abstract away much of the complexity of replication and sharding — but understanding the underlying concepts remains essential for configuring and operating them correctly.

# CDNs, Edge Computing, and Asynchronous Processing

## Overview

This chapter covers the last major layer of caching — one that operates at a **global scale** — and then moves into two closely related performance topics: edge computing and asynchronous processing. Together, these three concepts form a powerful toolkit for reducing both real and perceived latency in your backend systems.

---

## Content Delivery Networks (CDNs)

### Why CDNs Exist — The Physics Problem

To understand why CDNs matter, you first have to accept one fundamental constraint of our world: **the speed of light**.

Light travels through fiber optic cables at approximately **200,000 kilometers per second**. This is not the speed of light in a vacuum — it is the speed at which data travels through the undersea cables that form the backbone of the internet. No amount of engineering optimization can exceed this cap. This is a physics constraint, not a software one.

Now consider a real-world example. A user is in **Tokyo** and your server is in **US East (North Virginia)** — the region called `us-east-1`, where the majority of commercial applications are hosted. For a request to make a **round trip** — meaning the browser sends the request and receives the response back — the data travels roughly **20,000 kilometers**. At 200,000 km/s, that round trip alone takes approximately **100 milliseconds**.

And that is just the round trip. There is still a lot happening after the request arrives at the server:

- **Routing layer** — fairly fast, mostly regex-based routing to hand the request off to the right handler
- **Deserialization** — the HTTP message is converted into the data structure of the server's programming language. JSON becomes a JavaScript object in Node.js, or a struct in Go. This takes time proportional to payload size.
- **Business logic and database queries** — for a mid-level to complex query, assuming the server and database are in the same data center, this can take **50 to 100 milliseconds**
- **External API calls** — if the handler makes any external API calls, add another **200 milliseconds**

Add all of this together and that initial 100 milliseconds of unavoidable physical latency quickly becomes **500 to 800 milliseconds** for a user in Tokyo.

This is exactly the problem CDNs are designed to solve.

---

### How CDNs Work

Instead of every request traveling 20,000 kilometers to your primary server, CDNs place **edge locations** — also called **Points of Presence (PoPs)** — geographically close to your users.

So instead of a request from Tokyo traveling to North Virginia, it travels to a CDN node that is physically located in or near Tokyo. The round trip drops from 20,000 km to roughly **100 to 200 km**, and the latency drops from 100 milliseconds to **2 to 3 milliseconds**.

From a human perception standpoint, 100 milliseconds and 3 milliseconds might not feel dramatically different. But when you are thinking about **scaling and performance**, the difference is enormous. Cutting latency by 97% is a massive win.

---

### Benefits of Using CDNs

#### 1. Reduced Latency

As discussed above, CDN nodes placed close to users drastically reduce the round-trip time for requests. This is the primary and most obvious benefit.

#### 2. Reduced Load on Primary Servers

Because CDN nodes are distributed globally — one in Tokyo, one in Mumbai, one in Singapore, and so on — requests are served from the node closest to the user. Your primary server in the US does not receive nearly as much traffic.

As a result, your origin server may handle **50% less traffic** than it would without a CDN. This directly reduces your infrastructure costs and means you do not need to scale horizontally as aggressively, because a large portion of your traffic is being absorbed by CDN nodes.

#### 3. DDoS Protection and Security

One of the most relevant use cases for modern CDNs like **Cloudflare** is security, specifically protection against **DDoS (Distributed Denial of Service) attacks**.

In a DDoS attack, an attacker gains control of a large number of machines across the internet — perhaps 20,000 bots infected with some kind of malware. The attacker then directs all of these bots to flood your server with traffic simultaneously.

Your server has a finite amount of resources. Two things can happen:

- If you have **hard limits**, your server crashes
- If you have **horizontal scaling configured**, your infrastructure keeps spinning up new instances to handle the traffic — and you end up with a massive, unexpected cloud bill. Think $50,000 in a single day.

Either way, the damage is real — operational or financial.

This is where Cloudflare CDN acts as a **security layer in front of your servers**. All traffic passes through Cloudflare first. If it is a cache hit, the content is returned from the CDN edge node. If not, the request is forwarded to your server.

Cloudflare has been studying traffic patterns for years and has sophisticated mechanisms for detecting attack patterns. The moment an attack is detected, they can trigger countermeasures — like presenting CAPTCHAs to suspicious users. And because Cloudflare's CDN network is one of the largest in the world, the incoming traffic — even in terabytes or petabytes — gets distributed across their global infrastructure. Your servers never see it.

---

### What Content Goes into a CDN?

#### Static Content

The most obvious and widely used category is **static content** — content that does not change frequently:

- JavaScript bundles
- CSS bundles
- HTML files
- Images and videos
- Fonts

When you deploy a frontend application — say, a React single-page application (SPA) — it gets compiled into a bundle of JavaScript, CSS, and HTML. Instead of serving this bundle from your primary server every time, you cache it across CDN nodes. Users requesting your frontend application get it from the nearest CDN edge, not from your origin server. This is the standard deployment pattern for SPAs, static sites, blogs, and similar applications.

#### API Responses

Beyond static content, you can also cache certain **API responses** in your CDN. A good example is a product catalog on an e-commerce platform. If the catalog does not change frequently, there is no reason to hit your primary server every time a user loads it — you can serve it from the CDN cache.

Of course, this raises the question of stale data. CDNs handle this through **cache invalidation**, which in CDN terminology is called **purging the cache**. Cloudflare, for example, lets you attach **tags** to your cached content. When something changes — say, a user publishes a new blog post — you purge all cached content associated with that user's tag, and the CDN fetches fresh data from your primary server to cache again. This gives you granular control over what gets invalidated and when.

---

## Edge Computing

### What "Edge" Means

The word "edge" in the context of CDNs has always referred to CDN nodes being placed at the **edge of the network** — the first point of contact between a user's request and the internet infrastructure. CDN providers strategically place their nodes in collaboration with **ISPs (Internet Service Providers)**, so that a request from Tokyo hits a CDN node right at the entry point of the network, before it ever needs to travel further.

This is why CDN responses are so fast — they are positioned at the literal edge of the network.

### Traditional CDNs vs. Edge Computing

Traditionally, CDN nodes have been used purely for **serving static content**. A user requests a JavaScript bundle or an image, the CDN finds the file and sends it back. There is no computation happening — it is a straightforward file lookup and delivery.

**Edge computing** changes this. With edge computing, a request arrives at the CDN edge node and some **processing or computation happens there**, before the response is returned. That processing at the edge node layer is what we call edge computing.

### Why Edge Computing is Faster

Primary data centers like `us-east-1` on AWS are powerful but few in number. CDN edge nodes are far more numerous, because they are built on top of ISP infrastructure distributed all over the world — there are simply more of them, and they are physically closer to users.

Even if the processing itself takes the same amount of time it would on a primary server, the **round-trip latency is dramatically lower** because the user's request does not travel far. The result is that edge computing responses feel faster than traditional server-based responses.

### Common Use Cases for Edge Computing

#### Authentication

Consider a stateful authentication setup. A user in Tokyo sends a request with a session cookie. Traditionally, this request goes to your primary server in the US. The server checks the session ID against a database or Redis, determines the session is invalid, and sends back a `401 Unauthorized` response — after 100 milliseconds.

With edge computing, you run the same authentication logic at the edge node. If the session is invalid, the edge node returns `401` immediately — in **2 to 3 milliseconds**. Your primary server is never touched. Only requests from **authenticated users** are forwarded to your origin server, significantly reducing unnecessary traffic.

#### Localization and User Customization

Edge nodes know which region they are serving. If a request comes from Tokyo and the user's browser language is set to Japanese, the edge node can immediately serve the Japanese version of your content — without the request ever needing to reach your primary server. This makes location-based customization and user preference handling very fast.

Other solid use cases for edge computing include:

- **Validation** — Rejecting malformed requests at the edge before they consume server resources
- **Routing** — Deciding which backend service or server a request should be forwarded to based on request properties

---

### Constraints of Edge Computing

If edge computing is so fast and powerful, why not run everything there? The answer is **resource constraints**.

#### Limited Hardware Resources

CDN edge nodes are typically hosted on ISP infrastructure. ISPs' primary job is routing internet traffic — CDN is something they do in collaboration with providers like Cloudflare. As a result, the hardware at edge nodes is far less powerful than a primary data center server:

- A primary data center might have **16 GB of RAM** and multiple high-performance CPU cores
- An edge node might have **1 GB of RAM** and a single low-power CPU core

This severely limits the kinds of workloads you can run at the edge.

#### Runtime Restrictions

Edge computing environments come with strict runtime constraints. For example, **Cloudflare Workers** — one of the most popular edge computing platforms — uses **V8 isolates**, the JavaScript runtime from Chrome. But this environment:

- Cannot interact with the file system
- Cannot use TCP protocols directly
- Has limited access to system resources

These constraints exist by design, to keep edge computation lightweight and fast.

#### The Takeaway

Edge nodes and primary servers are **complementary**, not competitive. The edge handles lightweight, latency-sensitive tasks — authentication, validation, routing, localization — and forwards only the right requests to the primary server. The primary server handles all the heavy lifting. Together, they provide a seamless experience to users.

---

## Asynchronous Processing and Background Jobs

### The Core Problem

When we talk about reducing latency in a backend application — specifically **perceived latency**, the latency experienced by the user on the frontend — asynchronous processing is one of the most effective solutions available. And unlike horizontal scaling, which you typically adopt only after crossing certain traffic thresholds, **asynchronous processing is something you usually implement from the start** because of the immediate and obvious benefits it provides.

### How Synchronous Processing Works

In a typical synchronous API flow:

1. The user sends an HTTP request from the browser
2. The server performs some processing — validates the request, runs business logic, queries the database
3. Only after all processing is complete does the server send a response back

This is the right behavior for operations where **the user needs to see the result immediately**. For example, if a user updates their name from A to B on their profile page:

- The server runs an `UPDATE` query against the database
- Only after the database confirms success does the server respond with `200 OK`
- The user refreshes the page and immediately sees their new name

You cannot tell the user the update was successful before the database has confirmed it. The operation must be synchronous.

### When Synchronous Behavior is Unnecessary

However, there is a whole category of operations where **the user does not need to see the result instantly**. For these operations, making the user wait is wasted time.

A classic example is **sending an invitation email** in a SaaS application — the kind of team invite you see in tools like Jira or Notion.

#### The Synchronous Flow (Inefficient)

Here is what the typical synchronous flow looks like:

1. The user types an email address (`user1@gmail.com`) and clicks Invite
2. The browser makes an HTTP API call to the server
3. The server validates the request and checks the database to confirm this user is not already a team member (~50–100 ms)
4. The server saves an entry in an `invites` table with a `pending` status (~50 ms)
5. The server makes an API call to an external email provider like Mailchimp, SendGrid, or Resend to send the invitation email (~200–300 ms)
6. Only after the email provider confirms success does the server respond with `200 OK`

**Total user wait time: ~400 milliseconds**

The user sees a spinner for 400 milliseconds before seeing a success message.

#### The Asynchronous Flow (Efficient)

Now consider what changes if we decouple the email sending from the response:

1. The user types an email address and clicks Invite
2. The browser makes an HTTP API call to the server
3. The server validates the request and saves the invite entry in the database (~100 ms)
4. **The server immediately responds with `200 OK`** — the user sees a success message
5. In the background, the server **pushes a "send email" task into a queue** (Redis Queue or RabbitMQ)
6. A **worker (consumer)** picks up the task from the queue and makes the API call to the email provider — independently, without blocking the user

**Total user wait time: ~100 milliseconds**

The user does not need to know any of this background activity. They do not need to see the email leave the server in real time — they just need to know the invite was recorded. The invited user will eventually receive the email, come to the platform, and accept or decline.

---

### How the Queue System Works

The queue-based approach involves two roles:

- **Producer** — Your primary server, which creates tasks (jobs) and pushes them into the queue
- **Consumer / Worker** — A process that watches the queue, picks up tasks, and executes them

The consumer can be part of the same codebase as your server (a worker thread spun up from the same application), or it can be a completely separate codebase. The separate codebase approach is common in high-traffic systems because it lets you **independently scale your workers** based on queue depth, without scaling your entire server.

Common queue technologies:

- **Redis Queue (BullMQ for Node.js)** — BullMQ is one of the most popular background job libraries for Node.js. It uses Redis internally and handles error retrying, rate limiting, and other production-grade concerns out of the box.
- **RabbitMQ** — Another widely used option for message queue-based task processing
- **Kafka** — For event-driven architectures at scale

---

### More Examples of Asynchronous Processing

Sending invitation emails is one of the most cited examples, but the same pattern applies broadly.

#### Sending Notifications

Users do not expect notifications to appear the millisecond they perform an action. A slight delay is perfectly acceptable and is the standard user experience. Notifications are an ideal candidate for async processing.

#### Video Processing (YouTube's Approach)

When you upload a video to YouTube, the experience is:

1. You select a file and click Upload
2. You must keep the tab open while the file is being uploaded (the browser is reading bytes from your file system and streaming them to YouTube's servers)
3. After the upload completes, you can close the tab

In the background, YouTube pushes multiple tasks into a queue:

- Generate thumbnails
- Encode the video in multiple resolutions (HD, 4K, etc.)
- Generate subtitles

These tasks may run in parallel or in sequence depending on their internal logic. As a user, you might wait 10 minutes or 20 minutes. You do not expect instant results — and that is entirely acceptable. Video processing is a textbook case for asynchronous processing.

#### Account Deletion

This is a particularly instructive example. Consider a SaaS to-do application where a user has been active for 10 years and has accumulated a million to-dos, plus data across seven or eight other database tables.

**The naive synchronous approach:**

1. User clicks "Delete Account"
2. The server runs `DELETE` queries across all eight tables in a database transaction
3. Each deletion takes ~5 ms per table, so eight tables = ~40 ms, plus query planning and business logic
4. The total response time can reach **8 seconds**

8 seconds is an unacceptably bad user experience, regardless of what the operation is. Asking users to wait while staring at a spinner for 8 seconds is not something you can reasonably expect.

**The asynchronous approach:**

1. User clicks "Delete Account"
2. The server does minimal validation — confirms the user exists and is authenticated (~50–100 ms)
3. The server immediately responds with success and the browser logs the user out
4. In the background, the server pushes a `delete_user` task (with the user ID) into the queue
5. A worker picks it up and runs all the deletion queries — taking however long it takes, 5 seconds or 30 seconds — without the user waiting

The user only saw the spinner for ~100 milliseconds. Everything else happens in the background after they are already gone.

---

### How to Identify Which Tasks Should Be Asynchronous

The key question to ask is: **Does the user need to see the result of this operation immediately?**

Operations that are good candidates for asynchronous processing:

- Sending emails
- Sending push notifications
- Deleting user data
- Video uploads and video encoding
- Image uploads and image resizing
- Generating reports or exports
- Any external API call where the result does not need to be in the immediate response

If the user does not expect instant feedback — or if the operation involves an external system where response time is unpredictable — it is a strong candidate for offloading to a queue.

---

### Implementation Recommendation

For most Node.js backends, the simplest and most production-ready starting point is:

- Spin up a **managed Redis instance** (or use a managed service like Upstash)
- Use **BullMQ** as your queue library, which wraps Redis and provides built-in support for job retries, rate limiting, delays, concurrency control, and failure handling

This combination gives you a robust, scalable asynchronous processing setup without a lot of infrastructure complexity. As your scale grows, you can move workers into their own independently deployable services.

> **Asynchronous processing is not something to reach for only when scaling becomes a problem. It is a pattern to apply from the start, for any operation where immediate user feedback is not required. It is one of the easiest and highest-leverage ways to improve perceived performance in a backend system.**

---

## Summary

- **CDNs** solve the fundamental physics problem of network latency by caching content at edge nodes that are geographically close to users, reducing round-trip time from 100 ms to 2–3 ms
- The three main benefits of CDNs are: **lower latency**, **reduced load on origin servers**, and **DDoS protection**
- **Static content** (JS bundles, CSS, images, fonts, HTML) and certain **API responses** (like product catalogs) are the primary candidates for CDN caching
- CDN **cache invalidation (purging)** lets you remove stale content using tag-based strategies, so you are never serving outdated data
- **Edge computing** extends CDNs from pure file delivery to lightweight computation at the edge — authentication, validation, routing, and localization are the primary use cases
- Edge nodes have **hardware and runtime constraints** (limited RAM, no file system access, no TCP) that prevent them from replacing primary servers, but they work powerfully as a complementary layer
- **Asynchronous processing** decouples task execution from the HTTP request-response cycle by pushing jobs into a queue and processing them via workers in the background
- Common async use cases: **sending emails, notifications, account deletion, video and image processing**
- The identifying question is: **does the user need to see this result immediately?** If not, it belongs in a queue
- **BullMQ + Redis** is the recommended starting stack for background job processing in Node.js applications

# Microservices, Serverless Computing, and Scaling Mental Models

## Overview

This chapter covers two of the most discussed architectural patterns in modern backend engineering — **microservices** and **serverless computing** — and closes with a set of practical mental models for thinking about performance and scaling decisions. These are not just theoretical concepts; they are decisions you will face regularly as a backend engineer, and understanding the trade-offs clearly is what separates good engineers from great ones.

---

## Microservices vs. Monolith

### What is a Monolith?

A **monolith** is a backend application that is treated as a **single deployable unit**. It does not matter how many features the application has — authentication, order processing, notifications, payments, webhooks — all of these are just different files or modules living inside the same codebase. They interact with each other, they function together, and every time you make a change and deploy, you deploy all of them together as one single process.

If you are horizontally scaling, you run multiple instances of that same process — but each instance still contains everything.

Monoliths are intuitive and easy to manage:

- **Simple to develop** — all your code is in one place, one GitHub repository
- **Simple to test** — everything runs in one process, so you can test interactions between modules easily
- **Simple to deploy** — one build, one deployment artifact
- **Simple to refactor** — you make a change in one module, cross-reference another module in the same codebase, and deploy the whole thing

Given all this, a fair question is: why would you ever want anything else?

---

### Why Microservices Exist — The Real Reason

Here is the key insight that most people miss: **microservices are not primarily about scaling machine performance. They are about scaling your team.**

You can always horizontally scale a monolith to handle more traffic. The reasons people move to microservices are almost entirely organizational and operational, not computational. Microservices become relevant when you have a large engineering team — and large here means more than 100 or 200 developers working on the same application.

---

### Problems with Monoliths at Scale

#### 1. Deployment Dependency

Imagine a large e-commerce backend with separate teams working on payments, order processing, and notifications. The payments team finishes a critical feature and wants to deploy it immediately so users can start using it. But there are some half-finished changes from the notifications team sitting on the main branch — not broken, but not ready to ship to users either.

In a monolith, since you deploy everything as a single unit, the payments team cannot deploy independently. They are blocked by the state of another team's work.

Yes, there are workarounds — feature flags, stricter branch management policies. But at 500 or 1,000 developers, where teams are working rapidly and in parallel, this problem becomes a constant friction point that slows everyone down.

#### 2. Independent Scaling

Take those same three modules. The notifications module is lightweight — it inserts some database rows and manages some WebSocket connections. The payments module and order processing module are resource-intensive — they require significant CPU, memory, and can spike heavily under traffic.

In a monolith, when you scale, you scale the entire application. You cannot say "scale payments but leave notifications as-is." Every new instance includes the notifications code running on expensive hardware that it does not need. With microservices, each service can be scaled independently based on its own resource requirements and traffic patterns.

#### 3. Technology Stack Flexibility

Consider a blogging platform like Medium or Hashnode. One part of the backend handles Markdown parsing and rendering — tasks where the Node.js and Python ecosystems have rich, mature libraries. Another part handles image resizing and manipulation — a CPU-bound task where a systems language like **Go** or **Rust** can deliver 10x better performance than Python or Node.js (50 ms vs. 500 ms for the same operation).

In a monolith, you are locked into a single language and runtime for the entire application. With microservices, you can build the Markdown service in Node.js using the right npm package and the image processing service in Go or Rust for raw performance — and deploy each independently.

---

### Trade-offs and Disadvantages of Microservices

Every solution comes with trade-offs. Microservices come with significant complexity costs that you need to understand before deciding they are right for you.

#### 1. Network Complexity

In a monolith, one module calling another is a simple in-process function call — essentially free. In a microservices architecture, that same call becomes a **network call** — either HTTP or gRPC. Network calls introduce:

- **Latency** — even internal service-to-service calls add measurable overhead
- **Failure modes** — HTTP calls can fail. You now have to think about retries, timeouts, circuit breakers, and failure handling that simply did not exist before

#### 2. Debugging Complexity

In a monolith, debugging a request means looking at one log file. In a microservices architecture, a single user request might touch the load balancer, then the order service, then the payment service, then the notification service. Debugging that request means correlating logs across four different services simultaneously.

You have to adopt **distributed tracing** tools and architect your logging in a way that makes it possible to follow a single request across service boundaries. This is a non-trivial operational investment.

#### 3. Data Consistency

In a mature microservices architecture, each service typically owns its own database instance. This introduces the challenge of **data consistency across databases**. Even with replication, there is always some amount of replication lag. Changes in one database may not be immediately visible in another. Managing this — and building systems that handle the eventual consistency correctly — is a complex and ongoing engineering problem.

---

### When Should You Use Microservices?

Given all these trade-offs, the bar for adopting microservices should be high. Consider microservices when you have clear, affirmative answers to these questions:

- **Do you have a large team?** Large teams naturally have clear organizational boundaries. Microservices align technical boundaries with human organizational boundaries — which is why the architecture makes sense for them.
- **Do different parts of your system have genuinely different scaling needs?** If certain services need to scale independently, microservices give you that control.
- **Do different parts of your system require different technology stacks?** If you have a legitimate need for multiple languages or runtimes across different domains of your application, microservices enable that.

Unless you have clear answers to these questions, the complexity of microservices almost always outweighs the benefits. **A monolith that is well-structured and horizontally scaled is a perfectly valid architecture for the vast majority of applications.**

---

## Serverless Computing

### What Came Before Serverless — The Traditional Server Model

To understand serverless, you first need to understand what it replaces.

In the traditional model, you provision a **Virtual Machine (VM)** from a cloud provider — an EC2 instance on AWS, for example. That VM comes with a Linux operating system (typically Ubuntu or another Unix distribution). You then:

1. Install the software your application needs — Nginx, Docker, your language runtime, etc.
2. Configure your application to fetch and build your code
3. Expose your application to the network
4. **Manage that server for its entire lifetime** — DNS, configuration, updates, security patches, and everything else

This model has been reliable for decades. You know exactly what you are getting: a machine with 2 CPU cores, 16 GB of RAM, 30 GB of SSD storage, and 1 TB of monthly bandwidth. Your application runs on that machine and serves requests as they arrive.

---

### The Problems with Traditional Servers

#### Capacity Planning

The biggest challenge with traditional servers is **capacity planning** — deciding in advance how many servers you need and what specifications each one should have. You have to predict user traffic before deploying.

Two things go wrong:

- **Underprovision** — A traffic spike hits (from a viral blog post, a campaign, Black Friday). Your server cannot handle it. Requests slow down or fail. New users churn. Existing users look for alternatives. You suffer both reputation damage and financial loss.
- **Overprovision** — To avoid the above, you provision 32 GB of RAM when 8 GB would have been enough. You end up paying $5,000 for a week of compute when $500 would have covered you. The extra capacity sits idle and you pay for it regardless.

Neither outcome is good, and predicting user traffic precisely is essentially impossible.

#### Autoscaling — The Partial Solution

Autoscaling addresses underprovision and overprovision by automatically spinning up new server instances when resource usage crosses a threshold (say, 70% memory usage). When traffic drops, it scales back down.

Autoscaling is standard practice, but it comes with its own limitations:

- **Spin-up time** — Launching a new server instance takes time. You have to boot an operating system, pull and build your code, start the application process, and register it with the load balancer. This can take anywhere from a few seconds to a few minutes. During a sudden, sharp traffic spike, autoscaling may not react fast enough to prevent errors.
- **Maximum instance limits** — You have to configure a maximum. Set it too low and you are back to the underprovision problem. Set it too high and a traffic spike (or a DDoS attack) causes your instance count to hit 500 or 600, generating a $100,000 cloud bill in a single day.
- **Reactiveness** — Autoscaling is inherently reactive. It only spins up new instances after it detects that you are already under load. There is always a window of time between when load exceeds capacity and when new capacity is ready.
- **Always-on minimum cost** — Even with autoscaling, you have a minimum number of instances running at all times. You pay for those instances 24/7, whether they are serving 500 requests per second or zero. For applications with unpredictable or bursty traffic patterns, this is wasteful.

---

### What Serverless Is

Serverless computing eliminates the need to think about servers, operating systems, or machine capacity. You provide only two things:

1. **Your code** — written as discrete functions
2. **The events that trigger those functions** — typically HTTP routes, queue messages, file uploads, database changes, etc.

Everything else — the machines, the OS, the runtime environment, the scaling — is handled entirely by your serverless provider.

#### How a Serverless Request Works

Instead of your request going directly to an always-on server, it goes through an **API Gateway**. The API Gateway maps incoming HTTP routes to the corresponding serverless function. When a request arrives:

1. The API Gateway receives the HTTP call
2. Based on the route, it identifies the appropriate function
3. The serverless provider **spins up an instance** to run that function (if one is not already available)
4. The function executes your code and sends a response back
5. The instance either sits in a pool briefly — ready to handle the next request — or is terminated, depending on your provider's configuration

The key difference from traditional servers: **no machine is provisioned until a request actually arrives.** After the response is sent, the machine is released.

#### The Pricing Model

Traditional servers charge you 24/7 — as long as the server is running, you pay for it.

Serverless pricing is based purely on **actual execution time**. You pay only for the CPU and memory consumed while your function is actually running. Your monthly bill is the sum of all the milliseconds your code was executing. During periods of zero traffic, your cost is zero.

This makes serverless highly cost-effective for applications with irregular, bursty, or unpredictable traffic patterns.

---

### The Problems with Serverless

No solution comes without trade-offs. Serverless has three major ones.

#### 1. Cold Start Times

Because no machine is pre-assigned to your application, the very first request — or any request after a period of inactivity — requires spinning up a new instance from scratch. This spin-up time is called the **cold start**.

Cold start time comes from two sources:

- **Booting a new operating system or VM** — Traditional hypervisor-based VMs can be slow to boot. AWS Lambda addresses this with **Firecracker**, a microVM technology that boots extremely fast and has become the industry standard for serverless infrastructure.
- **Language runtime initialization** — Interpreted languages like JavaScript and Python have minimal startup overhead because they execute code line by line without a compilation phase. Languages like Java have a JVM startup cost. This is one reason **Cloudflare Workers** use V8 isolates (the JavaScript runtime from Chrome) — V8 isolates can boot in 0 to 1 milliseconds, giving Cloudflare Workers total cold start times in the range of 5 milliseconds.

Despite these advances, cold start latency will always exist in serverless architectures. Traditional always-on servers are inherently faster for the first request because they are always ready. This is a fundamental trade-off, not an engineering problem to be completely solved.

A common workaround is sending **automated pings** to keep a few instances warm at all times. However, if you rely on this too heavily, you end up with instances running 24/7 — which defeats the entire purpose of going serverless.

#### 2. Execution Limits

Serverless providers enforce limits on how long a function can run. AWS Lambda, for example, has a maximum execution time of 15 minutes. If you have a long-running operation — say, a process that takes 30 minutes — it will fail halfway through.

This makes serverless unsuitable for long-running procedures, persistent WebSocket connections, and operations that require maintaining state across multiple requests.

#### 3. Statelessness

Traditional servers are stateful. They maintain persistent TCP connections to databases, hold WebSocket connections open with clients, and can store temporary data in memory across requests. This statefulness is central to how most backend architectures work.

Serverless functions are essentially **stateless**. Because instances are short-lived and machines are swapped out continuously, you cannot rely on:

- Persistent database TCP connection pools
- In-memory state surviving across requests
- WebSocket connections that outlive a single function execution

All of these behaviors have to be reimagined. You need serverless-compatible databases (connection poolers like PgBouncer, or serverless-native databases), external state stores, and different approaches to real-time communication. Adopting serverless is not just a deployment change — it requires architectural changes throughout your backend.

---

### When to Use Serverless

Given these trade-offs, serverless is not a universal replacement for traditional servers. It is a powerful tool for specific use cases.

**Serverless is a poor fit for:**
- **Latency-sensitive user-facing applications** — Banking, payments, and any application where even occasional cold start latency is unacceptable
- **Long-running operations** — Any process that exceeds the provider's execution time limits
- **Applications requiring many persistent database connections** — Connection pooling is complex in serverless environments and requires additional infrastructure
- **WebSocket-heavy applications** — Maintaining long-lived connections is fundamentally incompatible with the stateless serverless model

**Serverless is an excellent fit for:**
- **Video and image processing** — Encoding, resizing, and thumbnail generation are irregular, compute-intensive tasks. Rather than paying for always-on heavy infrastructure, you spin up a serverless function per job and pay only for actual processing time.
- **Event-driven pipelines** — Processing messages from queues, responding to file uploads, reacting to database change events. These are natural fits for the event-triggered serverless model.
- **Scheduled or batch jobs** — Tasks that run periodically (nightly reports, data exports) rather than constantly.
- **Auxiliary backend operations** — Webhook handlers, background notifications, and other non-latency-critical operations that happen infrequently.

> **The current industry is somewhat overhyped about serverless.** It is a genuinely powerful tool for specific use cases, but it is not a universal replacement for traditional servers. Understanding where it fits, why it was invented, how it works, and when to use it makes it a valuable addition to your toolkit — not a default choice.

---

## Key Takeaways and Mental Models for Scaling

Having covered a broad range of topics — latency, throughput, database optimization, caching, vertical and horizontal scaling, distributed systems, API gateways, load balancers, microservices, and serverless — a few mental models tie all of this together.

### 1. Always Start With the Problem, Not the Solution

Every technique discussed in this series is a solution. Before reaching for any of them, you need to know precisely what problem you are solving.

**Measure your system first.** Use observability tools — logs, metrics, and distributed traces — to find out exactly where time is being spent. Tools like **Prometheus + Grafana** (for a custom, self-hosted setup) or **New Relic** (for a managed, easier-to-configure option) let you trace every request and identify your actual bottlenecks.

You need specific answers, not general ones:
- Is your database slow? Is it slow because of a missing index, or because of missing sharding?
- Is your API endpoint slow? Is it waiting on an external API call that should be moved to a background queue?

Without these specific answers, you risk committing the worst mistake in performance engineering: **fixing the wrong bottleneck**. When you fix the wrong bottleneck, you get the impression that you have improved the system — while the actual problem remains, and may compound over time.

> Measure everything. Profile your requests. Find the specific bottleneck. Only then choose the solution.

---

### 2. Always Prefer Simple Solutions

Complexity has a cost — always. Every component you add to a system is another component that can fail, another component you have to monitor, another component your team has to understand and operate.

Some examples of this principle in practice:

- **Vertical scaling before horizontal scaling** — A larger single server is simpler to manage than a distributed cluster. Exhaust vertical scaling before introducing the complexity of horizontal scaling.
- **Database indexing before Redis caching** — Adding the right index to a slow query is a far simpler solution than placing a Redis cache in front of your entire database layer. Try the simpler fix first.
- **Monolith before microservices** — A well-structured monolith is simpler to develop, test, debug, and deploy than a distributed microservices architecture.

This does not mean always choosing the absolute simplest approach. Sometimes complexity is genuinely necessary. But complexity always requires justification — it always has to earn its place. **Only accept complexity when simplicity is genuinely insufficient.**

---

### 3. Scale for the Problems You Actually Have

You do not need to architect for a million users on day one. Most applications will never reach a million users. Build for the scale you currently have, with a reasonable buffer for growth. As you grow, your observability tools will tell you where the next bottleneck is.

Generic architecture advice from Netflix, Google, or Facebook engineering blogs may not apply to your application. Those systems operate at extreme scale under constraints that are simply not relevant to most applications. **Learning from measuring your own system is far more valuable than importing solutions designed for someone else's problems.**

Your specific application has its own characteristics. The only way to understand them is to measure them.

---

### 4. Implement Observability From Day One

This is the one exception to the "prefer simple solutions" rule. Observability is not something to defer until you reach 50,000 users. It should be in place from the beginning.

**Observability means three things:**
- **Logs** — structured records of what your application is doing
- **Metrics** — quantitative measurements of system behavior over time (request rates, error rates, latency percentiles, queue depths)
- **Traces** — end-to-end records of individual requests as they flow through your system

With proper observability in place from day one:
- You never have to guess where your system is slow
- You get early warning before bottlenecks become crises
- You can make scaling and optimization decisions with confidence, based on real data
- You can diagnose and resolve production issues quickly when they do occur

Production-grade observability is an investment that pays off continuously — not just at scale, but from the very first user.

---

### 5. Performance Optimization is a Mindset, Not a Checklist

Just as security is an ongoing mindset rather than a one-time configuration, **performance engineering is a continuous practice** that develops through experience.

You will build systems, watch them struggle under real traffic, apply optimizations, observe the results, and iterate. Over time — through this cycle of building, measuring, and improving — you develop the intuition to make good architectural decisions earlier.

A backend engineer's job is not to predict every performance problem before it happens. It is to:

- **Build systems that handle problems gracefully when they do occur**
- **Develop the skills to measure, diagnose, and resolve problems quickly**
- **Build with enough observability that nothing is invisible when things go wrong**

No single tutorial, blog post, or video teaches you all of this. It comes from experience with real systems. But the engineers who internalize the habit of measuring first, preferring simplicity, and building with observability from the start — these are the engineers who build systems that scale reliably over time.

---

## Summary

- A **monolith** is a single deployable unit containing all functionality. It is simple to develop, test, and deploy, and is the right starting point for most applications.
- **Microservices** split an application into independently deployable services. Their primary benefit is **organizational** — enabling large teams to work and deploy independently — not raw machine performance.
- Microservices trade-offs include **network complexity**, **harder debugging**, and **data consistency challenges** across distributed databases. Only adopt them when you have a large team, different scaling needs per service, or legitimate multi-technology requirements.
- **Traditional servers** give you predictable, always-on compute but require capacity planning, have autoscaling limitations, and incur 24/7 costs regardless of traffic.
- **Serverless computing** eliminates server management and charges only for actual execution time, but introduces **cold start latency**, **execution time limits**, and **statelessness constraints**.
- Serverless fits best for **event-driven workloads, image and video processing, and bursty or irregular traffic** — not for latency-sensitive, long-running, or stateful applications.
- **Mental model 1 — Measure first:** Never reach for a solution before you know exactly where the bottleneck is.
- **Mental model 2 — Prefer simplicity:** Complexity always has a cost. Only accept it when simplicity is genuinely insufficient.
- **Mental model 3 — Scale for your actual problems:** Build for current scale with headroom. Do not over-engineer for hypothetical traffic.
- **Mental model 4 — Observe from day one:** Logs, metrics, and traces are not optional. They are the foundation of every good scaling decision.
- **Mental model 5 — Performance is a mindset:** It develops through experience, iteration, and the ongoing discipline of measuring everything.