# Serialization and Deserialization

# Introduction

Serialization and deserialization are fundamental concepts in backend development and network communication.

To understand them properly, we first need to understand how clients and servers communicate.

---

# Client and Server Communication

Typically, we have:

* A client
* A server

The client is usually the frontend application.

Examples:

* Browser applications
* React apps
* Angular apps
* Vue apps
* Mobile applications

The server may run:

* On localhost
* On a remote machine
* In the cloud
* On AWS
* On GCP
* On Azure

---

# Communication Between Client and Server

Clients and servers communicate through network protocols.

Common communication methods include:

* HTTP
* WebSockets
* gRPC

In this discussion, the focus is on:

> HTTP and REST API style communication.

---

# Example Scenario

Suppose:

* The client is a JavaScript application
* The server is built using Rust

The frontend sends an HTTP request.

A request may look like this:

```http
GET /something HTTP/1.1
Host: domain.com
```

Or for a POST request:

```http
POST /something HTTP/1.1
Host: domain.com
```

Along with:

* Headers
* Request body

This is how the client sends information to the server.

The server processes the request and returns a response.

---

# The Core Problem

The important question is:

> How do different systems understand each other's data?

For example:

* The client is written in JavaScript
* The server is written in Rust

Suppose the JavaScript app sends:

```js
{
  name: "Some String"
}
```

JavaScript and Rust have completely different type systems.

## JavaScript

* Dynamically typed
* Interpreted/runtime-oriented
* Flexible data types

## Rust

* Strictly typed
* Compiled
* Strong type safety

So the question becomes:

> How can data travel across the internet and still be understandable by another machine using a completely different language?

---

# Understanding the Need for a Common Standard

To solve this problem, systems need:

> A common standard format.

Both the client and server agree to:

* Send data in a specific format
* Receive data in the same format
* Convert data to and from that format

This standard must be:

* Language agnostic
* Platform independent
* Easy to parse
* Easy to transmit

---

# High-Level Understanding of the OSI Model

Before discussing serialization deeply, it helps to know about the OSI model.

You do not need expert-level networking knowledge, but a high-level understanding is useful.

---

# What is the OSI Model?

The OSI model explains how data travels over a network.

It consists of multiple layers.

At a high level:

## Top Layer

```text
Application Layer
```

## Bottom Layer

```text
Physical Layer
```

Between them are several intermediate layers.

---

# Same Structure Exists on Both Sides

Both:

* Client
* Server

have networking layers.

Example:

```text
Client:
Application Layer
...
Physical Layer

Server:
Application Layer
...
Physical Layer
```

---

# The Fundamental Networking Problem

Imagine you are asked to design communication between two machines.

Requirements:

* Machines are in different locations
* Connected through the internet
* Possibly written in different languages
* Need to exchange understandable data

A natural solution would be:

> Create a common standard.

---

# The Common Standard Approach

The process works like this:

## Client Side

The client:

1. Takes language-specific data
2. Converts it into a common format
3. Sends it over the network

Example:

```text
JavaScript Object → Common Format
```

---

## Server Side

The server:

1. Receives the common format
2. Converts it into server-native structures
3. Processes the data

Example:

```text
Common Format → Rust Struct
```

---

## Response Flow

When returning a response:

1. Server converts internal data into the common format
2. Sends it across the network
3. Client converts it back into native JavaScript objects

---

# Definition of Serialization and Deserialization

Serialization and deserialization are the processes of:

> Converting data to and from a common format.

This allows systems written in different languages and running in different environments to communicate.

---

# Serialization

Serialization means:

> Converting language-specific data into a standard transferable format.

Example:

```text
JavaScript Object → JSON
```

---

# Deserialization

Deserialization means:

> Converting the standard format back into native language-specific structures.

Example:

```text
JSON → Rust Struct
```

---

# Why Serialization Matters

Serialization is important because it enables:

* Cross-language communication
* Network transmission
* Data storage
* Platform independence
* Standardized APIs

---

# Backend Engineering and Technology Choices

Backend engineering contains hundreds of technologies.

It is impossible to master every technology immediately.

Instead, focus on:

* Commonly used standards
* Widely adopted technologies
* Industry-standard tools

---

# Common Communication Technologies

Examples include:

* HTTP
* WebSockets
* gRPC

HTTP REST APIs are still among the most widely used communication systems.

Understanding HTTP deeply provides a strong backend foundation.

---

# Common Database Technologies

Backend systems also use different databases.

## Relational Databases

Examples:

* PostgreSQL
* MySQL
* SQLite

## Non-Relational Databases

Examples:

* MongoDB
* DynamoDB

For learning purposes, PostgreSQL is often a strong choice because it is widely used in:

* Startups
* Enterprise systems
* Production-grade applications

---

# Serialization Standards

There are multiple serialization standards.

These standards define:

* How data is structured
* How it is transmitted
* How it is parsed

---

# Types of Serialization Standards

Serialization standards are broadly divided into two categories.

---

# 1. Text-Based Serialization Formats

Examples:

* JSON
* YAML
* XML

These formats are human-readable.

---

# 2. Binary Serialization Formats

Examples:

* Protocol Buffers (Protobuf)
* Avro

Binary formats are generally:

* Faster
* More compact
* More efficient

But they are less human-readable.

---

# Most Popular Format for HTTP APIs

For HTTP-based communication, the most widely used serialization format is:

# JSON

JSON is used extensively in REST APIs.

---

# What is JSON?

JSON stands for:

> JavaScript Object Notation

It looks very similar to JavaScript objects.

However:

> JSON is not limited to JavaScript.

It is language independent.

---

# Common Uses of JSON

JSON is used in:

* REST APIs
* Configuration files
* Logging systems
* Application data exchange
* Backend services

---

# Why JSON Became Popular

One major reason is:

> JSON is human-readable.

It is simple and easy to understand.

---

# Structure of a JSON Object

Example JSON:

```json
{
  "name": "John",
  "age": 25,
  "isAdmin": false,
  "skills": ["JavaScript", "Rust"]
}
```

---

# Important JSON Rules

## 1. Objects Use Curly Braces

```json
{
}
```

---

## 2. Keys Must Be Strings

Keys:

* Must be inside double quotes
* Must be strings

Correct:

```json
{
  "name": "John"
}
```

Incorrect:

```json
{
  name: "John"
}
```

---

## 3. Supported Value Types

JSON values can be:

* String
* Number
* Boolean
* Array
* Object
* Null

---

# Nested JSON Objects

JSON supports nesting.

Example:

```json
{
  "name": "John",
  "address": {
    "country": "India",
    "phoneNumber": 3456
  }
}
```

The nested object follows the same JSON rules.

---

# JSON During Client-Server Communication

Consider this request:

```http
POST /api/books
```

Request body:

```json
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert Martin"
}
```

This JSON gets transmitted over the network.

---

# What Happens Internally?

At the application layer:

* Data exists as JSON

Internally, networking layers convert data into:

* Frames
* Packets
* Bits
* Electrical/optical signals

Eventually:

* The server receives the bits
* Networking layers reconstruct the data
* The application layer receives JSON again

---

# Important Mental Model for Backend Engineers

As a backend engineer, your primary concern is:

```text
Application Data ↔ JSON
```

You usually do not need to manage:

* Packet routing
* Frames
* Physical transmission
* Bit-level networking

The networking stack handles those details.

---

# Server Response Example

The server may respond with:

```json
{
  "books": [
    {
      "id": 1,
      "title": "Clean Code",
      "author": "Robert Martin"
    }
  ]
}
```

The client receives this JSON and converts it into JavaScript objects.

Then the frontend renders the UI.

---

# Real-World Serialization Flow

## Client Side

```text
JavaScript Object
        ↓
      JSON
        ↓
Network Transmission
```

---

## Server Side

```text
Network Transmission
        ↓
      JSON
        ↓
Rust Struct / Backend Object
```

---

# Full Serialization Lifecycle

The complete cycle looks like this:

## Step 1

Client creates native objects.

---

## Step 2

Client serializes them into JSON.

---

## Step 3

JSON travels through the network.

---

## Step 4

Server deserializes JSON.

---

## Step 5

Server performs business logic.

---

## Step 6

Server serializes response data into JSON.

---

## Step 7

Client deserializes response JSON.

---

## Step 8

Frontend updates the UI.

---

# Final Summary

Serialization and deserialization are techniques used to:

> Convert data into a common standard format and reconstruct it back into native structures.

This allows:

* Clients and servers
* Different languages
* Different environments
* Different systems

to communicate reliably.

In modern HTTP APIs, the most common serialization format is:

# JSON

Understanding JSON serialization is a foundational backend development skill.
