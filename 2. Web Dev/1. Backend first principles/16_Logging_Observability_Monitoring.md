# Logging, Monitoring, and Observability

## Overview

Logging, monitoring, and observability is a huge topic — and each portion is deserving enough to get its own dedicated discussion. But this is also one of those topics where **not everything is pre-decided**. There are no hard rules. It is more of a spectrum.

These are **practices**, not a fixed checklist. Most companies, startups, and individuals implement logging, monitoring, and observability across a spectrum. We can never say that a product or company follows *all* the good logging, monitoring, and observability practices — there is no such thing. So the goal here is to get familiar enough with the different keywords, practices, products, and tools used across the industry without feeling intimidated.

---

## Why Do We Need This?

These days, in the modern internet, all our backend applications and full-stack applications run in a **distributed environment**. They run on different servers in different regions, and our users are spread across the whole world.

Given these scenarios, we need practices, tools, and methodologies so that we can **keep track of what is happening** in all our services, infrastructure components, and everything in between.

Keeping track essentially means monitoring a set of important parameters — things like:

- Is our application healthy right now?
- How many requests is the server handling per second?
- What is the state of our database connections?
- Did any requests fail, and if so, why?

---

## The Three Core Practices

### 1. Logging

Logging is basically **recording**. It is the practice of recording all the important events, all the suspicious events, and all the security-related events that happen across the entire lifecycle of requests and application execution.

Every important event that happens in your backend application should be recorded along with **metadata**. Metadata in this context means things like:

- The user ID that triggered the request
- The latency of the request
- Which method or function was triggered
- Any other contextual information that will help when you need to understand your system

Think of logs as a **journal or diary** that your backend application maintains — so that when the time comes, you will know what happened, when it happened, and exactly why it happened.

---

### 2. Monitoring

Monitoring is exactly what it sounds like — we want some way of **continuously keeping track of the state** of our applications and all the different components of our backend infrastructure.

This includes:

- The server's CPU usage
- The server's memory usage
- How many requests the server is processing per second at any given time
- The state of our database connections (e.g., how many connections are open when using connection pooling)

Monitoring basically means having **near-real-time data** about our system. Not exactly real-time — there is usually a delay of around 10 to 15 seconds. This is intentional. We do not want to overwhelm our logging, monitoring, and observability systems by pushing data every millisecond. So when we say "real-time" in this context, we mean approximately 10 to 15 seconds of delay.

---

### 3. Observability

Observability is a broader practice that includes several other components. If you go by theoretical terms, observability has **three pillars**. We call them pillars because a backend system can only be considered observable if all three components are in place.

Those three pillars are:

1. **Logs**
2. **Metrics**
3. **Traces**

We can call a system **observable** if we can determine the internal state of the application by looking at its external outputs and different parameters.

---

## The Three Pillars of Observability

### Pillar 1 — Logs

Logs are a record of all the important events happening across the request lifecycle — from application startup to application shutdown and everything in between.

We already covered this in the logging section above.

### Pillar 2 — Metrics

Metrics are closely related to monitoring. They are **concrete numbers** — either real-time values or historical values — that quantify the state of your system.

Examples of metrics:

- How many requests have been processed in the last hour?
- How many requests failed? (Any request returning a status code greater than 200 can be considered a failed request)
- If your application handles to-do lists, how many to-dos were created and how many failed?
- What is the average response time of your APIs?
- What is the current error rate?

Metrics can be configured in advance. When setting up your logging, monitoring, and observability system — both in your code and in your tooling — you define which metrics you care about.

### Pillar 3 — Traces

Traces are essentially **transactions**. A trace tracks the full journey of a request — from where it originated (a frontend system, a load balancer, or your backend application) to every single component it touched during execution.

For example, if you have:

```
Handler Layer → Service Layer → Validation Layer → Repository Layer → Database Layer
```

Using traces, you can track when a request started, which component it entered, and which component it was in when it failed. A trace is basically a transaction that includes all the different components involved in processing a single request.

---

## How Logging, Monitoring, and Observability Work Together

Each of the three pillars answers a different question when you are debugging your system:

| Pillar | Answers |
|---|---|
| **Logs** | What exactly happened? |
| **Metrics** | What patterns and trends exist in the data? |
| **Traces** | How did different components interact during a specific request? |

### A Real-World Debugging Workflow

Here is how this works in a production system that has proper logging, monitoring, and observability set up.

Let's say you have configured an alert that fires when your error rate goes above 80%. Something triggers it and you receive a Slack notification: *"Something is wrong with your API service."*

From there, the workflow looks like this:

1. **Go to Metrics** — You look at the metrics dashboard and confirm that your error rate is indeed above 80%.
2. **Go to Logs** — From the metrics, you jump directly to the related logs. The tooling will show you all the failed logs associated with that error spike.
3. **Go to Traces** — You click on a specific failed log, and the tool shows you the full trace for that request. It tells you: the request started at this function, traveled to this function, then to this function, and **failed at this particular point**.

Using this entire workflow, you can find out exactly where things went wrong and debug it instantly.

> **This is the whole benefit of implementing logging, monitoring, and observability in your backend systems.**

---

## A Note on the History of Observability

Observability as a practice is fairly modern — it has only been mainstream for the last few years.

Traditionally, about a decade back, the primary form of catching errors or preventing issues at an infrastructure level was mostly dependent on **monitoring practices alone**. The problem with that approach is that monitoring only tells you *that* there is a problem — it does not tell you *what* the problem is or *where* it originated.

With modern observability practices, your system not only informs you that something is wrong, but — provided you have implemented logs, metrics, and traces — it also tells you **exactly what is wrong and why**.

---

## Logging in Practice

### Log Levels

When logging a particular event, you assign a **level** to that log. Log levels help you filter and configure which events are recorded depending on the environment. The most common levels are:

#### `DEBUG`

Used during development when you are troubleshooting and need as many details about system behavior as possible. Debug logs can be overwhelming, which is why they are typically **disabled in production**. In local development, you want debug logs on so that you have the most granular information available.

#### `INFO`

Used for general application operations and business events. For example, in a to-do application, when a new to-do is created, you use `log.info` to record that event. Any kind of **successful operation** can use the info level.

#### `WARN`

Used for events that fall between info and error — not a successful operation, but also not critical enough to be classified as an error. For example, if a user's authentication attempt fails because they entered the wrong password, you log that as a warning. This is not a system error — it is the user's mistake — so `warn` is appropriate.

#### `ERROR`

Used for any kind of error — validation errors, failed database queries, and so on. This is one of the most common and important log levels. It is one of the primary reasons we implement logging in the first place.

#### `FATAL`

The most serious level. Once something is logged as fatal, your application mostly **stops and restarts** depending on your infrastructure configuration. Fatal means the issue is severe enough that the application is shutting down. Use this sparingly for truly catastrophic, unrecoverable failures.

---

### Structured vs Unstructured Logging

There are two kinds of log formats: **structured** and **unstructured**. When to use each depends on your environment.

#### Unstructured Logging (Development)

Unstructured logs are plain text, human-readable logs. When running your backend locally in a development environment, you want logs that are easy to read — ideally with colors, clear formatting, and plain English. This makes it easier to:

- Spot issues quickly
- Understand what is happening at a glance
- Fix problems efficiently

This is why during development we keep logs readable and human-friendly.

#### Structured Logging — JSON (Production)

In production, we log in **JSON format**. Instead of printing human-readable text, each log event is a JSON object containing all the relevant parameters — the status, the message, the user ID, the request ID, etc.

The reason we use JSON in production is that all these logs are being **parsed by log management tools** — things like the ELK stack, or the open-source Grafana stack (Loki, Promtail, Grafana). If logs are in plain text, these tools have a hard time extracting valuable parameters like user IDs or request IDs from unstructured strings.

JSON makes it easy for tools to parse logs and extract valuable, actionable information.

> **Summary:** In development, use unstructured (plain text) logging for readability. In production, use structured (JSON) logging for tool compatibility and easy parsing.

---

## Instrumentation and OpenTelemetry

Two terms you will hear frequently in the context of observability are **instrumentation** and **OpenTelemetry**.

### Instrumentation

Instrumentation is the practice of **measuring different attributes of your functions and components**. This is directly related to what it means for a system to be observable — you are inserting measurement logic into your application so that traces, metrics, and logs can be generated as requests flow through the system.

### OpenTelemetry

OpenTelemetry is a **standard** and a recent addition to the observability ecosystem. It provides a complete ecosystem of tools, SDKs, APIs, and best practices so that you can properly instrument your applications — regardless of which language you are using. Whether it is Node.js, Go, Python, or any other major language, the community has built enough resources and SDKs to support it.

OpenTelemetry is an **open standard**, which means even if you are using a proprietary observability tool like New Relic, you can still integrate an OpenTelemetry collector to have more control over how your requests and components are instrumented.

---

## A Real-World Code Walkthrough

> *Note: The goal here is not to memorize the code or get overwhelmed by it. The point is to see how these logging, monitoring, and observability practices actually look in a real application.*

The example is a to-do application with a backend written in Go. The application uses **New Relic** as its observability solution.

### Logger Initialization

The application creates a new logger before the application starts. Two important configurations are set up:

**1. Log Level Selection**

A function called `getLogLevel` checks whether the application is running in development mode (local) or production mode (deployed infrastructure). Based on that:

- **Development:** Returns `debug` level — more verbose, easier to trace issues locally
- **Production:** Returns `info` level — filters out debug noise, keeps logs meaningful

**2. Log Format Selection**

- **Development:** Logs are formatted as plain console output (human-readable)
- **Production:** Logs are formatted as JSON (machine-parseable)

If the format is changed to JSON and the environment is set to production, all logs output as structured JSON — great for tools, harder for humans. Switching back to development mode returns the readable, color-formatted console output.

### Monitoring Middleware

For monitoring to work, a middleware called the **New Relic middleware** is set up on the router. This middleware wraps the entire application. Every time a new request arrives, it **instruments the whole request** — recording the request's attributes and associating them with a transaction.

### Tracing in the Service Layer

Here is how tracing works in practice using the `createTodo` service function as an example.

**Step 1 — Extract the transaction from context**

A separate middleware called the **enhanced tracing middleware** runs at the very beginning of each request. This middleware:

- Creates a new transaction using New Relic
- Adds parameters to the transaction: service name, environment (local or production), IP address, user agent, request ID, user ID, user email, tenant ID — all the context needed for debugging
- Stores this transaction inside the request **context**

By the time the request reaches the `createTodo` service method, the transaction is already in the context and ready to be used.

**Step 2 — Use the transaction in the service**

The `createTodo` function:

1. Pulls the transaction out of the context
2. Defers ending the segment when the function returns (so the trace records the full duration of this service call)
3. Adds additional attributes to the transaction — the user ID and the title of the to-do being created
4. Logs an `info` level message that a `createTodo` operation is being performed with the given title
5. If a priority is passed in the payload, it is added as an attribute to the transaction and also logged
6. Logs another `info` level message that the process of creating a new to-do is being initiated
7. Executes the database operation

**Step 3 — Handle success and failure**

- **On error:** Logs the error at the `error` level, adds the error to the trace, and marks the operation result as `error` in the transaction attributes
- **On success:** Adds the operation and the newly created to-do's ID as attributes to the transaction, and logs a `debug` level message (which will not appear in production logs)

Finally, a business event log is recorded at the `info` level — capturing the operation name (`todo_created`) and all associated metadata: the to-do ID, title, category ID, and priority.

---

## Observability Tools

There are two main routes for setting up logging, monitoring, and observability: the **open-source route** and the **proprietary route**.

### Open-Source Stack (Grafana Stack)

Used in most enterprises. The components are:

- **Prometheus** — Collects and stores metrics
- **Grafana** — The dashboard and visualization layer for displaying metrics
- **Loki** — Log aggregation system
- **Promtail** — Agent that ships logs to Loki
- **Jaeger** — Distributed tracing system

This stack gives you full control and is highly customizable, but it requires a team with the resources and experience to configure, maintain, and operate all these individual open-source tools.

### Proprietary Solutions

If you do not have the team size or the resources to maintain the open-source stack, proprietary all-in-one solutions make more sense. The most common ones are:

- **New Relic** — A complete solution for logging, monitoring, and observability. Provides a unified dashboard for metrics, logs, and traces.
- **Datadog** — Another popular all-in-one observability platform

These tools simplify the setup significantly at the cost of vendor lock-in and licensing fees.

---

## The New Relic Dashboard — What Observability Looks Like in Practice

Here is what the workflow looks like when using New Relic as an example.

### Triggering Errors

After deploying the application and firing a few API requests without providing a valid token, the backend returns `401 Unauthorized` errors. These errors are captured and visible in the New Relic dashboard.

### Viewing Metrics

From the **Summary** section in New Relic, you can see:

- **Average transaction time** — How long requests take on average
- **Throughput** — How many requests are being processed
- **Error rate** — Percentage of failed requests

These are the metrics — actual numbers that quantify the state of the system.

### Jumping from Metrics to Logs

From the error rate metric, you can drill into the associated logs. Clicking on a specific log entry (e.g., an unauthorized error) reveals all the associated metadata captured at the time of the request:

- Application name
- Environment
- Error code (e.g., `401 Unauthorized`)
- Host name and IP address
- Log level
- Message content
- HTTP method (e.g., `GET`)
- API route (e.g., `/todos`)
- Span ID
- Timestamp

### Jumping from Logs to Traces

Each log is also connected to a **trace**. Clicking through from the log to the trace gives you the full transaction view — showing every component the request passed through and where it ultimately failed or succeeded.

### System-Level Monitoring

By clicking on the **Go runtime** section (since the backend is written in Go), you can see system-level monitoring data:

- Garbage collection time
- Memory usage (e.g., only 3 MB)
- Throughput
- Average response time

This is how **all three pillars — logs, metrics, and traces — work together** to give you a complete, debuggable picture of your system.

---

## Implementation is a Collective Effort

It is important to understand that logging, monitoring, and observability is not just a developer responsibility. It is a **collective effort**:

- **Developers** are responsible for implementing logging, tracing, and metrics in the application code — adding the right log levels, instrumenting service functions, creating transactions, and attaching relevant attributes
- **DevOps / Infrastructure engineers** are responsible for setting up and maintaining the correct collection pipelines, configuring alerting rules, and ensuring that all metrics, logs, and traces are properly gathered and stored in whatever tool is being used

Both sides have to do their part for the full workflow to function.

---

## Summary

Logging, monitoring, and observability is a spectrum — not a binary state. No system is 100% observable; every team implements these practices to the degree that their resources and priorities allow.

| Practice | What it tells you | Key output |
|---|---|---|
| **Logging** | What exactly happened | Events with metadata |
| **Monitoring** | What the system's health looks like | Near-real-time state |
| **Observability** | Where exactly things went wrong | Logs + Metrics + Traces |

The modern observability workflow — getting an alert, jumping to metrics, drilling into logs, and following a trace — is the most efficient way to debug production issues. The open-source Grafana stack and proprietary tools like New Relic and Datadog both provide this capability in different ways.

What matters most is that:

- You **implement it in your code** — log important events at the right levels, instrument your service functions, attach useful context to your transactions
- You **pick tooling that matches your team's capabilities** — open-source for teams with the resources to maintain it, proprietary for everyone else
- You treat it as a **first-class concern**, not an afterthought