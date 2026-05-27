# Caching

---

# What is Caching?

## Simple Definition

> Caching is a mechanism using which we decrease the amount of time and effort it takes to perform some amount of work.

## Technical Definition

Caching is a mechanism using which we can decrease the amount of time and effort it takes to retrieve data or to perform some kind of operation.

More specifically, caching means:

* Taking a **subset** of some primary data (not all of it)
* Storing that subset in a **location which is faster to access**
* That location also takes **less effort** to retrieve data from

The subset is chosen based on several parameters:

* Frequency of access of the data
* Probability of the data being accessed next
* Time-related patterns of access
* Other usage parameters

---

## Why Caching Matters

Caching is a huge factor in a lot of high-performance applications.

It is especially critical in systems that track latency in:

* Two-digit microseconds
* Single-digit milliseconds

Without caching, many modern platforms would be completely unable to serve millions of users efficiently.

---

# Real-World Examples of Caching

## Example 1: Google Search

### How Google Search Works Without Caching

When you type a query into Google and press Enter, the query goes through a complex algorithm and workflow that involves:

* **Crawling** — discovering web pages across the internet
* **Indexing** — organizing and categorizing those pages
* **Ranking** — determining which results are most relevant

This entire process runs across billions and billions of web pages.

It is **computationally expensive** — meaning it consumes:

* A large amount of CPU
* A large amount of memory
* Significant other computing resources

A query like "what is the weather today" is searched millions of times every day.

Without caching, Google's servers would need to:

* Recompute the results for every single query
* Run all ranking algorithms each time
* Fetch results from scratch every time

This would lead to:

* Very high latency
* Extremely high server load

### How Google Uses Caching

Google uses a **distributed in-memory caching system** to store query results.

The word "distributed" means the caching servers are spread across the entire world — not concentrated in a single location.

**How it works:**

1. A user searches for a query
2. The system first checks whether the result for that query is already present in the cache
3. If the result is found in the cache → this is called a **Cache Hit**
4. The cached result is returned to the user **instantly** — very fast retrieval
5. If the result is **not** found in the cache → this is called a **Cache Miss**
6. The system runs the full workflow (crawling, indexing, ranking)
7. The computed result is **stored in the cache**
8. The next time the same query is made by any user, the result can be served directly from cache

### Key Terms

| Term | Meaning |
|---|---|
| **Cache Hit** | The data we are looking for is found in the cache |
| **Cache Miss** | The data we are looking for is NOT found in the cache |

---

## Example 2: Netflix

### The Scale of Netflix

Netflix is a global streaming platform that delivers content — movies, series, anime — to millions of users all over the world.

For a single movie, Netflix stores **multiple versions** at different resolutions through a process called **encoding**:

* 1080p — for high-speed connections and large screens
* 720p — for medium-speed connections
* 480p — for slower connections and smaller screens

Netflix dynamically sends the most optimized version to each user depending on:

* The user's network speed
* The device being used

This minimizes bandwidth waste and reduces server load.

### The Problem: Delivering Terabytes of Content Globally

Netflix must deliver hundreds and thousands of terabytes of content to millions of users spread across the entire world — with **minimal buffering**.

If all requests went to Netflix's originating servers (located in the US), users from other parts of the world (such as India) would experience much higher latency simply because of geographic distance.

### The Solution: CDN (Content Delivery Network)

Netflix uses a system called a **CDN** — Content Delivery Network.

**How CDN works:**

* Netflix has its own **originating servers** — data centers in the US that store the actual movies and content
* Additionally, at different strategic locations all over the world, Netflix (and CDN providers) set up what are called **Edge Locations** or **Edge Servers**

**Edge Locations** are strategically placed servers whose purpose is to:

* Cache content that is popular in that region
* Serve users from the geographically closest server
* Minimize the latency of data delivery

**Example:**

* A user in India requests a movie
* Instead of the request traveling all the way to the US originating server, it is served from the nearest Edge server in India or South Asia
* The result is minimal buffering and fast content delivery

### Netflix and Caching Intelligence

Netflix does **not** cache all of its content at all Edge locations — that would be too expensive and impractical.

Instead, it uses:

* Machine learning algorithms
* Trend analysis
* Real-time data processing

...to decide **which subset of content** to cache at each edge location depending on what people in that region are watching.

This brings us back to the technical definition of caching:

> Taking a subset of the primary data and placing it in a location that is faster to access.

CDN is also used by many other companies beyond Netflix:

* Platforms like **Vercel** have an edge network where front-end apps are deployed — requests are served from the server closest to the user
* Static assets like JavaScript, HTML, and CSS files are also commonly served through CDNs

---

## Example 3: Twitter / X (Trending Topics)

### The Problem: Expensive Computation for Every Request

Twitter identifies trending topics by:

* Analyzing millions and billions of tweets in real time from all over the world
* Running machine learning-based algorithms
* Running trend-detection algorithms
* Processing terabytes and terabytes of data

This computation is extremely expensive and requires:

* Many GPUs
* Massive infrastructure
* Large amounts of memory and processing power

If Twitter ran this full computation **every time a user opened the trending section**, the servers would crash within minutes — because billions of users are potentially accessing this at the same time.

### The Solution: Caching Trending Topics

Twitter caches the trending topics in an **in-memory key-value store** like Redis.

**Why is it safe to cache trending topics?**

Because trends do not change in seconds or even minutes.

Example:

* If elections are trending in a country, that topic will remain trending for hours or even days
* There is no need to recompute it for every single request

**How it works:**

1. Every few minutes, Twitter runs all the trend-detection algorithms and machine learning models against the tweets from different regions
2. The computed trending topics are stored in a fast in-memory cache like Redis
3. When any user opens the trending section, the data is served **directly from the cache** — instantly
4. This is why trending topics load so quickly even with a reasonably fast internet connection

---

## The Common Pattern Across All Three Examples

From these three examples, a clear pattern emerges:

Caching is used in two major situations:

1. **Heavy Computation** — When computing the result is very expensive and we want to avoid doing it again and again for every request
2. **Heavy Data Delivery** — When we need to deliver large amounts of data to a large number of users, and we want to minimize both latency and server load

---

# Levels of Caching

As a backend engineer, you will primarily encounter three levels of caching:

1. **Network Level Caching**
2. **Hardware Level Caching**
3. **Software Level Caching**

Note: Software-based caching is called that because you interact with it through libraries and software interfaces — but it still ultimately depends on hardware (RAM) to deliver its performance.

---

# Network Level Caching

Two major use cases of caching at the network layer:

1. CDN (Content Delivery Network)
2. DNS (Domain Name System)

---

## 1. CDN — Content Delivery Network

### Core Idea

The whole idea of CDN is to **cache content on servers that are geographically closer to the end users**.

This is why CDN servers are also called:

* **Edge Nodes**
* **Edge Servers**
* **Edge Computing**

The word "edge" means the server is at the edge of the network — closest to the user — rather than at a central originating server far away.

### How CDN Works — Step by Step

**Step 1: User Makes a Request**

A user enters a URL into their browser — for an image, a video, or a web page.

**Step 2: DNS Query**

The browser sends a DNS query to resolve the domain name in the URL into an IP address.

(DNS takes domain names and converts them into IP addresses so browsers can find the actual server.)

**Step 3: CDN DNS Routes to Nearest PoP**

The CDN's DNS system routes the request to the **nearest PoP** (Point of Presence).

**PoP (Point of Presence)** is a geographic region that contains a cluster of multiple Edge Servers.

The CDN DNS routes the request based on multiple parameters:

* The user's geographic location
* Current network conditions
* The user's available bandwidth

**Example:** If the user has a very slow internet connection, the CDN DNS might route to a PoP that has lower-resolution versions of the video, so the content loads without buffering.

**Step 4: Edge Server Checks the Cache**

Once the request reaches the nearest PoP, an Edge Server checks:

> Is the requested content already cached here?

* **Cache Hit** → The content is found. The Edge Server immediately sends it to the user. This is the fast, happy path.
* **Cache Miss** → The content is not cached yet. The Edge Server fetches it from the **originating server** (e.g., Netflix's US data center), caches it locally, and then sends it to the user.

The next time any user in that region requests the same content, it will be served directly from the cache — no trip to the originating server needed.

**Step 5: TTL — Time to Live**

CDNs use a concept called **TTL (Time to Live)** to decide how long to keep a cached piece of content.

* Companies configure a fair TTL duration depending on how often content changes
* After the TTL expires, the next request fetches fresh content from the originating server
* The fresh content is then cached again with a new TTL

**Why TTL matters:**

Without TTL, users might be served outdated versions of content indefinitely — for example, an old version of a web page or a file that has since been updated.

---

## 2. DNS Caching

### Why DNS Needs Caching

DNS resolution is a multi-step recursive process that involves reaching out to multiple different servers to find the IP address of a domain.

Because this process is expensive and happens for **billions of requests at the same time**, DNS relies heavily on caching at multiple levels.

### How DNS Resolution Works — Step by Step

**Step 1: User Enters a URL**

A user types `example.com` into their browser and presses Enter.

**Step 2: DNS Query to a Recursive Resolver**

The user's device sends a DNS query to a **Recursive Resolver**.

The Recursive Resolver is typically provided by:

* The user's **ISP** (Internet Service Provider) — e.g., Jio, ACT, Airtel in India
* Or a public DNS provider like **Google Public DNS** or **Cloudflare DNS**

It is called "recursive" because it recursively queries different servers until it finds the IP address.

**Step 3: Recursive Resolver Checks Its Local Cache**

The Recursive Resolver first checks its own local cache:

* **Cache Hit** → The IP address for `example.com` is found. The resolver immediately returns it to the browser.
* **Cache Miss** → Not found. The resolver proceeds to query other servers.

**Step 4: Query the Root Servers**

The resolver queries one of the **Root Servers**.

There are around 13–14 root servers spread across the world.

Root Servers do **not** have the IP address of `example.com`, but they have the **addresses of Top-Level Domain (TLD) servers** — like `.com`, `.org`, `.in`, etc.

The root server returns the address of the `.com` TLD server.

**Step 5: Query the TLD Server**

The resolver queries the **TLD Server** for `.com`.

The TLD server also does not have the specific IP of `example.com`, but it has the address of the **Authoritative Name Server** for `example.com`.

**Step 6: Query the Authoritative Name Server**

The resolver queries the **Authoritative Name Server** for `example.com`.

This server **has the actual IP address** of `example.com`.

It returns the IP address to the Recursive Resolver, which returns it to the browser.

### Multiple Layers of DNS Caching

DNS caching happens at multiple layers to avoid going through this entire recursive process for every single request:

**Layer 1 — Operating System Cache**

Most operating systems (Windows, macOS, Linux) maintain a **local DNS cache**.

When you request a domain, your OS checks its own cache first.

* **Cache Hit** → IP is returned immediately. No need to contact the Recursive Resolver at all.
* **Cache Miss** → Proceeds to the browser cache.

**Layer 2 — Browser Cache**

Modern browsers (Chrome, Firefox, etc.) maintain their **own DNS cache**.

If the OS cache misses, the browser checks its own DNS cache.

* **Cache Hit** → IP is returned. No need to contact the Recursive Resolver.
* **Cache Miss** → The query is sent to the Recursive Resolver.

**Layer 3 — Recursive Resolver Cache**

The Recursive Resolver (from your ISP or public DNS) also maintains its own cache.

If the IP is found here, it is returned directly without querying root servers, TLD servers, or authoritative name servers.

**Layer 4 — Authoritative Name Server Cache**

Some Authoritative Name Servers also implement caching to further reduce the load of repeated lookups.

**Summary of DNS Cache Layers:**

```
User's Device
    → OS Cache (Layer 1)
        → Browser Cache (Layer 2)
            → Recursive Resolver Cache (Layer 3)
                → Root Servers
                    → TLD Servers
                        → Authoritative Name Server Cache (Layer 4)
                            → Authoritative Name Server (final answer)
```

Each layer of cache exists to **skip as many of the expensive recursive steps as possible**.

---

# Hardware Level Caching

## CPU Cache Hierarchy

If you have a computer science background, you are likely familiar with CPU-level caches. Caching is implemented at the hardware level to make computations and CPU-level operations faster.

The memory hierarchy from fastest to slowest:

```
L1 Cache  (fastest, smallest)
L2 Cache
L3 Cache  (shared between CPU cores)
RAM / Main Memory (Random Access Memory)
Secondary Storage (Hard Disk / SSD)
Network Storage
```

**How it works:**

* The CPU keeps small amounts of data in L1, L2, and L3 caches for extremely fast access
* Data that is frequently used or likely to be used soon is loaded into these caches by **predictive algorithms** inside the CPU

**Example — Arrays and Sequential Access:**

When you start traversing the elements of an array sequentially (e.g., index 0, 1, 2, 3...), the CPU's predictive algorithms detect the pattern and **pre-load the entire array into cache** (L1 or L2).

This is why **sequential access of array elements is extremely fast** — the data is already in the CPU cache before you even ask for it.

---

## RAM — Random Access Memory (Primary Storage)

RAM is also called **Main Memory** or **Primary Storage**.

### Why RAM is Fast

RAM stores data in capacitors and uses **direct electrical signals** to access any memory address instantly — no matter which part of memory you are accessing.

This is why it is called **Random Access Memory**:

> It does not matter from which direction or in what order you access data — the speed is almost constant.

Compare this to a **Hard Disk Drive (HDD)**:

* An HDD has a mechanical head that physically revolves over the spinning disk to find data
* This is a **mechanical operation** and is much slower

### Limitations of RAM

| Property | RAM | Secondary Storage (HDD/SSD) |
|---|---|---|
| **Speed** | Very fast | Slower |
| **Capacity** | Limited | Very large |
| **Persistence** | Volatile (data lost on power off) | Non-volatile (data persists) |
| **Cost** | More expensive per GB | Cheaper per GB |

Because RAM is **volatile**, data stored in RAM is lost when the machine is turned off or restarted.

Because of this tradeoff, **RAM cannot replace secondary storage** — they serve different purposes.

---

# Software Level Caching (In-Memory Databases)

## What Are In-Memory Databases?

Technologies like **Redis** and **Memcached** (also known as Mcache) are called **in-memory key-value store databases**.

They are called "in-memory" because they store data in **RAM (primary storage)** rather than on disk.

This is also why data access from these systems is **extremely fast** compared to traditional disk-based databases.

### Cloud Equivalents

On cloud platforms, managed services provide similar in-memory caching:

* **AWS ElastiCache** — a managed in-memory caching service on Amazon Web Services

---

## Why "In-Memory" Databases?

The four components in the name "in-memory key-value NoSQL database" explained:

**1. In-Memory**

Unlike traditional databases like PostgreSQL or MySQL that store data on disk, technologies like Redis store data in RAM — which makes data access operations extremely fast.

**2. Key-Value Based**

Instead of strict relational schemas with tables and rows, these databases store data as simple **key-value pairs**:

* A **key** — a unique identifier
* A **value** — which can be anything: a string, a number, a list, a JSON object, etc.

This makes storage and retrieval very simple:

```
SET user:session:abc123  "{ userId: 42, role: 'admin' }"
GET user:session:abc123
→ "{ userId: 42, role: 'admin' }"
```

**3. NoSQL**

These databases do not enforce the strict schemas of traditional SQL (relational) databases. There are no tables to define, no foreign keys, no complex joins required.

**4. Database**

Despite being in-memory and simple in structure, they are still full databases — they support persistence through mechanisms that write data to secondary storage behind the scenes, so data can be recovered when the program restarts.

---

## Persistence in In-Memory Databases

Technologies like Redis handle the tension between speed and persistence like this:

* **At runtime** — all reads and writes happen against data stored in RAM (fast)
* **For persistence** — Redis periodically writes data to secondary storage (disk) in the background
* **On restart** — data is loaded from disk back into RAM

This gives you the **speed of RAM** during operation and the **safety of disk** for persistence.

---

# Caching Strategies

When using in-memory databases like Redis in backend development, there are two primary caching strategies:

---

## 1. Lazy Caching (Cache-Aside)

This is the most common caching pattern.

**How it works:**

1. A client makes a request for a resource
2. The server first checks the cache
3. **Cache Hit** → The resource is found in cache → Return it to the client immediately
4. **Cache Miss** → The resource is not in cache
   * Fetch the resource from the primary database or storage
   * Store the fetched result in the cache
   * Return the result to the client
5. The next time any client requests the same resource, it is served from the cache

**Why "Lazy"?**

Because you do not proactively fill the cache based on predictions. You only cache data **when someone actually requests it**.

**Diagram:**

```
Client Request
    → Check Cache
        → Cache Hit  → Return result immediately ✓
        → Cache Miss → Fetch from DB → Store in Cache → Return result
```

---

## 2. Write-Through Caching

This strategy is primarily about **keeping the cache up to date** whenever data changes.

**How it works:**

Every time a write operation occurs (POST, PUT, PATCH) that changes a resource in the database:

1. Update the **database** with the new data
2. **At the same time**, in the same API call, update the **cache** with the new data

Both operations happen together in the same execution flow.

**Advantage:**

* The cache is **always fresh** — you never serve stale (outdated) data
* No need to worry about cache invalidation for write operations

**Disadvantage:**

* Every write operation carries additional overhead — you must update both the database and the cache simultaneously

---

## Comparison of Caching Strategies

| Property | Lazy Caching | Write-Through Caching |
|---|---|---|
| **When cache is populated** | On first read (after a miss) | On every write |
| **Cache freshness** | May serve stale data | Always fresh |
| **Write overhead** | None | Yes — dual update required |
| **Complexity** | Lower | Higher |
| **Best for** | Read-heavy, infrequent writes | Write-heavy, freshness-critical data |

---

# Eviction Policies

## Why Eviction Policies Are Needed

In-memory caches like Redis use **RAM**, which has a limited capacity compared to disk storage.

At some point, the cache will run out of memory — and when that happens, some existing data must be **evicted (removed)** to make room for new data.

Remember: cache is only supposed to hold a **subset** of the primary storage — the most important or most frequently accessed data. Old or less important data needs to be let go.

**Eviction policies** define the rules for **which data gets removed** when the cache is full.

---

## Eviction Policy Types

### 1. No Eviction

* No eviction policy is configured
* When the cache is full and a new entry needs to be added, the system simply **returns an error**
* Not practical in most production scenarios

---

### 2. LRU — Least Recently Used

The system tracks **when each key was last accessed**.

When the cache is full and a new key needs to be inserted:

* The system finds the key that was **accessed the longest time ago** (least recently used)
* That key is **evicted**
* The new key is inserted in its place

**Example:**

```
Cache keys: [1 (accessed today), 2 (accessed today), 3 (accessed today), 4 (accessed yesterday)]
New key 5 needs to be inserted → Cache is full
→ Key 4 is least recently used → Evict key 4 → Insert key 5
```

---

### 3. LFU — Least Frequently Used

The system tracks **how many times each key has been accessed** over its lifetime in the cache.

When the cache is full:

* The system finds the key with the **lowest access frequency** (accessed the fewest times)
* That key is **evicted**
* The new key is inserted

**Example:**

```
Cache keys:
  Key 1 → accessed 5 times
  Key 2 → accessed 10 times
  Key 3 → accessed 6 times
  Key 4 → accessed 23 times

New key 5 needs to be inserted → Cache is full
→ Key 1 has the lowest frequency → Evict key 1 → Insert key 5
```

---

### 4. TTL-Based Eviction — Time to Live

Each key in Redis can have a **TTL (Time to Live)** configured — a duration after which that key automatically expires and is removed from cache.

When the cache is full and a new key needs to be inserted:

* The system looks for keys whose TTL is **expiring soonest**
* Those keys are evicted first
* The new key is inserted

**Example:**

```
Key A → TTL expires in 5 minutes
Key B → TTL expires in 30 minutes
Key C → TTL expires in 2 minutes

New key D needs to be inserted → Cache is full
→ Key C expires soonest → Evict Key C → Insert Key D
```

TTL can also be used standalone as an **automatic cache invalidation mechanism** — not just for eviction but to ensure that cached data does not stay stale beyond a configured duration.

---

## Eviction Policy Comparison

| Policy | Based On | When to Use |
|---|---|---|
| **No Eviction** | — | Never in production (returns errors when full) |
| **LRU** | Recency of access | General-purpose; good default |
| **LFU** | Frequency of access | When some data is always in demand |
| **TTL-Based** | Time to expiry | When data has a known "freshness window" |

---

# Real-World Use Cases of Redis and In-Memory Caches in Backend Development

---

## 1. Database Query Caching

**The Problem:**

Some SQL queries involve:

* Many table joins
* Large aggregations
* Millions of rows of data

These are **computationally expensive** queries.

If these queries power APIs that are hit frequently (e.g., a dashboard or landing page), the database gets flooded with expensive computation requests — increasing latency and straining resources.

**The Solution:**

Cache the result of the expensive query in Redis with a configured TTL.

**How it works:**

1. An API request comes in
2. Check if the result is in the cache
3. **Cache Hit** → Serve the result from Redis immediately
4. **Cache Miss** → Execute the expensive DB query → Store the result in Redis with a TTL → Return the result to the client
5. For the duration of the TTL, all subsequent requests are served from cache
6. If underlying data changes, manually invalidate or update the cache entry

**Real-World Examples:**

* **Amazon** — caches product details, prices, and inventory data to avoid querying the database for every user who views a product page. During a sale, a single product page might get millions of hits — without caching, the database would be overwhelmed.
* **Twitter / Facebook** — caches user profile data. Profile data changes very infrequently (maybe a few times per year). If a celebrity's profile gets a million hits per day, it makes no sense to query the database each time. The profile is served from cache, and the cache is invalidated only when the user makes a change.

**The Pattern:**

> When an operation is **read-heavy** and writes are **infrequent**, caching is an excellent solution.

---

## 2. Session Storage

**The Problem:**

In a typical authentication flow:

* After a successful login, a **session token** is generated for the user
* This session token must be validated on **every subsequent API call** the user makes
* If sessions are stored in a traditional relational database (PostgreSQL, MySQL), every API call requires a database query — adding latency to every single request

**The Solution:**

Store session tokens in Redis.

**Why Redis for sessions?**

* Fetching session data from Redis (RAM) is **orders of magnitude faster** than fetching from a disk-based relational database
* This reduces latency on every single API call that requires authentication
* It also reduces the load on the primary database

---

## 3. API Response Caching (External API Caching)

**The Problem:**

Your backend calls an external API (e.g., a Weather API) and uses the data to build a response for your frontend.

* Every time your frontend makes a request, your backend makes a request to the external API
* If you have many users making many requests, you quickly accumulate a large number of external API calls
* External APIs often have **rate limits** (maximum requests per minute/hour) or **per-request pricing**
* You will hit your rate limit or run up a large bill

**The Solution:**

Cache the external API response in Redis with an appropriate TTL.

**How it works:**

1. Frontend makes a request to your backend
2. Backend checks if the weather data is already in Redis
3. **Cache Hit** → Return cached data immediately — no external API call needed
4. **Cache Miss** → Call the external Weather API → Cache the result in Redis with a TTL (e.g., 1 hour) → Return the result
5. For the next 1 hour, all frontend requests use the cached value
6. After 1 hour, the TTL expires → next request fetches fresh data from the external API

**Why this works for weather data:**

Weather data does not change in seconds or even minutes — it is safe to cache for a short period (minutes to hours) without serving meaningfully stale data.

---

## 4. Rate Limiting

**The Problem:**

Rate limiting is a mechanism that restricts how many requests a particular client can make within a given time window.

Common reasons to implement rate limiting:

* Prevent abuse from bots
* Protect expensive APIs from being overwhelmed
* Reduce server load

**How Rate Limiting is Typically Implemented:**

Rate limiting is usually implemented as **middleware** that sits between the incoming request and the actual route handler.

**Step-by-Step Flow:**

1. A request arrives at the server
2. The rate limiting middleware extracts the client's IP address from the request headers — typically from the `X-Forwarded-For` header (added by reverse proxies like Nginx)
3. The middleware uses the IP address as a **key** in Redis
4. It fetches the **counter** for that IP from Redis
5. If the counter is below the limit (e.g., 50 requests per minute):
   * Increment the counter
   * Allow the request to proceed
6. If the counter equals or exceeds the limit:
   * Block the request
   * Return HTTP status `429 Too Many Requests`

**Why Redis (Not a Relational Database) for Rate Limiting:**

* Every single incoming request triggers a counter read + write
* If this were done against PostgreSQL or MySQL, it would add significant latency to every API call and flood the database with lightweight counter operations
* Redis counters can be incremented in microseconds — adding near-zero overhead to each request
* Using Redis also keeps the rate limiting load completely separate from the primary database

**Example Redis Counter:**

```
Key:   "rate_limit:192.168.1.1"
Value: 37   (requests made in current 1-minute window)
TTL:   auto-expires after 1 minute (window resets)
```

---

# Summary

## When to Use Caching

| Situation | Use Caching? |
|---|---|
| Very expensive computation that repeats often | ✅ Yes |
| Delivering large data (video, images) to many users globally | ✅ Yes |
| Frequently read data that changes rarely | ✅ Yes |
| External API calls with rate limits or per-request billing | ✅ Yes |
| Session token validation on every request | ✅ Yes |
| Rate limiting counters | ✅ Yes |
| Data that changes on every write | ⚠️ Use write-through or careful TTL |
| Data that must always be perfectly real-time | ❌ Avoid or use very short TTL |

---

## Key Takeaways

* **Caching** = storing a subset of data in a faster-to-access location to reduce time and effort of retrieval
* **Cache Hit** = data found in cache → fast return
* **Cache Miss** = data not in cache → fetch from source → store in cache → return
* **CDN** = network-level cache for geographically distributing content to edge servers
* **DNS Caching** = multi-layer caching at OS, browser, resolver, and authoritative server levels to avoid expensive recursive DNS lookups
* **Hardware Caching** = L1/L2/L3 CPU caches and RAM for fast computation
* **In-Memory Databases (Redis, Memcached)** = software-level caches using RAM for extremely fast key-value data access
* **Lazy Caching** = only cache on first read (after a miss)
* **Write-Through Caching** = update cache on every write to keep it always fresh
* **Eviction Policies** = rules for removing data when cache is full: No Eviction, LRU, LFU, TTL-based
* **TTL (Time to Live)** = configurable duration after which cached data automatically expires

---

## Popular In-Memory Cache Technologies

| Technology | Notes |
|---|---|
| **Redis** | Most popular; open source; supports rich data types; supports persistence |
| **Valkey** | Open-source alternative to Redis |
| **Memcached** | Simpler, high-performance distributed memory cache |
| **AWS ElastiCache** | Managed Redis/Memcached on AWS |

To get started practically:

* Install Redis or Valkey locally
* Use the appropriate client library for your language (e.g., `node-redis` for Node.js)
* Storing and retrieving data is as simple as providing a key and a value — much simpler than SQL queries
* Observe the difference in access speed compared to a traditional relational database