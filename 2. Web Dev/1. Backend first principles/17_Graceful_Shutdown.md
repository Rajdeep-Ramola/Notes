# Graceful Shutdown

## Overview

Picture a very realistic scenario: you are in the middle of processing a critical payment transaction and suddenly your server needs to restart for a deployment. Someone pushed something to the production environment and the server needs to redeploy itself.

We do have techniques like **zero-downtime deployment** that ensure the old server does not go down before the new server — the one with the updated code — comes up and is ready to receive traffic. Those mechanisms exist. But at some point, when the new server is ready to go online and receive traffic, the old server has to shut down. It has to stop receiving traffic and the transition happens to the new server.

During that critical moment, what if you are in the middle of a transaction — say, buying something from Amazon or Flipkart? What happens to that payment? Does it get lost in the digital world? Does the customer get charged twice because of some kind of race condition?

These are the scenarios you have to think about as a backend engineer. This is not a new problem — it has existed since the start of servers and backends. And of course, we already have a solution: **Graceful Shutdown**.

Graceful shutdown is exactly as the name sounds. We want to stop our server gracefully, not abruptly, not suddenly. To oversimplify it, we want to teach our backend good manners. It cannot just slam the door when it is time to leave. Instead, your application politely finishes whatever it is doing, finishes its ongoing conversations, says goodbye to all the guests, cleans up after itself, and then finally closes the door.

When you have guests over and it is time to sleep, you do not just push them out and slam the door in their face. You perform some steps. Your backend should do the same.

Caring about graceful shutdown gives your application a very good user experience and avoids critical issues like data corruption. If you are in the middle of a payment transaction, you can avoid problems like double-charging the customer, losing the transaction, or having to process unnecessary refunds.

---

## Process Life Cycle Management

Your backend is an application that runs as a **process** inside some server, some computer. Everything that runs in an operating system runs as a process. And like all living things, every process has a life cycle:

- When it starts and how it starts
- When it ends and how it ends

Processes are born when they start, they live while they are executing, and they die when they are terminated. This whole thing is called the **life cycle of a process**.

To understand and implement graceful shutdown, you must first understand this process life cycle because the two are very closely connected.

### How the OS Communicates with a Process

When your operating system decides it is time for your application to stop running, it does not just pull the plug or kill the process outright. It follows an established protocol of communication — a way to say, "it is time for you to stop," and the application responds with, "give me a few seconds, then I will stop myself."

Of course, this conversation does not happen through text. Programs do not understand text. This communication between your application and your operating system happens through a concept called **signals**.

---

## Signals

Signals are an important concept in Unix-based operating systems. When we say Unix, we mean all Linux distributions — Arch Linux, Ubuntu, and so on — and also macOS, which originated from a Unix kernel. When we talk about servers, we almost always mean Linux, because 99% of the time when you deploy an application on a cloud provider, it runs on a Linux-based operating system.

Unix operating systems use signals for **IPC — Inter-Process Communication**. This is a technique through which two processes can communicate with each other using an established protocol.

Here is how it works: your application runs inside a process and it **registers handlers**. Handlers are pieces of code that run behind the scenes and wait for specific signals from the operating system. Through these handlers, your application essentially tells the operating system: "When you want me to stop, send me this specific signal, and I will handle it appropriately using predefined steps."

There are three major signals to understand:

---

### 1. SIGTERM — Terminate

**SIGTERM** stands for Signal + Terminate. It is the **polite way** for the operating system to ask your application to shut down. Think of it like someone coming up behind you and gently tapping your shoulder — "Hey, excuse me, could you please finish up and leave?" It is a very gentle nudge, a gentle request.

Because of this, your application has an opportunity to complete whatever it is already doing. It has a window of time — typically a few seconds — to finish up before it must exit.

**What does SIGTERM ask the application to do?**

1. **Finish existing requests** — Your backend is likely processing multiple HTTP requests concurrently. When it receives SIGTERM, it must finish those before stopping.
2. **Clean up resources** — Release file handles, close database connections, close network connections, and free up any system resources that were acquired during execution.
3. **Exit** — Terminate cleanly after steps 1 and 2 are complete.

**Who uses SIGTERM?**

SIGTERM is mostly used by deployment systems, process managers, and orchestration platforms like:

- **Kubernetes**
- **systemd**
- **PM2** (a popular Node.js process manager)

These tools use SIGTERM to let your application finish whatever it is doing, clean up, and leave gracefully.

---

### 2. SIGINT — Interrupt

**SIGINT** stands for Signal + Interrupt. If you have worked with any command-line or terminal-based applications, you have almost certainly already used this signal — it is triggered by pressing **Ctrl+C** on your keyboard.

When you are running a backend process locally and want to stop it, you press Ctrl+C and the process stops. Because it requires a human to press a keyboard shortcut, SIGINT is also called a **user-initiated shutdown** and is mostly used in development environments.

Process-to-process communication between programs normally does not use SIGINT since it requires a physical key press.

**How should SIGINT be handled?**

In pretty much all cases, you want to handle SIGINT the exact same way you handle SIGTERM. It does not matter whether your backend is running locally and you are stopping it with Ctrl+C, or it is running in an AWS EC2 instance and a process manager like PM2 is sending a SIGTERM signal.

What matters is the **intention** — we want to shut down. And we want to shut down in a clean, graceful way. Whether the shutdown is initiated by a human or by a program, the steps should be the same.

---

### 3. SIGKILL — Kill

**SIGKILL** stands for Signal + Kill, and it is exactly what it sounds like — it instantly kills the application.

The critically important thing about SIGKILL is that it **cannot be caught and cannot be ignored**. Your application cannot register a handler to detect it or perform any kind of cleanup when it arrives. If your application is sent a SIGKILL signal, it has no choice but to stop at that exact moment. Nothing else happens.

Think of it as a **nuclear option**. The difference between SIGTERM and SIGKILL is like the difference between clicking your system's shutdown button versus walking over to the power plug and yanking it out. Your computer just dies. That is exactly how SIGKILL works.

**Why this matters for graceful shutdown:**

If you do not respect the polite signals — SIGTERM and SIGINT — which give you time to finish your work and clean up, eventually you will receive a SIGKILL. And at that point, you do not even get the opportunity to clean up after yourself. This is exactly why implementing graceful shutdown properly is so important.

---

## The Two Key Steps of Graceful Shutdown

Now that we understand signals, let us go deeper into the two important things that happen during a graceful shutdown:

1. Finishing existing (in-flight) requests — **Connection Draining**
2. Cleaning up resources — **Resource Cleanup**

---

## Connection Draining

### What are In-Flight Requests?

Your HTTP backend is capable of processing multiple requests concurrently at the same time. When it is time to stop your server, it is very likely that your backend is already processing a number of those requests — it could be 10, 12, hundreds, or thousands depending on the scale of your system. These are called **in-flight requests** — the requests already being processed by your server at that particular moment.

### The Restaurant Analogy

Imagine you have gone to a restaurant with friends and for some reason — it is closing time, say 10:30 or 11 at night — the restaurant has to close. The restaurant owners cannot just come up, turn off all the lights, and throw everyone out. Instead, here is what happens:

1. **Stop letting new customers in.** Someone at the reception is asked to stop allowing new people to enter the restaurant. You do not want new customers you then have to turn away.
2. **Announce to existing customers** that it is time to close. "You have 15 to 20 minutes — finish your meal, take your time, pay your bills, pay your tips, and then please leave."

This is exactly the same process for your backend, and we call it **connection draining**.

### How Connection Draining Works

When your application receives a shutdown signal (SIGTERM from a process manager, or SIGINT from a Ctrl+C), here is what it does:

**Step 1 — Stop accepting new connections.** Just as you stop letting new customers into the restaurant, the very first thing your application must do is stop accepting new HTTP requests from any client. This prevents the situation from becoming more complex and messy.

**Step 2 — Let existing requests finish.** Allow the in-flight requests — the ones already being processed — to complete as soon as possible.

**Step 3 — Close the connection.**

### Connection Draining by Application Type

The technical implementation of connection draining differs depending on the architecture of the application:

- **HTTP backends** — Stop accepting new HTTP requests from clients and allow all in-flight requests to finish and return a response.
- **Database applications** — Finish all existing queries or transactions. Stop accepting new queries or transactions before closing the connection.
- **WebSocket-based connections** — First notify the clients that the connection is closing, then close the socket. You cannot close the socket abruptly.

The high-level idea is the same regardless of the architecture: stop accepting new connections, finish the existing ones, then close.

### The Timeout Challenge

The major challenge with connection draining is **timing**. You want to give existing connections enough time to complete their work, but you cannot wait indefinitely. There has to be a limit.

Most production backend systems implement a **timeout mechanism** — commonly around **30 seconds**, though it varies. This is the maximum duration the system will wait. After that, it forces a stop.

Most of the time, 30 seconds is more than enough, because if you have stopped accepting new requests, the existing ones should complete quickly. But if some blocking operation cannot finish within this window, the server will be forcefully stopped. The timeout is the hard limit — you have exactly this amount of time to finish.

### Choosing the Right Timeout

Choosing the right timeout is an interesting design decision:

- **Too short** — You risk interrupting legitimate, real operations.
- **Too long** — The shutdown process becomes sluggish, which impacts your deployment speed and system responsiveness.

The right timeout depends on your **application's typical request duration** and your **operational requirements**. For a traditional HTTP backend, 30 seconds is usually more than enough. For WebSocket-heavy or more complex architectures, you need to understand your system and decide accordingly.

### Coordination with Load Balancers and Service Discovery

Connection draining also requires coordination with your **load balancers** and **service discovery systems** — including your health check systems and the process of registering and deregistering with service discovery.

Service discovery is the mechanism that controls how your deployed services — your backend, your database, your Elasticsearch instance — know about each other and communicate with each other after deployment. During a graceful shutdown, your instance also needs to deregister itself from service discovery so it stops receiving new traffic.

---

## Resource Cleanup

### What is Resource Cleanup?

Think about your desk. When it is time to leave the house or go to sleep, you do some kind of cleanup — you take your coffee cup to the sink, manage your cables, and so on. In the context of your backend application, **resource cleanup** means the same thing.

Resources include things like:

- **File handles** — When your application accesses a location in your file system, the operating system gives it a handle to that file. You must release this handle when you are done with it. If you do not, your application will keep consuming more and more RAM until you run out of it.
- **Network connections** — Your operating system limits the number of network connections a particular process can have open simultaneously. If you do not release network connections after using them, you will eventually run out of memory or face severe performance issues.
- **Database connections** — Before your backend shuts down, all database transactions your backend was handling must either be **committed** or **rolled back** explicitly by your application. If you do not do this, those transactions may enter an inconsistent state, leading to deadlocks, data corruption, and a variety of other serious issues.
- **Temporary files** — Files created during execution that are no longer needed.
- **Caches** — Any in-memory or external cache state that was being managed.

### The Order of Cleanup Matters

An important rule to follow when cleaning up resources: **clean up in the reverse order of acquisition**.

If during startup your application first established a Redis connection, then a database connection, then started an HTTP server — during shutdown you should close them in the reverse order: HTTP server first, then the database connection, then the Redis connection.

The reason for this is to **prevent situations where you clean up a resource that another resource still depends on**. Reversing the order of cleanup ensures that dependencies are respected and that nothing is left in an inconsistent state.

---

## Putting It All Together — A Practical Example

Let us look at a realistic Go-based backend as an example to tie everything together. You do not need to understand the code — just follow the narrative.

### Registering Signal Handlers

The first thing the application does on startup is **register handlers** that wait for signals from the operating system. When an interrupt signal is received — whether SIGTERM or SIGINT — it calls a dedicated `graceful shutdown` function.

### Inside the Graceful Shutdown Function

The shutdown function performs the following steps in order:

**Step 1 — Shut down the HTTP server.** Most HTTP frameworks and libraries provide a built-in method for this. When called, the library internally stops receiving new connections and waits for all existing in-flight requests to finish. The most common naming for this is something like `server.Shutdown()`.

**Step 2 — Close the database connection.** The database layer stops accepting new queries and new transactions. It finishes the ones already in progress. Then it releases all the TCP connections it was holding open to the database.

To understand this: your backend connects to your database over **TCP connections**. When you use database connection pooling, you have a pool of these active TCP connections. When shutting down, the database layer first stops accepting new queries through these connections, finishes existing transactions, and then closes each connection in the pool one by one.

**Step 3 — Close background job processing connections.** If your application uses a background job processing library — for example, one that uses Redis as its queue backend — you call the method that shuts down the worker, waits for all running jobs to finish, and then closes the Redis connection.

### What the Logs Tell You

When this is working correctly, you can observe the full lifecycle in the application's logs:

**Startup logs:**
- Connected to the database
- Started the background job server
- Server started and ready to receive requests

**Shutdown logs (triggered by Ctrl+C / SIGTERM):**
- Received interrupt signal
- Closed database connection
- Stopping background job processing server
- Waiting for all workers to finish
- All workers have finished
- Server has exited properly

This whole workflow — startup, running, and the graceful stopping phase — is called the **process life cycle**. The stopping phase specifically is the **graceful shutdown procedure**.

---

## Why This Matters in Production

Implementing graceful shutdown properly is especially important during deployments. Without it, you risk:

- **Corrupting in-flight requests** — A payment that was halfway through processing gets abandoned
- **Data corruption** — Database transactions left in an inconsistent state, leading to deadlocks
- **Double charging** — A payment is triggered twice due to a race condition
- **Poor user experience** — Customers see errors or lost orders during deployment windows

With graceful shutdown:

- In-flight requests are completed before the server stops
- Database transactions are properly committed or rolled back
- All system resources are released cleanly
- Deployments are seamless from the user's perspective

---

## Summary

Graceful shutdown is the practice of stopping your backend application in a controlled, step-by-step manner rather than abruptly pulling the plug.

- Every application runs as a **process** in the operating system, and every process has a **life cycle**
- The OS communicates with your process using **signals** — SIGTERM, SIGINT, and SIGKILL
- **SIGTERM** is the polite shutdown request sent by deployment tools and process managers like Kubernetes, systemd, and PM2
- **SIGINT** is the user-initiated shutdown triggered by Ctrl+C, mostly used in development
- **SIGKILL** is the nuclear option — it cannot be caught or ignored and kills the process instantly with no cleanup
- If you do not respond to the polite signals, you will eventually receive a SIGKILL
- The two key steps during a graceful shutdown are **connection draining** (stop accepting new requests, finish existing ones) and **resource cleanup** (release file handles, database connections, network connections, and temporary files in the reverse order of acquisition)
- **Timeouts** set a hard limit on how long the shutdown procedure can wait — commonly 30 seconds for most HTTP backends
- **Always clean up resources in the reverse order of acquisition** to avoid dependency issues
- Most frameworks provide built-in utilities for graceful shutdown — understanding what happens under the hood is what makes you a better backend engineer