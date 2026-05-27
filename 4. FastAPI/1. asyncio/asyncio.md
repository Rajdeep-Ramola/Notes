# Python `asyncio` — The Complete Guide

> *Based on a comprehensive video tutorial covering everything from core terminology to real-world performance optimization.*

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [What is Concurrency?](#2-what-is-concurrency)
3. [How asyncio Works — Key Characteristics](#3-how-asyncio-works--key-characteristics)
4. [Core Terminology](#4-core-terminology)
   - [The Event Loop](#41-the-event-loop)
   - [Awaitables](#42-awaitables)
   - [Coroutines](#43-coroutines)
   - [Tasks](#44-tasks)
   - [Futures](#45-futures)
5. [Code Examples — Step by Step](#5-code-examples--step-by-step)
   - [Example 1 — Synchronous Code (Baseline)](#51-example-1--synchronous-code-baseline)
   - [Example 2 — Common Mistake: Awaiting Coroutines Directly](#52-example-2--common-mistake-awaiting-coroutines-directly)
   - [Example 3 — The Correct Way: `create_task`](#53-example-3--the-correct-way-create_task)
   - [Example 4 — Understanding Await Order vs Execution Order](#54-example-4--understanding-await-order-vs-execution-order)
   - [Example 5 — Blocking the Event Loop with Synchronous Code](#55-example-5--blocking-the-event-loop-with-synchronous-code)
   - [Example 6 — Running Blocking Code in Threads and Processes](#56-example-6--running-blocking-code-in-threads-and-processes)
   - [Example 7 — gather, Task Lists, and Task Groups](#57-example-7--gather-task-lists-and-task-groups)
6. [Real-World Example — Image Downloader and Processor](#6-real-world-example--image-downloader-and-processor)
   - [The Synchronous Baseline](#61-the-synchronous-baseline)
   - [Profiling: Finding IO-bound vs CPU-bound Code](#62-profiling-finding-io-bound-vs-cpu-bound-code)
   - [Async v1 — Threads for Downloads, Threads for Processing](#63-async-v1--threads-for-downloads-threads-for-processing)
   - [Async v2 — httpx + aiofiles + ProcessPoolExecutor](#64-async-v2--httpx--aiofiles--processpoolexecutor)
   - [Async v3 — Adding Semaphores and CPU Worker Limits](#65-async-v3--adding-semaphores-and-cpu-worker-limits)
7. [gather vs Task Groups — Choosing the Right Tool](#7-gather-vs-task-groups--choosing-the-right-tool)
8. [Async Context Managers](#8-async-context-managers)
9. [Async Iterators](#9-async-iterators)
10. [Common Pitfalls](#10-common-pitfalls)
    - [Forgetting to await Tasks or Coroutines](#101-forgetting-to-await-tasks-or-coroutines)
    - [Script Ending Before Tasks Complete](#102-script-ending-before-tasks-complete)
    - [Blocking the Event Loop with Synchronous Code](#103-blocking-the-event-loop-with-synchronous-code)
11. [When to Use asyncio vs Threads vs Multiprocessing](#11-when-to-use-asyncio-vs-threads-vs-multiprocessing)
12. [The asyncio Ecosystem](#12-the-asyncio-ecosystem)
13. [Debugging Tips](#13-debugging-tips)
14. [Quick Reference Summary](#14-quick-reference-summary)

---

## 1. Introduction

`asyncio` is Python's built-in library for writing **concurrent code**. It can seem intimidating at first with all the different terminology and moving parts, but once you understand how it works under the hood, using it becomes intuitive.

This guide covers:

- How `asyncio` works under the hood
- All core terminology (event loop, coroutines, tasks, futures, awaitables)
- Step-by-step code examples with explanations of what the event loop is doing at each step
- How to determine *when* and *where* to use `asyncio` using profiling
- A real-world codebase converted from synchronous to fully optimized async
- When to choose `asyncio` vs threads vs multiprocessing

> **Note:** This guide uses the **modern, current way** to use `asyncio`. The library has evolved significantly over the years with different APIs for running the event loop, scheduling tasks, etc. Everything here reflects current best practice.

---

## 2. What is Concurrency?

Before diving into `asyncio`, it's important to understand what concurrency actually is, and how it differs from normal synchronous execution.

### Synchronous Execution

With **synchronous code** (what we normally write), one thing happens after another. A good analogy is a **Subway restaurant**: you place your order, and the employee makes your entire sandwich from start to finish before moving on to the next customer.

### Concurrent Execution

With **concurrent code**, multiple things can be in progress at the same time. Think of a **McDonald's**: someone takes your order and immediately moves on to the next customer, while your food is being prepared in the background.

### ⚠️ Critical Misconception: Async ≠ Faster

> **Asynchronous code does not automatically mean faster code.**

It simply means that instead of sitting idly waiting for something like a network request or database query to finish, your program can do other useful work in the meantime.

This is why `asyncio` excels at **IO-bound tasks** — tasks where your program is waiting on something external:

- Network requests
- Database queries
- File access
- Any waiting on external services

For tasks that require heavy **computation** (CPU-bound), you would want to use **multiprocessing** instead, because `asyncio` is single-threaded and runs on a single process.

---

## 3. How asyncio Works — Key Characteristics

- **Single-threaded** — runs on a single process
- Uses **cooperative multitasking** — tasks voluntarily give up control
- Excels at **IO-bound tasks** where time is spent waiting
- Does **not** speed up CPU-bound tasks (use multiprocessing for those)

---

## 4. Core Terminology

This is the section that trips most people up when first learning `asyncio`. Let's walk through each concept carefully.

### 4.1 The Event Loop

The **event loop** is the engine that runs and manages asynchronous functions. Think of it as a scheduler or controller. It:

- Keeps track of all tasks
- When a task is suspended (because it's waiting for something), control returns to the event loop
- The event loop then finds another task to either start or resume

You must have a running event loop for any asynchronous code to work. The modern way to start one is:

```python
asyncio.run(main())
```

`asyncio.run()` is responsible for:
1. Starting up the event loop
2. Running tasks until they're marked as complete
3. Closing down the event loop when done

### 4.2 Awaitables

You'll see the `await` keyword everywhere in asynchronous code. **Awaitables** are objects that implement a special `__await__` method under the hood.

**Why can't we `await` a synchronous function?**

Synchronous libraries don't have a mechanism to work with the event loop. They don't know how to yield control and resume later. For example:

- `time.sleep()` — **not** awaitable, blocks the event loop
- `asyncio.sleep()` — **awaitable**, properly suspends the coroutine and yields control

Also, you can only use the `await` keyword **inside** a function that has the `async` keyword. Trying to use `await` outside an `async` function will produce a syntax error/warning.

**What does `await` do?**

When you `await` something, you're telling the event loop:
> "Pause the execution of the current function and yield control back to the event loop, which can then run another task. Resume me when this awaitable completes."

### 4.3 Coroutines

In Python's `asyncio`, there are **three main types of awaitable objects**:

1. **Coroutines** — created when you call an `async` function
2. **Tasks** — wrappers around coroutines that are scheduled on the event loop
3. **Futures** — low-level objects representing eventual results

#### Coroutine Function vs Coroutine Object

There are two terms to understand here:

| Term | Definition | Example |
|---|---|---|
| **Coroutine function** | A function defined with `async def` | `async def fetch_data(param):` |
| **Coroutine object** | The awaitable returned when you *call* a coroutine function | `obj = fetch_data(1)` — `obj` is the coroutine object |

Coroutines are a bit like **generators** in the sense that they can suspend execution and resume later, but they're specifically designed to work with an event loop. They have extra features that `asyncio` needs to schedule them, await IO, and coordinate multiple tasks.

```python
async def async_function(test_param: str) -> str:
    print("This is an asynchronous coroutine function.")
    await asyncio.sleep(0.1)
    return f"Async Result: {test_param}"

# Calling the coroutine function returns a coroutine OBJECT — it doesn't run yet
coroutine_obj = async_function("Test")
print(coroutine_obj)  # <coroutine object async_function at 0x...>

# To actually run it, you must await it
coroutine_result = await coroutine_obj
print(coroutine_result)  # Async Result: Test
```

> **Important:** When you `await` a coroutine object directly (without wrapping it in a task first), it is both **scheduled on the event loop AND run to completion at the same time** — you get no concurrency benefit.

### 4.4 Tasks

**Tasks** are wrapped coroutines that can be executed independently. They are the primary mechanism for running coroutines **concurrently**.

When you wrap a coroutine in a task using `asyncio.create_task()`, it is handed over to the event loop and **scheduled to run** whenever the event loop gets a chance. The task:

- Tracks whether the coroutine finished successfully
- Tracks if it raised an error
- Tracks if it was cancelled

Unlike coroutine objects, tasks can sit in the event loop's queue waiting to run while other tasks execute. **This is the key to asyncio concurrency** — you can queue up multiple tasks at once, and the event loop runs them concurrently while they wait on IO.

```python
task = asyncio.create_task(async_function("Test"))
print(task)  # <Task pending name='Task-1' coro=<async_function() running at ...>>

task_result = await task
print(task_result)  # Async Result: Test
```

> **Tasks are Futures under the hood**, but with extra logic to actually run the coroutine and do the work. That's why we work with tasks rather than futures in most code.

### 4.5 Futures

**Futures** are low-level objects representing an eventual result. If you're coming from the JavaScript world, futures are a lot like **Promises** — a promise of a result that will be available later.

A future's state can be:
- `PENDING` — the future doesn't have any result or exception yet
- `CANCELLED` — it was cancelled via `future.cancel()`
- `FINISHED` — either a result was set via `future.set_result()` or an exception via `future.set_exception()`

```python
loop = asyncio.get_running_loop()
future = loop.create_future()
print(f"Empty Future: {future}")  # <Future pending>

future.set_result("Future Result: Test")
future_result = await future
print(future_result)  # Future Result: Test
```

> **You almost never work with futures directly in application code.** You only need them if you're writing low-level async IO code, such as building an async-compatible framework. `asyncio` uses futures under the hood when scheduling tasks, but you won't interact with them directly.

Here is the full terminology reference file from the tutorial:

```python
# terms.py
import asyncio
import time


def sync_function(test_param: str) -> str:
    print("This is a synchronous function.")
    time.sleep(0.1)
    return f"Sync Result: {test_param}"


# ALSO KNOWN AS A COROUTINE FUNCTION
async def async_function(test_param: str) -> str:
    print("This is an asynchronous coroutine function.")
    await asyncio.sleep(0.1)
    return f"Async Result: {test_param}"


async def main():
    sync_result = sync_function("Test")
    print(sync_result)

    # --- Futures ---
    # loop = asyncio.get_running_loop()
    # future = loop.create_future()
    # print(f"Empty Future: {future}")
    # future.set_result("Future Result: Test")
    # future_result = await future
    # print(future_result)

    # --- Coroutine Object ---
    # coroutine_obj = async_function("Test")
    # print(coroutine_obj)
    # coroutine_result = await coroutine_obj
    # print(coroutine_result)

    # --- Task ---
    # task = asyncio.create_task(async_function("Test"))
    # print(task)
    # task_result = await task
    # print(task_result)


if __name__ == "__main__":
    asyncio.run(main())
```

---

## 5. Code Examples — Step by Step

Now let's walk through progressively more complex examples to really understand how `asyncio` behaves.

---

### 5.1 Example 1 — Synchronous Code (Baseline)

This is plain synchronous code with no `asyncio` at all. It establishes our baseline before we start introducing async patterns.

```python
# example_1.py
import time


def fetch_data(param):
    print(f"Do something with {param}...")
    time.sleep(param)
    print(f"Done with {param}")
    return f"Result of {param}"


def main():
    result1 = fetch_data(1)
    print("Fetch 1 fully completed")
    result2 = fetch_data(2)
    print("Fetch 2 fully completed")
    return [result1, result2]


t1 = time.perf_counter()

results = main()
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**What happens:**

1. `fetch_data(1)` runs — prints, sleeps for 1 second, prints, returns
2. `fetch_data(2)` runs — prints, sleeps for 2 seconds, prints, returns
3. Total time: **3 seconds** (1 + 2)

**Key observation:** During both sleeps, the program is just sitting idle doing nothing. This is wasted time that concurrent code could use to do other work.

**Execution flow:**

```
main()
  └─ fetch_data(1) → sleeps 1s (BLOCKED) → done
  └─ fetch_data(2) → sleeps 2s (BLOCKED) → done
Total: 3 seconds
```

---

### 5.2 Example 2 — Common Mistake: Awaiting Coroutines Directly

This is what someone's first attempt at converting to `asyncio` often looks like — and it contains a very common mistake.

```python
# example_2.py
import asyncio
import time


async def fetch_data(param):
    print(f"Do something with {param}...")
    await asyncio.sleep(param)
    print(f"Done with {param}")
    return f"Result of {param}"


async def main():
    task1 = fetch_data(1)  # Could be awaited directly — just a coroutine OBJECT
    task2 = fetch_data(2)  # Could be awaited directly — just a coroutine OBJECT
    result1 = await task1
    print("Task 1 fully completed")
    result2 = await task2
    print("Task 2 fully completed")
    return [result1, result2]


t1 = time.perf_counter()

results = asyncio.run(main())
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**Result: still 3 seconds — no concurrency benefit!**

**Why?**

The common misconception is that calling a coroutine function like `fetch_data(1)` creates a task and schedules it on the event loop. **It does not.** It just creates a **coroutine object**.

When you `await` that coroutine object, you're scheduling it AND running it to completion **at the same time**. You never have two things running concurrently — only one task exists on the event loop at any point.

**Event loop visualization:**

```
Event Loop
  └─ main (running)
       ├─ task1 = fetch_data(1)   → just a coroutine object, nothing scheduled
       ├─ task2 = fetch_data(2)   → just a coroutine object, nothing scheduled
       ├─ await task1             → schedules AND runs to completion (1s)
       └─ await task2             → schedules AND runs to completion (2s)
Total: 3 seconds (no concurrency)
```

---

### 5.3 Example 3 — The Correct Way: `create_task`

The **only change** from Example 2 is that we now use `asyncio.create_task()` to wrap our coroutines before awaiting them.

```python
# example_3.py
import asyncio
import time


async def fetch_data(param):
    print(f"Do something with {param}...")
    await asyncio.sleep(param)
    print(f"Done with {param}")
    return f"Result of {param}"


async def main():
    task1 = asyncio.create_task(fetch_data(1))  # Scheduled on the event loop immediately!
    task2 = asyncio.create_task(fetch_data(2))  # Scheduled on the event loop immediately!
    result1 = await task1
    print("Task 1 fully completed")
    result2 = await task2
    print("Task 2 fully completed")
    return [result1, result2]


t1 = time.perf_counter()

results = asyncio.run(main())
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**Result: 2 seconds — both tasks ran concurrently!**

**Output:**
```
Do something with 1...
Do something with 2...
Done with 1
Task 1 fully completed
Done with 2
Task 2 fully completed
['Result of 1', 'Result of 2']
Finished in 2.00 seconds
```

Notice how "Do something with 1..." and "Do something with 2..." print back-to-back, before either one finishes. That's concurrency in action.

**Event loop visualization:**

```
Event Loop
  └─ main (running)
       ├─ create_task(fetch_data(1)) → Task 1 SCHEDULED ✓
       ├─ create_task(fetch_data(2)) → Task 2 SCHEDULED ✓
       ├─ await task1
       │    └─ main SUSPENDS
       │    └─ event loop finds Task 1 (ready) → runs until await asyncio.sleep(1) → SUSPENDS Task 1
       │    └─ event loop finds Task 2 (ready) → runs until await asyncio.sleep(2) → SUSPENDS Task 2
       │    └─ [Both timers running concurrently in background]
       │    └─ Timer 1 completes → Task 1 WAKES UP → finishes → main WAKES UP
       ├─ print "Task 1 fully completed"
       ├─ await task2 → Task 2 already suspended, just wait for timer
       │    └─ Timer 2 completes → Task 2 WAKES UP → finishes → main WAKES UP
       └─ print "Task 2 fully completed"
Total: 2 seconds (longest task duration)
```

**The key insight:** `asyncio.create_task()` is what schedules the coroutine onto the event loop. Without it, the coroutine is just an object sitting in memory — not queued for execution.

---

### 5.4 Example 4 — Understanding Await Order vs Execution Order

This example teaches an important but subtle concept: **what you `await` first doesn't control what runs first** — it only controls what you wait for before moving on.

```python
# example_4.py
import asyncio
import time


async def fetch_data(param):
    print(f"Do something with {param}...")
    await asyncio.sleep(param)
    print(f"Done with {param}")
    return f"Result of {param}"


async def main():
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    result2 = await task2   # ← awaiting task2 FIRST this time
    print("Task 2 fully completed")
    result1 = await task1   # ← awaiting task1 second
    print("Task 1 fully completed")
    return [result1, result2]


t1 = time.perf_counter()

results = asyncio.run(main())
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**You might expect:** Task 2 runs first, or Task 2 completes first in the output.

**What actually happens:**

```
Do something with 1...
Do something with 2...
Done with 1         ← task1 still finishes first (shorter sleep)
Done with 2
Task 2 fully completed
Task 1 fully completed
['Result of 1', 'Result of 2']
Finished in 2.00 seconds
```

**Why?**

The event loop uses a **FIFO (First In, First Out) queue** for ready tasks. Both tasks were scheduled in the same order (task1 then task2), so the event loop runs them in that order. Awaiting task2 first does **not** make task2 run first.

**What `await` *does* guarantee:** You won't move past the `await` line until that specific thing is done.

- `await task2` → "I won't move past this line until task2 is complete"
- task1 is already done (saved in memory) → `await task1` just retrieves the result

> **Practical takeaway:** Don't try to reason about exactly which task runs at which moment — let the event loop handle that. What you *do* control is: "don't move forward until this is done." The event loop runs whatever is ready.

---

### 5.5 Example 5 — Blocking the Event Loop with Synchronous Code

This is a critically important example that shows what happens when you accidentally use blocking synchronous code inside an async function.

```python
# example_5.py
import asyncio
import time


async def fetch_data(param):
    print(f"Do something with {param}...")
    time.sleep(param)          # ← BLOCKING CALL — not awaitable, can't yield control
    print(f"Done with {param}")
    return f"Result of {param}"


async def main():
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    result1 = await task1
    print("Task 1 fully completed")
    result2 = await task2
    print("Task 2 fully completed")
    return [result1, result2]


t1 = time.perf_counter()

results = asyncio.run(main())
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**Result: 3 seconds — no concurrency!**

**Output:**
```
Do something with 1...
Done with 1            ← task1 finishes before task2 even starts
Task 1 fully completed
Do something with 2...
Done with 2
Task 2 fully completed
Finished in 3.00 seconds
```

**Why does `time.sleep` break concurrency?**

`time.sleep` is not awaitable. It was never coded to know how to:
- Pause its own execution
- Yield control back to the event loop
- Resume later

When the event loop runs `fetch_data(1)` and hits `time.sleep(1)`, the task **never suspends itself**. It just sits there blocking the entire thread — and since `asyncio` is single-threaded, the event loop is completely frozen for that entire second. Task 2 never gets a chance to run.

**Event loop visualization (blocked):**

```
Event Loop
  └─ main (running)
       ├─ create_task(fetch_data(1)) → Task 1 SCHEDULED
       ├─ create_task(fetch_data(2)) → Task 2 SCHEDULED
       ├─ await task1 → main SUSPENDS
       │    └─ event loop runs Task 1
       │         └─ time.sleep(1) → EVENT LOOP IS FROZEN for 1 second (no suspension!)
       │         └─ Task 1 finishes → main WAKES UP
       ├─ await task2 → main SUSPENDS
       │    └─ event loop runs Task 2
       │         └─ time.sleep(2) → EVENT LOOP IS FROZEN for 2 seconds
       │         └─ Task 2 finishes → main WAKES UP
Total: 3 seconds
```

> **This could be ANY blocking code** — not just `time.sleep`. It could be `requests.get()`, any other synchronous IO library call, or any compute-heavy operation. If it doesn't `await`, it blocks.

---

### 5.6 Example 6 — Running Blocking Code in Threads and Processes

Sometimes you need to use synchronous blocking code that has no async alternative. `asyncio` provides a way to run it in threads or processes, and the event loop will manage those for you.

```python
# example_6.py
import asyncio
import time
from concurrent.futures import ProcessPoolExecutor


def fetch_data(param):
    # Note: regular synchronous function, NOT async
    print(f"Do something with {param}...", flush=True)
    time.sleep(param)
    print(f"Done with {param}", flush=True)
    return f"Result of {param}"


async def main():
    # --- Run in Threads ---
    task1 = asyncio.create_task(asyncio.to_thread(fetch_data, 1))
    task2 = asyncio.create_task(asyncio.to_thread(fetch_data, 2))
    result1 = await task1
    print("Thread 1 fully completed")
    result2 = await task2
    print("Thread 2 fully completed")

    # --- Run in Process Pool ---
    loop = asyncio.get_running_loop()

    with ProcessPoolExecutor() as executor:
        task1 = loop.run_in_executor(executor, fetch_data, 1)
        task2 = loop.run_in_executor(executor, fetch_data, 2)

        result1 = await task1
        print("Process 1 fully completed")
        result2 = await task2
        print("Process 2 fully completed")

    return [result1, result2]


if __name__ == "__main__":
    t1 = time.perf_counter()

    results = asyncio.run(main())
    print(results)

    t2 = time.perf_counter()
    print(f"Finished in {t2 - t1:.2f} seconds")
```

**Result: ~4 seconds** (two groups of 2 seconds each — one for threads, one for processes, each group running concurrently)

#### Running in Threads with `asyncio.to_thread()`

```python
asyncio.create_task(asyncio.to_thread(fetch_data, 1))
```

- `asyncio.to_thread()` wraps a synchronous function in a future and makes it awaitable
- Pass the **function itself** (not called) and its arguments **separately** — `asyncio.to_thread` will call it when ready
- The event loop kicks off the thread and suspends the task, freeing the event loop to run other work
- **Do not do:** `asyncio.to_thread(fetch_data(1))` ← this calls the function immediately before passing it

#### Running in Processes with `ProcessPoolExecutor`

```python
loop = asyncio.get_running_loop()

with ProcessPoolExecutor() as executor:
    task1 = loop.run_in_executor(executor, fetch_data, 1)
    task2 = loop.run_in_executor(executor, fetch_data, 2)
    result1 = await task1
    result2 = await task2
```

- Need `asyncio.get_running_loop()` because we're using `loop.run_in_executor()`
- `ProcessPoolExecutor` from `concurrent.futures` manages the process pool
- Pass the executor, function, and arguments separately — same pattern as `to_thread`
- Each spawned process re-runs the script, so the `if __name__ == "__main__":` guard is **required** to prevent infinite process spawning

> **Note:** The `flush=True` argument in print statements is used here because when running code outside the current thread, print output can get buffered and appear in unexpected order. `flush=True` forces immediate output.

---

### 5.7 Example 7 — `gather`, Task Lists, and Task Groups

So far we've been creating and awaiting tasks one at a time manually. There are better ways to run many tasks at once.

```python
# example_7.py
import asyncio
import time


async def fetch_data(param):
    await asyncio.sleep(param)
    return f"Result of {param}"


async def main():
    # --- Method 1: Create Tasks Manually ---
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    result1 = await task1
    result2 = await task2
    print(f"Task 1 and 2 awaited results: {[result1, result2]}")

    # --- Method 2: Gather Coroutines ---
    coroutines = [fetch_data(i) for i in range(1, 3)]
    results = await asyncio.gather(*coroutines, return_exceptions=True)
    print(f"Coroutine Results: {results}")

    # --- Method 3: Gather Tasks ---
    tasks = [asyncio.create_task(fetch_data(i)) for i in range(1, 3)]
    results = await asyncio.gather(*tasks)
    print(f"Task Results: {results}")

    # --- Method 4: Task Group ---
    async with asyncio.TaskGroup() as tg:
        results = [tg.create_task(fetch_data(i)) for i in range(1, 3)]
        # All tasks are awaited automatically when the context manager exits
    print(f"Task Group Results: {[result.result() for result in results]}")

    return "Main Coroutine Done"


t1 = time.perf_counter()

results = asyncio.run(main())
print(results)

t2 = time.perf_counter()
print(f"Finished in {t2 - t1:.2f} seconds")
```

**Result: ~8 seconds** (4 groups × 2 seconds each, each group running concurrently)

#### Method 2: `asyncio.gather()` with Coroutines

```python
coroutines = [fetch_data(i) for i in range(1, 3)]
results = await asyncio.gather(*coroutines, return_exceptions=True)
```

- The `*` (asterisk) **unpacks** the list — `gather` doesn't accept a list directly, it takes individual awaitables as positional arguments
- When passing coroutine objects to `gather`, they are scheduled and run to completion — concurrently
- `return_exceptions=True` is **strongly recommended** (see the gather vs task groups section below)

#### Method 3: `asyncio.gather()` with Tasks

```python
tasks = [asyncio.create_task(fetch_data(i)) for i in range(1, 3)]
results = await asyncio.gather(*tasks)
```

- Tasks are **already scheduled** when created — they're in the event loop queue immediately
- Use tasks over raw coroutines if you need to **monitor or interact** with them before they complete (cancel, check status, etc.)
- If you just want the results, passing coroutines directly is fine

#### Method 4: `asyncio.TaskGroup`

```python
async with asyncio.TaskGroup() as tg:
    results = [tg.create_task(fetch_data(i)) for i in range(1, 3)]
# Tasks are automatically awaited here when the context manager exits
print([result.result() for result in results])
```

- Uses an **async context manager** (`async with`)
- You don't `await` anything explicitly — the task group awaits all its tasks automatically on exit
- To get results after the block, call `.result()` on each task object

---

## 6. Real-World Example — Image Downloader and Processor

Now let's look at how you'd take a real synchronous codebase and convert it to async step by step.

**The script downloads 12 high-definition images from Unsplash and then runs an edge-detection algorithm on each one.**

---

### 6.1 The Synchronous Baseline

```python
# real_world_example_sync_v1.py
import time
from pathlib import Path

import requests
from PIL import Image

IMAGE_URLS = [
    "https://images.unsplash.com/photo-1516117172878-fd2c41f4a759?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1532009324734-20a7a5813719?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1524429656589-6633a470097c?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1530224264768-7ff8c1789d79?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1564135624576-c5c88640f235?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1541698444083-023c97d3f4b6?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1522364723953-452d3431c267?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1493976040374-85c8e12f0c0e?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1530122037265-a5f1f91d3b99?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1516972810927-80185027ca84?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1550439062-609e1531270e?w=1920&h=1080&fit=crop",
    "https://images.unsplash.com/photo-1549692520-acc6669e2f0c?w=1920&h=1080&fit=crop",
]

ORIGINAL_DIR = Path("original_images")
PROCESSED_DIR = Path("processed_images")


def download_single_image(session: requests.Session, url: str, img_num: int) -> Path:
    print(f"Downloading {url}...")
    ts = int(time.time())
    url = f"{url}?ts={ts}"  # Add timestamp to avoid caching issues
    response = session.get(url, timeout=10, allow_redirects=True)
    response.raise_for_status()

    filename = f"image_{img_num}.jpg"
    download_path = ORIGINAL_DIR / filename

    with download_path.open("wb") as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)

    print(f"Downloaded and saved to: {download_path}")
    return download_path


def download_images(urls: list) -> list[Path]:
    with requests.Session() as session:
        img_paths = [
            download_single_image(session, url, img_num)
            for img_num, url in enumerate(urls, start=1)
        ]
    return img_paths


def process_single_image(orig_path: Path) -> Path:
    save_path = PROCESSED_DIR / orig_path.name

    with Image.open(orig_path) as img:
        data = list(img.getdata())
        width, height = img.size
        new_data = []

        for i in range(len(data)):
            current_r, current_g, current_b = data[i]
            total_diff = 0
            neighbor_count = 0

            for dx, dy in [(1, 0), (0, 1)]:
                x = (i % width) + dx
                y = (i // width) + dy

                if 0 <= x < width and 0 <= y < height:
                    neighbor_r, neighbor_g, neighbor_b = data[y * width + x]
                    diff = (
                        abs(current_r - neighbor_r)
                        + abs(current_g - neighbor_g)
                        + abs(current_b - neighbor_b)
                    )
                    total_diff += diff
                    neighbor_count += 1

            if neighbor_count > 0:
                edge_strength = total_diff // neighbor_count
                if edge_strength > 30:
                    new_data.append((255, 255, 255))
                else:
                    new_data.append((0, 0, 0))
            else:
                new_data.append((0, 0, 0))

        edge_img = Image.new("RGB", (width, height))
        edge_img.putdata(new_data)
        edge_img.save(save_path)

    print(f"Processed {orig_path} and saved to {save_path}")
    return save_path


def process_images(orig_paths: list[Path]) -> list[Path]:
    img_paths = [process_single_image(orig_path) for orig_path in orig_paths]
    return img_paths


def main():
    ORIGINAL_DIR.mkdir(parents=True, exist_ok=True)
    PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

    start_time = time.perf_counter()
    img_paths = download_images(IMAGE_URLS)
    proc_start_time = time.perf_counter()
    processed_paths = process_images(img_paths)
    finished_time = time.perf_counter()

    dl_total_time = proc_start_time - start_time
    proc_total_time = finished_time - proc_start_time
    total_time = finished_time - start_time

    print(f"\nDownloaded {len(img_paths)} images in: {dl_total_time:.2f} seconds. "
          f"{(dl_total_time / total_time) * 100:.2f}% of total time")
    print(f"Processed {len(processed_paths)} images in: {proc_total_time:.2f} seconds. "
          f"{(proc_total_time / total_time) * 100:.2f}% of total time")
    print(f"\nTotal execution time: {total_time:.2f} seconds.")


if __name__ == "__main__":
    main()
```

**Baseline performance:**
| Phase | Time | % of Total |
|---|---|---|
| Download 12 images | ~13 seconds | ~55% |
| Process 12 images | ~10 seconds | ~44% |
| **Total** | **~23 seconds** | **100%** |

---

### 6.2 Profiling: Finding IO-bound vs CPU-bound Code

Before optimizing, you need to know *where* time is being spent. Guessing wrong leads to wasted effort.

#### Tips for Identifying IO-bound vs CPU-bound

**Vocabulary clues in function names:**

| Words that suggest IO-bound | Words that suggest CPU-bound |
|---|---|
| fetch, get, download, request | compute, calculate, process |
| query, read, write, connect | transform, analyze, render |
| send, receive, stream | encode, decode, compress |

**But for certainty, use a profiler.**

#### Using Scalene Profiler

[Scalene](https://github.com/plasma-umass/scalene) is a high-performance CPU, GPU, and memory profiler for Python that requires no changes to your code.

**Install:**
```bash
pip install scalene
# or with uv:
uv add scalene
```

**Run with profiling:**
```bash
# Using uv:
uv run -m scalene --html --output-file profile_report.html real_world_example_sync_v1.py

# Or with python:
python -m scalene --html --output-file profile_report.html real_world_example_sync_v1.py
```

**Reading the report:**

The report breaks time into:
- **Python time** — time spent running Python code (likely CPU-bound work)
- **Native time** — time in compiled C extensions
- **System time** — time waiting on system calls, usually IO

**Findings from our script:**
- `download_single_image` → most time is **system time** → IO-bound → good candidate for `asyncio` or threads
- `process_single_image` → most time is **Python time** → CPU-bound → will benefit from **multiprocessing**, not async

---

### 6.3 Async v1 — Threads for Downloads, Threads for Processing

The first version uses `asyncio` to manage threads for both downloading and processing. We do this to demonstrate that threads **won't** speed up CPU-bound work, even if they speed up IO-bound work.

```python
# real_world_example_async_v1.py
import asyncio
import time
from pathlib import Path

import requests
from PIL import Image

IMAGE_URLS = [ ... ]  # same URLs as before

ORIGINAL_DIR = Path("original_images")
PROCESSED_DIR = Path("processed_images")


def download_single_image(url: str, img_num: int) -> Path:
    # Plain synchronous function — no async
    print(f"Downloading {url}...")
    ts = int(time.time())
    url = f"{url}?ts={ts}"
    response = requests.get(url, timeout=10, allow_redirects=True)  # requests.get instead of session
    response.raise_for_status()

    filename = f"image_{img_num}.jpg"
    download_path = ORIGINAL_DIR / filename

    with download_path.open("wb") as f:
        for chunk in response.iter_content(chunk_size=8192):
            f.write(chunk)

    print(f"Downloaded and saved to: {download_path}")
    return download_path


async def download_images(urls: list) -> list[Path]:
    async with asyncio.TaskGroup() as tg:
        tasks = [
            tg.create_task(asyncio.to_thread(download_single_image, url, img_num))
            for img_num, url in enumerate(urls, start=1)
        ]

    img_paths = [task.result() for task in tasks]
    return img_paths


def process_single_image(orig_path: Path) -> Path:
    # ... same edge detection algorithm (sync) ...
    pass


async def process_images(orig_paths: list[Path]) -> list[Path]:
    async with asyncio.TaskGroup() as tg:
        tasks = [
            tg.create_task(asyncio.to_thread(process_single_image, orig_path))
            for orig_path in orig_paths
        ]

    img_paths = [task.result() for task in tasks]
    return img_paths


async def main():
    ORIGINAL_DIR.mkdir(parents=True, exist_ok=True)
    PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

    start_time = time.perf_counter()
    img_paths = await download_images(IMAGE_URLS)

    proc_start_time = time.perf_counter()
    processed_paths = await process_images(img_paths)

    finished_time = time.perf_counter()
    # ... print timing ...


if __name__ == "__main__":
    asyncio.run(main())
```

**Key changes made:**

- `download_single_image` stays a **regular sync function** (not async) — we'll delegate it to threads
- Removed the `requests.Session` (thread-safety concerns) and used `requests.get` directly
- `download_images` and `process_images` are now **coroutines** (`async def`)
- Inside each, we use a `TaskGroup` to launch threads with `asyncio.to_thread()`
- `main()` is now `async` and uses `await` on these coroutines
- Entry point uses `asyncio.run(main())`

**v1 Performance Results:**
| Phase | Sync | Threads | Speedup |
|---|---|---|---|
| Download 12 images | 13s | **~2s** | **6.5×** |
| Process 12 images | 10.5s | 10.6s | **No change** |
| **Total** | **23s** | **~12s** | **~2×** |

**Conclusion:** Threads dramatically sped up IO-bound downloading but did **nothing** for CPU-bound processing. This confirms our profiling results.

---

### 6.4 Async v2 — httpx + aiofiles + ProcessPoolExecutor

Now we go further:

- Replace `requests` with `httpx` (native async HTTP client) — no more threads for downloads
- Replace synchronous file writes with `aiofiles`
- Replace threads for processing with `ProcessPoolExecutor` (multiprocessing) for true CPU parallelism

**Install dependencies:**
```bash
pip install httpx aiofiles
# or with uv:
uv add httpx aiofiles
```

```python
# real_world_example_async_v2.py
import asyncio
import time
from concurrent.futures import ProcessPoolExecutor
from pathlib import Path

import aiofiles
import httpx
from PIL import Image

IMAGE_URLS = [ ... ]  # same URLs

ORIGINAL_DIR = Path("original_images")
PROCESSED_DIR = Path("processed_images")


async def download_single_image(
    client: httpx.AsyncClient,
    url: str,
    img_num: int,
) -> Path:
    print(f"Downloading {url}...")
    ts = int(time.time())
    url = f"{url}?ts={ts}"

    response = await client.get(url, timeout=10, follow_redirects=True)  # ← await, async client
    response.raise_for_status()

    filename = f"image_{img_num}.jpg"
    download_path = ORIGINAL_DIR / filename

    async with aiofiles.open(download_path, "wb") as f:         # ← async file open
        async for chunk in response.aiter_bytes(chunk_size=8192):  # ← async iterator
            await f.write(chunk)                                 # ← await write

    print(f"Downloaded and saved to: {download_path}")
    return download_path


async def download_images(urls: list) -> list[Path]:
    async with httpx.AsyncClient() as client:       # ← async context manager for client
        async with asyncio.TaskGroup() as tg:
            tasks = [
                tg.create_task(download_single_image(client, url, img_num))
                for img_num, url in enumerate(urls, start=1)
            ]

        img_paths = [task.result() for task in tasks]

    return img_paths


def process_single_image(orig_path: Path) -> Path:
    # ... same sync edge detection algorithm ...
    pass


async def process_images(orig_paths: list[Path]) -> list[Path]:
    loop = asyncio.get_running_loop()

    with ProcessPoolExecutor() as executor:
        tasks = [
            loop.run_in_executor(executor, process_single_image, orig_path)
            for orig_path in orig_paths
        ]

        processed_paths = await asyncio.gather(*tasks)

    return processed_paths


async def main():
    ORIGINAL_DIR.mkdir(parents=True, exist_ok=True)
    PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

    start_time = time.perf_counter()
    img_paths = await download_images(IMAGE_URLS)

    proc_start_time = time.perf_counter()
    processed_paths = await process_images(img_paths)

    finished_time = time.perf_counter()
    # ... timing output ...


if __name__ == "__main__":
    asyncio.run(main())
```

**Key changes from v1:**

| Change | Reason |
|---|---|
| `requests` → `httpx.AsyncClient` | Native async HTTP, no threads needed for downloads |
| `session.get(...)` → `await client.get(...)` | Non-blocking network request |
| `allow_redirects` → `follow_redirects` | httpx parameter name difference |
| `open(...)` → `aiofiles.open(...)` | Non-blocking async file writes |
| `for chunk` → `async for chunk` | Async iterator for streaming response body |
| `f.write(chunk)` → `await f.write(chunk)` | Async write |
| Threads for processing → `ProcessPoolExecutor` | True CPU parallelism for CPU-bound work |
| `asyncio.gather(*tasks)` in `process_images` | Collect all process futures |

**v2 Performance Results:**
| Phase | Sync | v2 (async + multiprocessing) | Speedup |
|---|---|---|---|
| Download 12 images | 13s | **~1.6s** | **8×** |
| Process 12 images | 10.5s | **~3.25s** | **~3×** |
| **Total** | **~23s** | **~5s** | **~4.7×** |

This is a massive improvement across the board.

---

### 6.5 Async v3 — Adding Semaphores and CPU Worker Limits

The v2 script blasts off all 12 downloads and all processing tasks simultaneously. With 12 images this is manageable, but with thousands of URLs you could:

- Overwhelm your own machine
- Hammer external servers (potentially getting rate-limited or banned)

**Solution:** Use a **Semaphore** to limit concurrent downloads, and limit the `ProcessPoolExecutor` to the number of CPU cores.

```python
# real_world_example_async_v3.py
import asyncio
import os
import time
from concurrent.futures import ProcessPoolExecutor
from pathlib import Path

import aiofiles
import httpx
from PIL import Image

DOWNLOAD_LIMIT = 4          # Max concurrent downloads
CPU_WORKERS = os.cpu_count()  # Max processes = number of CPU cores

IMAGE_URLS = [ ... ]  # same URLs

ORIGINAL_DIR = Path("original_images")
PROCESSED_DIR = Path("processed_images")


async def download_single_image(
    client: httpx.AsyncClient,
    url: str,
    img_num: int,
    semaphore: asyncio.Semaphore,  # ← Accept semaphore as argument
) -> Path:
    async with semaphore:          # ← Only DOWNLOAD_LIMIT tasks can be in here at once
        print(f"Downloading {url}...")
        ts = int(time.time())
        url = f"{url}?ts={ts}"

        response = await client.get(url, timeout=10, follow_redirects=True)
        response.raise_for_status()

        filename = f"image_{img_num}.jpg"
        download_path = ORIGINAL_DIR / filename

        async with aiofiles.open(download_path, "wb") as f:
            async for chunk in response.aiter_bytes(chunk_size=8192):
                await f.write(chunk)

        print(f"Downloaded and saved to: {download_path}")
        return download_path


async def download_images(urls: list) -> list[Path]:
    dl_semaphore = asyncio.Semaphore(DOWNLOAD_LIMIT)  # ← Create semaphore
    async with httpx.AsyncClient() as client:
        async with asyncio.TaskGroup() as tg:
            tasks = [
                tg.create_task(
                    download_single_image(client, url, img_num, dl_semaphore)
                )
                for img_num, url in enumerate(urls, start=1)
            ]

        img_paths = [task.result() for task in tasks]

    return img_paths


def process_single_image(orig_path: Path) -> Path:
    # ... same sync edge detection ...
    pass


async def process_images(orig_paths: list[Path]) -> list[Path]:
    loop = asyncio.get_running_loop()

    with ProcessPoolExecutor(max_workers=CPU_WORKERS) as executor:  # ← Limit workers
        tasks = [
            loop.run_in_executor(executor, process_single_image, orig_path)
            for orig_path in orig_paths
        ]

        processed_paths = await asyncio.gather(*tasks, return_exceptions=True)

    return processed_paths


async def main():
    ORIGINAL_DIR.mkdir(parents=True, exist_ok=True)
    PROCESSED_DIR.mkdir(parents=True, exist_ok=True)

    start_time = time.perf_counter()
    img_paths = await download_images(IMAGE_URLS)

    proc_start_time = time.perf_counter()
    processed_paths = await process_images(img_paths)

    finished_time = time.perf_counter()
    # ... timing output ...


if __name__ == "__main__":
    asyncio.run(main())
```

#### How Semaphores Work

`asyncio.Semaphore(n)` allows at most `n` coroutines to be inside the `async with semaphore:` block at the same time. The rest will wait at the entrance until one of the active ones exits.

```python
dl_semaphore = asyncio.Semaphore(4)

async with semaphore:
    # At most 4 coroutines can be here simultaneously
    # The 5th one waits at this line until one of the 4 exits
    response = await client.get(...)
```

**What you see in output with the semaphore:**
```
Downloading URL1...
Downloading URL2...
Downloading URL3...
Downloading URL4...
# ← Only 4 at a time! Next 4 start when one of these finishes
Downloaded and saved to: image_1.jpg
Downloading URL5...
...
```

#### Limiting ProcessPoolExecutor Workers

```python
ProcessPoolExecutor(max_workers=CPU_WORKERS)
```

`CPU_WORKERS = os.cpu_count()` ensures we don't spin up more processes than there are physical CPU cores — spawning more processes than cores doesn't improve performance and adds overhead.

**v3 Performance (with limits):**
- Slightly slower than v2 (due to limits) but still dramatically faster than synchronous
- Much more considerate of server resources and your own machine
- Scales safely to thousands of URLs

---

## 7. `gather` vs Task Groups — Choosing the Right Tool

Both `asyncio.gather()` and `asyncio.TaskGroup` run multiple coroutines concurrently, but they handle **errors differently**.

### `asyncio.gather()` — Two Modes

#### Mode 1: `return_exceptions=False` (default — not recommended)

```python
results = await asyncio.gather(task1, task2, task3)
```

- If any task fails, **raises the first exception** immediately
- Other tasks are **not cancelled** — they keep running as orphaned tasks
- You get none of the successful results
- **Avoid this mode** — orphaned tasks are a resource leak and hard to debug

#### Mode 2: `return_exceptions=True` (recommended when using gather)

```python
results = await asyncio.gather(task1, task2, task3, return_exceptions=True)
```

- **All tasks run to completion**, regardless of failures
- Result is a list where each position is either:
  - The return value (if the task succeeded)
  - The exception object (if it failed)
- **Use this when** you want all tasks to run even if some fail

**Best use case:** Crawling many URLs where you don't want one bad URL to stop all the others.

### `asyncio.TaskGroup` — Fail Together or Succeed Together

```python
async with asyncio.TaskGroup() as tg:
    task1 = tg.create_task(fetch_data(1))
    task2 = tg.create_task(fetch_data(2))
# Automatically awaits all tasks on exit
```

- If **any task fails**, all other tasks in the group are **cancelled**
- Raises an `ExceptionGroup` containing all exceptions (including from cancelled tasks)
- No option to keep running after a failure
- Better error reporting and cleanup than `gather`
- **Use this when** all tasks must succeed together — if one fails, the whole operation is invalid

### Summary Table

| Feature | `gather(return_exceptions=False)` | `gather(return_exceptions=True)` | `TaskGroup` |
|---|---|---|---|
| If one task fails | Raises exception immediately | All tasks still run | Cancels all tasks |
| Other tasks cancelled on failure | No (orphaned!) | No | Yes |
| Error reporting | First exception only | Each position is result or exception | ExceptionGroup with all errors |
| Get partial results | No | Yes | No |
| Recommended? | ❌ Rarely | ✅ Yes, when partial results needed | ✅ Yes, when all-or-nothing needed |

---

## 8. Async Context Managers

You've seen `async with` used for both `httpx.AsyncClient` and `asyncio.TaskGroup`. This is an **async context manager**.

Just like regular context managers can run code on setup and teardown, async context managers can do **async IO operations** during setup or teardown — things like opening network connections, acquiring database connections, or coordinating tasks.

```python
# Regular context manager
with open("file.txt") as f:
    data = f.read()

# Async context manager
async with aiofiles.open("file.txt") as f:
    data = await f.read()
```

**Why async with for httpx.AsyncClient?**

```python
async with httpx.AsyncClient() as client:
    # Client is set up with connection pooling
    response = await client.get(url)
    # ...
# Client connection pool is properly torn down here
```

The async client reuses the same connection pool across all requests — much more efficient than creating a new connection for every request. The `async with` ensures the pool is properly closed when we're done.

**Why async with for TaskGroup?**

```python
async with asyncio.TaskGroup() as tg:
    tasks = [tg.create_task(fetch_data(i)) for i in range(10)]
# TaskGroup automatically awaits all tasks and handles cleanup here
```

The task group does a lot of async work (waiting for task completions, handling cancellations) when exiting — which is why it needs to be an async context manager.

---

## 9. Async Iterators

In `real_world_example_async_v2.py`, we introduced an async iterator:

```python
async for chunk in response.aiter_bytes(chunk_size=8192):
    await f.write(chunk)
```

**Why `async for` instead of regular `for`?**

With a regular `for` loop, Python immediately pulls the next value from the iterator. But when streaming a network response, getting the next chunk of data involves **waiting for IO** — more data has to arrive from the server.

`response.aiter_bytes()` returns an **async iterator** where each step does an `await` internally to get the next chunk. Using `async for` tells Python: "For each iteration, await the next chunk before proceeding."

```python
# Synchronous (blocks while waiting for chunk):
for chunk in response.iter_content(chunk_size=8192):
    f.write(chunk)

# Asynchronous (yields control while waiting for chunk):
async for chunk in response.aiter_bytes(chunk_size=8192):
    await f.write(chunk)
```

This is why the whole enclosing function must also be `async def` — you can only use `async for` inside an `async` function.

---

## 10. Common Pitfalls

### 10.1 Forgetting to await Tasks or Coroutines

```python
async def main():
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    # result1 = await task1   ← FORGOT TO AWAIT
    # result2 = await task2   ← FORGOT TO AWAIT
    return []
```

**What happens:**

- You might not get any errors at all — Python will just note that the tasks were cancelled
- The tasks never actually run to completion
- Depending on whether you have return statements, the output may look superficially normal
- On closer inspection, you'll see the tasks were cancelled before finishing

**Fix:** Always `await` every task or coroutine you create.

### 10.2 Script Ending Before Tasks Complete

```python
async def main():
    task1 = asyncio.create_task(fetch_data(1))
    task2 = asyncio.create_task(fetch_data(2))
    result1 = await task1
    await asyncio.sleep(0.1)   # ← Only waiting 0.1s, but task2 needs 2s!
    # task2 is silently abandoned
```

**What happens:**

- `task1` completes fine
- `task2` starts running but the script moves on after only 0.1 seconds
- `task2` is abandoned mid-execution — no error, no result

**Fix:** Ensure every task is properly awaited before the `asyncio.run()` call returns.

### 10.3 Blocking the Event Loop with Synchronous Code

We covered this in depth in Example 5, but to summarize:

**Any of these block the event loop:**

```python
# ❌ Blocks
time.sleep(1)

# ❌ Blocks
requests.get(url)

# ❌ Blocks
open("file.txt").read()

# ❌ Blocks — any heavy computation
for i in range(10_000_000):
    result += i
```

**Async alternatives:**

```python
# ✅ Yields control
await asyncio.sleep(1)

# ✅ Yields control
await client.get(url)   # httpx or aiohttp

# ✅ Yields control
async with aiofiles.open("file.txt") as f:
    data = await f.read()

# ✅ Offload to process
loop.run_in_executor(executor, heavy_compute_function, args)
```

**How to avoid accidentally blocking:**

- Use a good linter (e.g., **Ruff** with async rules enabled) — it catches many common async mistakes
- Enable debug mode during development:

```python
asyncio.run(main(), debug=True)
```

This surfaces issues like coroutines that weren't awaited, tasks that took too long, and blocked event loop warnings. Remember to **disable debug mode in production**.

---

## 11. When to Use asyncio vs Threads vs Multiprocessing

Here is the decision tree:

```
Is the work IO-bound?
(waiting on network, database, file system)
│
├── YES → Is there an async-compatible library available?
│          │
│          ├── YES → Use asyncio with the async library
│          │         (httpx, aiofiles, asyncpg, SQLAlchemy async, etc.)
│          │
│          └── NO  → Use asyncio.to_thread() to run sync code in threads
│                    (asyncio manages the threads for you)
│
└── NO → Is the work CPU-bound?
         (heavy computation, image processing, data crunching)
         │
         └── YES → Use multiprocessing (ProcessPoolExecutor)
                   Asyncio can manage these via loop.run_in_executor()
```

### Quick Reference

| Scenario | Tool | Why |
|---|---|---|
| Web requests (async library available) | `asyncio` + `httpx`/`aiohttp` | Non-blocking IO on current thread |
| Web requests (must use `requests`) | `asyncio.to_thread()` | Offload blocking call to thread |
| File IO | `asyncio` + `aiofiles` | Non-blocking file operations |
| Database queries | `asyncio` + async driver (asyncpg, SQLAlchemy async) | Non-blocking DB calls |
| Image/video processing | `ProcessPoolExecutor` | True CPU parallelism across cores |
| Data analysis, computation | `ProcessPoolExecutor` | True CPU parallelism across cores |
| Mixed IO + CPU | `asyncio` for IO + `ProcessPoolExecutor` for CPU | Best of both worlds |

### How to Tell if Code is IO-bound or CPU-bound

If you're unsure, **profile it** with Scalene:

- High **system time** in a function → IO-bound → async or threads
- High **Python time** in a function → CPU-bound → multiprocessing

---

## 12. The asyncio Ecosystem

The async ecosystem in Python has grown substantially. Key libraries:

| Category | Library |
|---|---|
| **HTTP clients** | `httpx`, `aiohttp` |
| **Web frameworks** | `FastAPI`, `Starlette`, `Sanic` |
| **File IO** | `aiofiles` |
| **Databases (PostgreSQL)** | `asyncpg`, `SQLAlchemy (async)` |
| **Databases (MySQL)** | `aiomysql` |
| **Task queues** | `Celery` (with async support), `arq` |
| **Testing** | `pytest-asyncio` |

As more libraries adopt `asyncio`, writing fully non-blocking applications becomes easier. The pattern established in this guide — use async libraries for IO, multiprocessing for CPU — applies across all of these.

---

## 13. Debugging Tips

### Enable Debug Mode

```python
asyncio.run(main(), debug=True)
```

Debug mode enables:
- Warnings for coroutines that are created but never awaited
- Warnings for tasks that take too long (potentially blocking the event loop)
- Detailed tracebacks for async code

**Always disable in production** — it adds overhead.

### Use a Linter

A linter like **Ruff** (with async rules enabled) will catch many common async mistakes:
- Using `await` outside `async` functions
- Missing `await` on coroutines/tasks
- Using blocking calls inside async functions

### Common Error Messages

| Message | Likely Cause |
|---|---|
| `RuntimeWarning: coroutine 'X' was never awaited` | You called an async function but forgot to `await` the result |
| `SyntaxError: 'await' outside async function` | `await` used in a regular (non-`async def`) function |
| `RuntimeError: This event loop is already running` | Trying to call `asyncio.run()` from inside a running event loop |
| Tasks showing as `cancelled` unexpectedly | Script ended before tasks finished — ensure all tasks are awaited |

---

## 14. Quick Reference Summary

### Key Concepts

| Term | Definition |
|---|---|
| **Event Loop** | The scheduler that runs and manages all async tasks |
| **Coroutine function** | A function defined with `async def` |
| **Coroutine object** | What you get when you *call* a coroutine function (before awaiting) |
| **Task** | A coroutine wrapped with `asyncio.create_task()`, scheduled on the event loop |
| **Future** | Low-level promise object; tasks are built on top of futures |
| **Awaitable** | Any object with `__await__`; coroutines, tasks, and futures are all awaitables |
| **Semaphore** | Limits concurrent access to a resource to at most N coroutines at once |

### Essential Patterns

```python
# Start the event loop
asyncio.run(main())

# Schedule a task (concurrent execution)
task = asyncio.create_task(some_coroutine())

# Await a single task
result = await task

# Await multiple tasks — all must succeed
async with asyncio.TaskGroup() as tg:
    tasks = [tg.create_task(coro(i)) for i in range(10)]
results = [t.result() for t in tasks]

# Await multiple tasks — continue even if some fail
results = await asyncio.gather(*coroutines, return_exceptions=True)

# Run sync code in a thread (non-blocking)
result = await asyncio.to_thread(sync_function, arg1, arg2)

# Run sync code in a process (non-blocking, true parallelism)
loop = asyncio.get_running_loop()
with ProcessPoolExecutor() as executor:
    result = await loop.run_in_executor(executor, sync_function, arg)

# Limit concurrency with semaphore
semaphore = asyncio.Semaphore(4)
async with semaphore:
    result = await some_limited_resource()

# Async context manager
async with httpx.AsyncClient() as client:
    response = await client.get(url)

# Async iterator
async for chunk in response.aiter_bytes():
    await file.write(chunk)

# Debug mode
asyncio.run(main(), debug=True)
```

### The Golden Rules

1. **Never use blocking calls inside async functions** — use async alternatives or `to_thread()`
2. **Use `asyncio.create_task()` to get concurrency** — awaiting coroutines directly gives you no benefit
3. **What you `await` first ≠ what runs first** — `await` only guarantees you won't move on until it's done
4. **`asyncio` for IO-bound, multiprocessing for CPU-bound**
5. **Profile before optimizing** — don't guess whether something is IO or CPU bound
6. **Add limits with semaphores** when running many concurrent operations
7. **Prefer `asyncio.gather(..., return_exceptions=True)` or `TaskGroup`** over the default gather behavior