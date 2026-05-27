# Concurrency in Backend Systems — Part 1

## Overview

Concurrency is one of the most important concepts in backend engineering because every backend system, regardless of language or framework, must handle **multiple things happening at once**.

If your server could process only one incoming request at a time, every other user would either:

* wait until the current request finishes, or
* receive an error because the server is busy.

That approach breaks immediately in production systems where thousands of users may send requests simultaneously.

This chapter builds a mental model of how backend systems process multiple requests, how CPUs and operating systems participate in that work, and how programming languages expose concurrency using constructs like `async`, `await`, threads, and event loops.

This is not about framework-specific implementation.

The goal is to understand **what happens underneath your code**.

Understanding this helps with:

* debugging performance issues,
* structuring backend code better,
* making architectural decisions,
* and reasoning about scalability.

---

# Why Concurrency Matters

Most developers learn concurrency through programming language APIs.

Examples:

* JavaScript → `async` / `await`
* Go → goroutines
* Java → threads / virtual threads
* Python → async coroutines / threading

We learn that these tools “make things concurrent,” but often without understanding what they actually do behind the scenes.

The deeper understanding matters because backend applications spend most of their life:

* waiting on databases,
* waiting on network calls,
* waiting on files,
* or waiting on external services.

Concurrency exists so your server does not waste resources while waiting.

---

# The Cost of Not Using Concurrency

Consider a typical API request.

## Request lifecycle

A browser sends a request to your backend server.

The backend:

* receives the request,
* runs routing logic,
* executes business logic,
* makes database queries,
* waits for database response,
* then sends the response back to the client.

Visually:

```text
Browser → Backend Server → Database → Backend Server → Browser
```

---

## The important question

What is your server doing **while waiting** for the database response?

Imagine:

* local DB query → ~1–2 ms
* DB in another availability zone → ~20–30 ms
* DB in another region → ~90–100 ms

During those milliseconds, if your server handles only one request synchronously:

**it is doing nothing.**

The CPU sits idle waiting for network packets.

---

## Quantifying the waste

Modern CPUs can execute roughly:

```text
~3 billion instructions per second
≈ 3 million instructions per millisecond
```

If the server waits 100 ms for a DB response:

```text
3 million × 100 = 300 million instructions
```

The CPU could have executed **300 million instructions**.

But in a synchronous single-request model:

```text
Instructions executed = 0
```

That is pure waste.

---

## Real-world API latency breakdown

A typical medium-complexity API might involve:

* 3–5 database queries
* Redis cache calls
* email service API calls
* filesystem access

Suppose:

* 5 network operations total
* average wait = 50 ms each

Then:

```text
5 × 50ms = 250ms spent waiting on IO
```

Meanwhile CPU computation may only take:

```text
10ms
```

So:

```text
CPU busy = 10ms
Waiting = 250ms
```

Which means:

```text
~95% of time spent waiting
```

This is exactly the problem concurrency solves.

---

# IO Bound vs CPU Bound Operations

Understanding this distinction is critical.

---

## IO Bound

IO-bound means work is blocked waiting for an external resource.

Examples:

* database queries
* HTTP requests
* calling email APIs
* Redis operations
* file upload/download
* reading files
* logging
* stdout / stdin

In these operations, CPU is mostly idle.

It sends the request and waits.

---

## CPU Bound

CPU-bound means the CPU is actively computing.

Examples:

* JSON parsing
* validation logic
* serialization / deserialization
* encryption / decryption
* JWT verification
* image processing
* video encoding
* matrix multiplication / ML workloads

These consume CPU cycles heavily.

---

## Key takeaway

Most backend applications are:

# **IO Bound**

not CPU bound.

That is why concurrency is essential in backend systems.

---

# Concurrency vs Parallelism

These are commonly confused, but they are different.

---

# Parallelism

Parallelism means:

# **Executing multiple instructions at the exact same time**

This requires hardware support.

Example:

* CPU Core 1 executes Task A
* CPU Core 2 executes Task B

Both are running simultaneously.

---

# Concurrency

Concurrency means:

# **Handling multiple tasks at once by switching between them efficiently**

Tasks make progress together.

But at any instant only one may be actively running on the CPU.

This can happen even on a single CPU core.

---

## Short definition

A useful way to remember it:

### Parallelism

> Doing multiple things at once

### Concurrency

> Dealing with multiple things at once

---

# Visualizing Concurrent Request Handling

Assume two requests arrive together.

* Request A
* Request B

Single CPU core.

Timeline:

```text
Request A: CPU work → waiting for DB → CPU work → response
Request B:       waiting → CPU work → waiting → CPU work → response
```

What happens:

1. Request A gets CPU.
2. It processes for a few milliseconds.
3. It makes DB query.
4. While waiting for DB → CPU becomes free.
5. Request B now gets CPU.
6. It processes.
7. It later waits on IO.
8. When A’s DB responds, A becomes runnable again.
9. CPU picks A again.
10. Eventually both finish.

The CPU keeps jumping to whichever task is ready.

This prevents CPU waste.

---

# Event Loop Model (Visual)

Below is a reconstruction of the event loop model described in the transcript:

```text
┌─────────────────────────────────────────────┐
│              EVENT LOOP MODEL              │
└─────────────────────────────────────────────┘

                 Event Loop
                (1 Thread)
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼

 ┌────────────┐ ┌────────────┐ ┌────────────┐
 │ Request A  │ │ Request B  │ │ Request C  │
 │ waiting on │ │ waiting on │ │ ready to   │
 │ DB query   │ │ HTTP call  │ │ run        │
 └────────────┘ └────────────┘ └────────────┘
                                   │
                                   ▼
                     ┌────────────────────────┐
                     │ Loop runs C callback   │
                     └────────────────────────┘

When A's DB responds → callback queued
When B's HTTP responds → callback queued
Event loop processes whichever callback is ready
```

The event loop does not wait for blocked tasks.

It runs whatever is ready.

---

# Why This Matters Practically

When two requests are in progress concurrently:

From the client’s perspective:

```text
Both requests are running at the same time
```

But from CPU perspective:

```text
Only one executes at a given instant
```

That is concurrency.

If multiple CPU cores are available and both execute simultaneously:

That becomes parallelism.

---

# Threading Model Deep Dive

One major way operating systems achieve concurrency is using **threads**.

---

## What is a thread?

A thread is:

> An independent unit of execution managed by the operating system.

When OS creates a thread it also creates:

* a **stack**
* instruction pointer
* metadata for scheduling

---

## Stack stores

* function calls
* local variables
* execution frames

Example:

```js
function main() {
  let a = 3;
  getUsers();
}
```

`main()` and `getUsers()` are tracked in the stack.

Variable `a` also lives there.

---

## Instruction Pointer

Tracks:

> Which instruction the thread is currently executing.

This lets OS resume execution after pausing.

---

# Thread Scheduling

Multiple threads are managed by the OS scheduler.

Scheduler decides:

* which thread runs now
* which pauses
* which blocks
* which resumes later

Each thread gets a **time slice**.

Example:

```text
Thread 1 → 2 ms
Thread 2 → 2 ms
Thread 3 → 2 ms
```

Then CPU switches.

This is called:

# Preemptive Scheduling

because execution can be paused before completion.

---

# Blocking Operations in Threads

If a thread performs:

* DB query
* network request
* file IO

it becomes blocked.

The OS marks it blocked and schedules another thread.

When IO completes:

the thread becomes runnable again.

---

# Shared Memory Between Threads

Threads inside the same process can share memory.

That means:

* heap objects
* global variables
* maps
* pointers

can be accessed by multiple threads.

This makes communication fast.

But it also introduces synchronization problems like race conditions, which will be discussed later.

---

# Thread Overhead & Context Switching

Threads are powerful, but expensive.

## 1. Memory Overhead

Every thread requires stack memory.

Often several KB to MB.

With thousands of requests:

memory usage becomes massive.

---

## 2. Creation Overhead

Creating a thread requires:

* kernel system call
* stack setup
* scheduler registration
* metadata allocation

This adds latency.

---

## 3. Context Switching Overhead

Switching threads requires:

* saving CPU registers
* storing execution state
* loading next thread state
* restoring registers

This consumes CPU time without doing useful application work.

Too many threads → too much context switching → worse performance.

---

# Event Loop Model Deep Dive

The second major concurrency model is the **event loop**.

Instead of many threads:

usually we have:

# One thread + loop + callbacks

---

## How it works

When a task starts an IO operation:

Instead of blocking:

it says:

> “Resume me when this IO completes.”

Then it yields control back to the event loop.

The event loop continues executing other tasks.

When IO finishes:

callback is queued.

Loop picks it up and resumes execution.

---

## Why event loop is efficient

Benefits:

* minimal memory overhead
* no thread-per-request model
* almost no context switching cost
* excellent for IO-heavy workloads

---

# The Biggest Rule of Event Loop Systems

# Never block the event loop.

If CPU-heavy code runs for 100 ms:

the entire loop stops.

During that time:

* no request can progress
* callbacks cannot execute
* latency spikes for everyone

This is why event-loop-based languages like JavaScript rely heavily on:

* callbacks
* promises
* `async`
* `await`

These are language-level tools that make asynchronous behavior easier to express.

---

# End of Part 1

This section covered:

* why concurrency matters in backend systems
* cost of synchronous execution
* IO-bound vs CPU-bound work
* concurrency vs parallelism
* request scheduling
* thread-based concurrency
* thread overhead
* event loop architecture
* why event loops work so well for backend IO workloads

# Concurrency, IO-Bound vs CPU-Bound, and Async Primitives

## Overview

This chapter covers one of the most fundamental concepts in backend engineering — what it means for an operation to be **IO-bound** versus **CPU-bound**, and how different concurrency models (threading, event loops, and virtual threads) handle these operations. By the end, you should have a solid mental model of why certain tools exist and when to use them.

> **The single goal of this chapter:** Understand the difference between IO-bound and CPU-bound operations. Everything else — threads, locks, async/await, goroutines — exists to serve that understanding.

---

## The Threading Model — How It Works

### Handling Multiple Requests with Threads

When a backend uses a threading model for concurrency, it spins up multiple OS-level threads to handle multiple requests simultaneously. Let's walk through what actually happens, step by step.

Imagine two incoming requests — **Request A** and **Request B**.

```
Thread 1 (handling Request A):
─────────────────────────────────────────────────────────
1. Parse incoming HTTP request                  [RUNNING]
2. Call db.query("SELECT * FROM users")
   └─ Sends bytes over TCP to database
   └─ Calls read() on socket, which BLOCKS      [BLOCKED]

OS Scheduler sees Thread 1 blocked, switches to Thread 2

Thread 2 (handling Request B):
─────────────────────────────────────────────────────────
1. Parse incoming HTTP request                  [RUNNING]
2. Call db.query("SELECT * FROM orders")
   └─ Sends bytes over TCP to database
   └─ Calls read() on socket, which BLOCKS      [BLOCKED]

OS Scheduler switches to Thread 3, or if none ready, idles

... time passes, network packets arrive ...

OS wakes Thread 1: data arrived on its socket   [RUNNABLE]
Thread 1 scheduled, continues:
3. Process query results                        [RUNNING]
4. Serialize HTTP response
5. Send response, thread returns to pool
```

### Breaking Down Each Phase

**Step 1 — Parsing the Request**

When the backend receives a request, the first thing it does is parse the HTTP body, query parameters, headers, etc. This is a **CPU operation** — it involves reading bytes from memory and deserializing them into a native data structure (an object in JavaScript, a dict in Python, a struct in Go).

**Step 2 — Making a Database Call**

After parsing and validation, the handler needs to fetch data. For example:

```sql
SELECT * FROM users
```

To execute this query, two things happen under the hood:
1. A TCP connection is established with the database (or retrieved from a connection pool), and the query is sent as bytes over the wire.
2. The thread then calls `read()` on the database socket — waiting for the response to come back. This `read()` call is a **blocking IO operation**. The CPU has nothing useful to do here — it is simply waiting for the network.

**Step 3 — OS Scheduler Takes Over**

The moment a thread blocks on IO, the OS scheduler detects this and **context switches** to another thread. This is expensive:
- The state of Thread 1 (its stack, registers, instruction pointer) must be saved.
- The state of Thread 2 must be loaded.
- Thread 2 starts executing from where it left off.

This happens for Thread 2 as well — it parses its request, fires off its own DB query (`SELECT * FROM orders`), and then blocks on `read()`. If there is a Thread 3 available, the scheduler switches to that. If not, everything waits.

**Step 4 — Resuming After IO Completes**

When the network packets arrive (the database response), the OS wakes the appropriate thread, marks it as `RUNNABLE`, and eventually schedules it back onto a CPU core. The thread then:
- Reads and parses the raw bytes from the database socket into a native data structure.
- Serializes that data structure into JSON (or whatever format is being used).
- Sends the HTTP response back to the client.
- Returns to the thread pool.

### What This Looks Like in Code (Python / Blocking Style)

```python
def handle_request(request):
    user_id = request.params["id"]
    # This line blocks the thread — no async/await, pure threading model
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    return json_response(user)
```

Even though this looks like sequential code, behind the scenes the thread is being paused and resumed around the `db.query()` call. The blocking is invisible to you as a developer reading the code — but the OS scheduler is doing a lot of work on your behalf.

---

## The Event Loop Model — Non-Blocking Concurrency

### How a Single Thread Handles Multiple Requests

The event loop model achieves concurrency **without multiple OS threads**. A single thread manages many requests by never blocking — instead of waiting for IO, it registers a callback and immediately moves on to the next task.

```
Single Thread running Event Loop:
─────────────────────────────────────────────────────────

1. Request A arrives
   └─ Parse HTTP request
   └─ Start async DB query (sends TCP, registers callback)
   └─ Returns immediately (NOT blocking!)

2. Request B arrives (microseconds later)
   └─ Parse HTTP request
   └─ Start async DB query (sends TCP, registers callback)
   └─ Returns immediately

3. Event loop checks: any I/O ready? No.
   Loop waits on epoll (efficient OS-level wait)

4. DB response for Request B arrives first
   └─ epoll returns, indicates socket B readable
   └─ Loop invokes Request B's callback
   └─ Callback processes results, sends HTTP response

5. DB response for Request A arrives
   └─ Loop invokes Request A's callback
   └─ Callback processes results, sends HTTP response

One thread, but both requests handled efficiently.
No blocking — yielding control between I/O operations.
```

### The Key Difference from Threading

In the threading model, pausing and resuming uses **OS-native data structures** — instruction pointers, stacks, CPU registers. This is expensive.

In the event loop model, pausing and resuming uses **callbacks** — lightweight function references registered against IO events. No context switching at the OS level. No stack allocation per task.

### How the Event Loop Actually Monitors IO

The event loop is literally an infinite loop. In each iteration it asks the operating system: *"Has any of the IO I'm waiting for completed?"*

It uses OS-level primitives for this:
- **Linux:** `epoll`
- **macOS:** `kqueue`
- **Windows:** `IOCP`

These are extremely efficient mechanisms for monitoring hundreds or thousands of sockets simultaneously. The moment any socket becomes readable (a DB response arrived, an API call returned), the loop is notified, and it invokes the relevant callback.

---

## Async Primitives — Callbacks, Promises, and async/await

### Why These Primitives Exist

Because of the constraint of the event loop architecture, any language that uses an event loop (JavaScript being the most prominent example) provides async primitives:
- **Callbacks**
- **Promises**
- **async/await**

These are all **syntactic sugar** — convenient wrappers that give you the right mental model for how your program will execute. Their job is to help you **yield control at IO-bound points** so you never accidentally block the event loop.

The moment you start a database query, you write `await`. The moment you make an API call, you write `await`. By writing `await`, you are saying:

> "I am giving up my CPU processing slice. When this network operation completes, give it back to me so I can finish this function."

### Callbacks (Pre-ES6 JavaScript)

This is what concurrency looked like in JavaScript before ES6:

```javascript
function handleRequest(req, sendResponse) {
    db.query("SELECT * FROM users WHERE id = ?", req.params.id, function(err, user) {
        if (err) return sendResponse(500, err);
        sendResponse(200, user);
    });
}
```

The third argument to `db.query` is a **callback** — a piece of logic registered with the event loop that will only run once the IO response arrives.

**The Problem — Callback Hell**

If you need to chain multiple async operations (a DB query, then an API call, then another DB query), you end up nesting callbacks inside callbacks:

```javascript
db.query("SELECT * FROM users WHERE id = ?", id, function(err, user) {
    db.query("SELECT * FROM orders WHERE user_id = ?", user.id, function(err, orders) {
        apiCall("/shipping", orders, function(err, shipping) {
            // ... and so on
        });
    });
});
```

This is **callback hell** — deeply nested, hard to read, hard to debug.

### async/await (Modern JavaScript / ES6+)

`async/await` is syntactic sugar over callbacks that eliminates nesting while preserving the same underlying event loop behavior:

```javascript
async function handleRequest(req) {
    const user = await db.query("SELECT * FROM users WHERE id = ?", req.params.id);
    return user;
}
```

The moment you write `await`, everything after that line (until the function ends) is conceptually placed into a callback that runs when the IO completes. You get linear, readable code — but the event loop works the same way underneath.

**Rules of async/await:**
- You can only use `await` inside an `async` function.
- Writing `await` yields control back to the event loop for the duration of the IO operation.
- The event loop is free to execute other tasks while waiting.

---

## How async/await Works Under the Hood — State Machines

### The Mental Model

When your programming language runtime sees an `async` function, it transforms it into a **state machine**. Each `await` keyword represents a state transition.

Consider this function:

```javascript
async function fetchUserData(userId) {
    const user = await db.getUser(userId);
    const orders = await db.getOrders(user.id);
    return { user, orders };
}
```

### The State Machine Equivalent

Here is the conceptual equivalent of what the runtime actually runs:

```javascript
function fetchUserData(userId) {
    let state = 0;
    let user, orders;

    function step() {
        switch (state) {
            case 0:
                state = 1;
                db.getUser(userId).then(result => {
                    user = result;
                    step(); // transition to state 1
                });
                break;
            case 1:
                state = 2;
                db.getOrders(user.id).then(result => {
                    orders = result;
                    step(); // transition to state 2
                });
                break;
            case 2:
                return { user, orders };
        }
    }

    return step();
}
```

### Execution Timeline

| Phase | State | What happens |
|---|---|---|
| **Execution 0** | `state = 0` | `db.getUser()` is called; callback registered; event loop is free |
| **Execution 1** | `state = 1` | DB responds with `user`; `db.getOrders()` called; event loop is free again |
| **Execution 2** | `state = 2` | DB responds with `orders`; result returned |

Between each execution phase, the event loop is completely free to run other tasks.

### Two Things This Explains

**1. Why you can only use `await` inside `async` functions**

Because the function has to be transformed into a state machine. If a function is not marked `async`, the runtime has no way to set up that transformation.

**2. Why blocking the event loop is so dangerous**

If you block the event loop (e.g., with a heavy synchronous computation), the state machine can never transition from state 0 to state 1. The entire program freezes. No other requests can be handled. This is why you must **never block the event loop** with CPU-intensive work in an event-loop-based architecture.

---

## Virtual Threads and Goroutines — The Go Model

### What is a Virtual Thread?

A **virtual thread** (or **goroutine** in Go) is a thread-like abstraction managed not by the OS, but by the **language runtime**. It gives you:
- The **readable, sequential code style** of threads (no callbacks, no `await`).
- The **efficiency of event loops** (the runtime — not the OS — handles scheduling).

Go does not call them virtual threads. It calls them **goroutines**. But conceptually they serve the same purpose — they are lightweight, cheap to create, and managed by the Go runtime scheduler rather than the OS.

### Go Creates a Goroutine Per Request

In Go's standard HTTP library, a new goroutine is created for **every incoming HTTP request**. This would be catastrophic with OS threads (each thread costs megabytes of memory), but with goroutines it is perfectly fine because goroutines are extremely cheap.

From Go's standard library source:

```go
// Inside the serve() function of Go's net/http package:
go c.serve(connCtx) // Creates a new goroutine for each connection
```

The `go` keyword spins up a new goroutine. It is that simple.

### What Actually Happens — The Go Runtime Scheduler

```
Lightweight Threads: What Actually Happens
─────────────────────────────────────────────────────────────────

Goroutine 1 (Request A):          Goroutine 2 (Request B):
──────────────────────            ──────────────────────
1. Parse request                  1. Parse request
2. db.Query(...)                  2. db.Query(...)
   └─ Runtime detects                └─ Runtime detects
      network I/O                       network I/O
   └─ Parks goroutine,              └─ Parks goroutine,
      continues others                  continues others

┌─────────────────────────────────────────────────────────────┐
│              Go Runtime Scheduler (not OS!)                  │
│                                                             │
│  M (OS threads):  [ M1 ]   [ M2 ]   [ M3 ]   [ M4 ]       │
│                     |        |        |        |            │
│  G (goroutines):  [G1]     [G3]     [G5]     [G7]          │
│                   [G2]     [G4]     [G6]     [G8]          │
│                    :        :        :        :             │
│                 (queue)  (queue)  (queue)  (queue)          │
│                                                             │
│  When G1 blocks on I/O, M1 picks up G2 from queue.         │
│  No OS context switch. Just swapping pointers.              │
└─────────────────────────────────────────────────────────────┘
```

### The Three Layers Explained

**Layer 1 — OS Threads (M)**

Go creates one OS thread per CPU core by default (controlled by `GOMAXPROCS`). With 4 CPU cores, you get M1, M2, M3, M4. These are real OS threads — expensive to create, expensive to switch.

**Layer 2 — Goroutines (G)**

For every request, every background task, every concurrent operation, Go creates a goroutine. Goroutines start with roughly 2–8 KB of stack space (compared to 1–8 MB for OS threads), and they can grow and shrink dynamically. You can have thousands or even millions of goroutines at once.

**Layer 3 — The Go Runtime Scheduler**

The Go runtime scheduler maps goroutines onto OS threads using a work-stealing algorithm. Each OS thread has a local queue of goroutines to run. When one goroutine blocks on IO, the OS thread does **not** block — it simply picks up the next goroutine from its queue. This is just a pointer swap. No OS context switch needed.

```
Code looks like blocking threads:
    user := db.Query("SELECT ...")  // blocks, but cheaply

But the runtime handles multiplexing, not the OS.
Thousands of goroutines on a handful of OS threads.
```

### What Go Handler Code Looks Like

```go
func handleGetUser(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("id")

    // This looks blocking — and it is, but only at the goroutine level.
    // The OS thread (M) is not blocked. The Go scheduler picks up another goroutine.
    user, err := db.Query("SELECT * FROM users WHERE id = $1", userID)
    if err != nil {
        http.Error(w, err.Error(), 500)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

The code reads like a straightforward sequential program. No callbacks. No `await`. But the Go runtime scheduler is quietly pausing and resuming goroutines around every IO operation, achieving the same efficiency as an event loop — without forcing you to write callback-style code.

---

## Race Conditions and Shared State Problems

### What is a Race Condition?

Like any concurrent system, concurrency comes with its own class of bugs. The root cause of almost all concurrency bugs is **shared state** — when two or more threads or goroutines try to read and modify the same piece of memory simultaneously.

### Example 1 — Lost Update (Threading)

Consider a simple counter variable starting at `0`. Two threads try to increment it simultaneously. To increment a value, a thread must perform three operations:

1. **READ** the current value from memory into a CPU register.
2. **ADD** 1 to the register.
3. **WRITE** the register value back to memory.

Now look at what happens when two threads interleave:

```
Race Condition: Lost Update
──────────────────────────────────────────────────────────────

Memory: counter = 0

Time    Thread A                    Thread B
────    ────────                    ────────
 t1     READ counter (gets 0)
 t2                                 READ counter (gets 0)
 t3     ADD 1 (register = 1)
 t4                                 ADD 1 (register = 1)
 t5     WRITE counter ← 1
 t6                                 WRITE counter ← 1

Memory: counter = 1  (should be 2!)

Both threads read 0, both compute 1, both write 1.
One increment was completely lost.
```

Both threads read `0`, both compute `1`, both write `1`. The final value is `1` instead of `2`. **One increment was silently lost.** This is called a **lost update** and is a classic example of a **race condition**.

### Example 2 — Race Condition in Async Code

You might think: *"I use JavaScript with a single thread and async/await — I'm safe from race conditions."* Unfortunately, that is not true.

```
Race Condition in Async Code
──────────────────────────────────────────────────────────────

balance = 100

withdraw(100) #1                    withdraw(100) #2
──────────────                      ──────────────

if (100 >= 100) ✓
await process...
|
| (yielded)
|                                   if (100 >= 100) ✓
|                                   await process...
|                                   |
|                                   | (yielded)
|
balance = 100 - 100
balance is now 0
                                    balance = 100 - 100
                                    (but 100 was the OLD value!)
                                    balance is now -100
```

**What happened:**

```javascript
let balance = 100;

async function withdraw(amount) {
    if (balance >= amount) {          // check passes for both calls
        await processWithdrawal();    // yields control here
        balance = balance - amount;   // by now, balance may already be 0
    }
}

withdraw(100);  // call #1
withdraw(100);  // call #2
```

1. `withdraw(100)` call #1 checks: `100 >= 100` → true. Enters the block.
2. Hits `await processWithdrawal()` → yields control to the event loop.
3. `withdraw(100)` call #2 starts executing. Checks: `100 >= 100` → still true (balance hasn't changed yet).
4. Also hits `await` → yields control.
5. Call #1 resumes. Sets `balance = 100 - 100 = 0`.
6. Call #2 resumes. Sets `balance = 0 - 100 = -100`. ❌

The check was valid when it happened, but the state changed before the write. This is a classic **time-of-check to time-of-use (TOCTOU)** race condition — and it happens in single-threaded async code because `await` creates yield points where other code can run.

---

## Solutions — Locks, Mutexes, and Channels

### 1. Locks / Mutexes

A **mutex** (mutual exclusion) is a primitive that ensures only one thread can execute a critical section of code at a time.

```python
import threading

lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:               # acquire the lock
        value = counter      # READ
        value += 1           # ADD
        counter = value      # WRITE
                             # lock is released automatically
```

- The moment Thread A acquires the lock and enters the block, all other threads that call `increment()` must **wait** until Thread A is done and releases the lock.
- This prevents interleaved reads and writes.
- The tradeoff is that it introduces **serialization** — threads can no longer truly run concurrently for this section of code.

### 2. Channels (Go)

In Go, the idiomatic solution to shared state problems is **channels** — a communication primitive that lets goroutines pass messages to each other rather than sharing a variable directly.

The philosophy in Go is:

> *"Don't communicate by sharing memory; share memory by communicating."*

Instead of having multiple goroutines all read and write to a shared `counter` variable, you designate a **single goroutine** as the owner of that state. All other goroutines send messages to it requesting updates. Only the owner goroutine reads and writes the variable, so there is no concurrent access.

```go
func counterManager(ch <-chan int) {
    count := 0
    for delta := range ch {
        count += delta
        fmt.Println("Counter:", count)
    }
}

func main() {
    ch := make(chan int)
    go counterManager(ch)
    ch <- 1  // goroutine 1 sends increment request
    ch <- 1  // goroutine 2 sends increment request
}
```

Only `counterManager` ever touches `count`. Race condition eliminated by design.

---

## Concurrency Models — A Side-by-Side Comparison

| | Threading | Event Loop | Goroutines (Go) |
|---|---|---|---|
| **Unit of concurrency** | OS Thread | Callbacks / async tasks | Goroutine (virtual thread) |
| **Managed by** | Operating System | Language runtime (single thread) | Go runtime scheduler |
| **Memory per unit** | 1–8 MB per thread | Shared (single thread) | ~2–8 KB per goroutine |
| **Context switching cost** | High (OS-level) | None (single thread) | Very low (pointer swap) |
| **Code style** | Sequential, blocking | Callback / async/await | Sequential, blocking |
| **Best for** | CPU-bound work | IO-bound, high concurrency | Both IO and CPU-bound |
| **Risk** | Race conditions, high memory | Callback hell, blocking the loop | Race conditions if channels misused |

---

## IO-Bound vs CPU-Bound — The Core Distinction

### IO-Bound Operations

An **IO-bound** operation is one where the CPU is idle — it is waiting for something external to complete. The CPU has no computation to perform; it is simply waiting for a response.

Examples:
- Reading a response from a database socket (`read()` on a TCP socket)
- Making an HTTP call to an external API
- Reading a file from disk
- Waiting for a message from a queue

In these cases, the CPU is not the bottleneck. The **network or storage** is. This is why event loops and goroutines shine here — you can handle thousands of concurrent IO operations without needing thousands of OS threads.

### CPU-Bound Operations

A **CPU-bound** operation is one where the processor is actively computing. There is no waiting — the CPU is at full utilization doing the work.

Examples:
- Image compression or resizing
- Video encoding / transcoding
- Cryptographic operations
- Number crunching (machine learning inference, simulations)
- Parsing large files into memory

In these cases, concurrency without parallelism does not help. You need **real parallelism** — multiple CPU cores doing work simultaneously. This is where OS threads with true parallel execution win.

---

## Summary and Key Takeaways

### IO-Bound Workloads → Prefer Async / Event Loop / Goroutines

For services that spend most of their time waiting for IO — web servers, API gateways, services that make many outbound API calls, services that talk to databases — the following primitives are significantly more efficient than a raw threading model:

- `async/await` (JavaScript, Python, Rust, C#)
- Goroutines (Go)
- Virtual Threads (modern Java)

These allow **high concurrency** — handling thousands of simultaneous connections — without the memory and switching overhead of thousands of OS threads.

### CPU-Bound Workloads → Prefer Threads with Parallelism

For services that do heavy computation — image processing, video encoding, ML inference — you need real CPU cores working in parallel. Threading with true parallelism is more effective than an event loop here, because an event loop is fundamentally single-threaded and cannot utilize multiple CPU cores simultaneously.

### Concurrency vs Parallelism

| Concept | Definition |
|---|---|
| **Concurrency** | Keeping the CPU productive by not wasting cycles waiting for IO. Multiple tasks are in progress, but not necessarily at the exact same instant. |
| **Parallelism** | Using multiple CPU cores to do multiple things at the exact same instant. |

Most backend work leans heavily on **concurrency**, not parallelism. But understanding the distinction is what lets you make the right architectural decision.

### Async Primitives Are Syntactic Sugar

All of these — callbacks, promises, async/await — are **syntactic sugar** on top of the same underlying mechanism: yielding control at IO-bound points so the event loop can do other work. `await` does not change what happens. It changes how readable your code is.

### Shared State Is the Root of All Concurrency Bugs

Almost every concurrency bug — race conditions, lost updates, stale reads — comes from multiple concurrent executors touching the same piece of memory. The solutions are:
- **Locks / Mutexes** — serialize access to the shared resource.
- **Channels (Go)** — eliminate sharing by having a single owner communicate via messages.
- **Immutability** — if state cannot be changed, it cannot be raced on.

> **The key insight:** Concurrency lets your program stay productive and not waste CPU cycles while waiting for IO. Parallelism lets you use multiple CPU cores to do multiple things at the exact same moment. Most backend systems need the former. Understanding both is what makes you an effective backend engineer.