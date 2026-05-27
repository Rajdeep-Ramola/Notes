# Routing in Backend Development

## Introduction to Routing

We learned about different HTTP methods and why they are important in HTTP semantics.

HTTP methods describe the **intent** of a request.

You can think of HTTP methods as expressing the **"what"** of a request:

* What action do you want to perform?
* What is your intention?
* Do you want to:

  * Fetch data?
  * Add data?
  * Update data?
  * Delete data?

Examples:

| HTTP Method | Intent      |
| ----------- | ----------- |
| GET         | Fetch data  |
| POST        | Add data    |
| PATCH / PUT | Update data |
| DELETE      | Remove data |

Routing expresses the **"where"** of a request.

It tells the server:

* Which resource you want to access
* Where you want your request to go
* On which resource you want to perform the action

---

# Understanding Routing with an Example

Suppose we send the following request:

```http
GET /users
```

The server responds with:

```json
[
  {
    "id": 1,
    "name": "John"
  },
  {
    "id": 2,
    "name": "Alice"
  }
]
```

Here:

* `GET` = the action or intention
* `/users` = the route or resource

So the complete meaning becomes:

> "I want to fetch user data from the server."

The server combines:

1. The HTTP method
2. The route/path

and maps them to a specific handler.

That handler performs:

* Business logic
* Database operations
* Authentication
* Validation
* Data fetching

and finally returns a response.

---

# Definition of Routing

Routing is:

> Mapping URL paths and HTTP methods to server-side logic.

The server takes:

* The request method
* The route/path

and maps them to a handler function.

---

# Understanding Routes Through API Requests

## Example 1: GET Request

```http
GET /api/books
```

### Breakdown

| Part       | Meaning            |
| ---------- | ------------------ |
| GET        | Action / intention |
| /api/books | Route / resource   |

The server receives the request and returns book data.

Example response:

```json
[
  {
    "id": 1,
    "title": "Book A"
  },
  {
    "id": 2,
    "title": "Book B"
  }
]
```

---

## Example 2: POST Request

```http
POST /api/books
```

The route is the same:

```http
/api/books
```

But the method changed from `GET` to `POST`.

Now the intention changes.

Instead of fetching books, the request creates a new book.

---

# How the Server Identifies Routes

The server uses:

* Method
* Route

as a combined key.

For example:

| Method | Route      | Meaning     |
| ------ | ---------- | ----------- |
| GET    | /api/books | Fetch books |
| POST   | /api/books | Create book |

Even though the route is identical, the methods differentiate them.

So these become two unique routing paths.

---

# Types of Routing

## 1. Static Routes

The earlier examples:

```http
GET /api/books
POST /api/books
```

are examples of **static routes**.

---

## Why Are They Called Static Routes?

Because the route never changes.

Example:

```http
/api/books
```

This string remains constant.

There are no variable values inside the route.

Characteristics of static routes:

* Constant path
* No dynamic parameters
* Predictable structure
* Usually return the same type of resource

---

# Dynamic Routes

Now consider this request:

```http
GET /api/users/123
```

Here:

* `123` represents the user ID
* The route changes dynamically

This API is used to fetch details of one specific user.

The server extracts the ID from the route and performs operations such as:

* Database lookup
* Fetching user details
* Returning the user data

---

# How Dynamic Route Matching Works

On the server side, routing logic often looks like this:

```js
router.get('/api/users/:id', handler)
```

### Breakdown

| Part       | Meaning                 |
| ---------- | ----------------------- |
| GET        | HTTP method             |
| /api/users | Static portion          |
| :id        | Dynamic route parameter |

The colon (`:`) convention indicates:

> This part of the route is dynamic.

---

# Understanding Dynamic Parameters

When the request:

```http
GET /api/users/123
```

reaches the server:

* `/api` matches
* `/users` matches
* `123` gets inserted into `:id`

So:

```js
id = "123"
```

Even numeric values become strings in route parameters.

---

# Human Readability in REST APIs

One major goal of REST APIs is semantic readability.

The route:

```http
GET /api/users/123
```

can be read naturally as:

> Fetch data of the user whose ID is 123.

This human-readable structure is one of the core ideas behind RESTful API design.

---

# Path Parameters / Route Parameters

Dynamic values inside routes are called:

* Path parameters
* Route parameters

Example:

```http
/api/users/:id
```

Here:

```http
:id
```

is the path parameter.

They are called path parameters because they are part of the URL path itself.

---

# Query Parameters

Now consider this request:

```http
GET /api/search?query=some+value
```

### Breakdown

| Part        | Meaning                       |
| ----------- | ----------------------------- |
| /api/search | Route                         |
| ?           | Beginning of query parameters |
| query       | Key                           |
| some+value  | Value                         |

This section:

```http
?query=some+value
```

is called a **query parameter**.

---

# Why Do We Need Query Parameters?

In `POST` or `PUT` requests, we can send data using the request body.

But in REST APIs:

> GET requests generally do not use a request body.

So if we want to send additional information in a GET request, we use query parameters.

---

# Why Not Use Path Parameters for Everything?

Suppose we tried this:

```http
/api/search/some-value
```

Technically, it works.

But it becomes difficult to maintain and loses semantic clarity.

REST APIs aim to provide meaningful and structured endpoints.

That is why query parameters exist.

---

# Query Parameters as Metadata

Query parameters are commonly used to send:

* Filters
* Search values
* Sorting information
* Pagination data
* Metadata about the request

They are typically represented as key-value pairs.

---

# Pagination Example

Suppose we have this endpoint:

```http
GET /api/books
```

The server returns paginated data.

Example response:

```json
{
  "data": [...],
  "total": 100,
  "currentPage": 1,
  "totalPages": 5,
  "limit": 20
}
```

---

# Understanding Pagination Metadata

### `data`

Contains the current chunk of books.

### `total`

Total number of books available.

### `currentPage`

Current page being viewed.

### `totalPages`

Total number of pages.

### `limit`

Number of items returned per page.

---

# Fetching the Next Page

Suppose the client wants page 2.

The request becomes:

```http
GET /api/books?page=2
```

The server reads the query parameter:

```http
page=2
```

and returns the second page.

---

# Other Common Uses of Query Parameters

Query parameters are frequently used for:

## Filtering

```http
/api/books?category=science
```

## Sorting

```http
/api/books?sort=price
```

## Sort Order

```http
/api/books?order=asc
```

These values help customize the response.

---

# Nested Routes

Nested routing is not exactly a separate routing type.

It is a very common REST API design practice.

Nested routes help express relationships between resources.

---

# Example of Nested Routing

Consider this route:

```http
GET /api/users/123/posts/456
```

### Breakdown

| Segment    | Meaning        |
| ---------- | -------------- |
| /api/users | Users resource |
| /123       | User ID        |
| /posts     | Posts resource |
| /456       | Post ID        |

---

# Understanding Nested Resource Semantics

This route means:

> Fetch post 456 belonging to user 123.

Each level adds more semantic meaning.

---

# Different Levels of Nesting

## Level 1

```http
GET /api/users
```

Meaning:

> Fetch all users.

---

## Level 2

```http
GET /api/users/123
```

Meaning:

> Fetch details of user 123.

---

## Level 3

```http
GET /api/users/123/posts
```

Meaning:

> Fetch all posts of user 123.

---

## Level 4

```http
GET /api/users/123/posts/456
```

Meaning:

> Fetch post 456 of user 123.

---

# Why Nested Routes Are Useful

Nested routes:

* Improve semantic clarity
* Represent relationships naturally
* Help organize APIs
* Improve readability
* Scale well for medium and large APIs

They are extremely common in REST APIs.

---

# Route Versioning and Deprecation

Another important concept is route versioning.

Example routes:

```http
GET /api/v1/products
```

and

```http
GET /api/v2/products
```

---

# Why API Versioning Exists

Suppose version 1 returns:

```json
{
  "data": [
    {
      "id": 1,
      "name": "Laptop",
      "price": 1000
    }
  ]
}
```

Later, requirements change.

Now version 2 returns:

```json
{
  "data": [
    {
      "id": 1,
      "title": "Laptop",
      "price": 1000
    }
  ]
}
```

Notice:

* `name` changed to `title`

Even small response changes can break clients.

---

# Real-World Need for Versioning

Imagine:

* Initially you only served a web application
* Later you also serve:

  * React Native apps
  * Android apps
  * Flutter apps

New clients may require different response structures.

Without versioning, changing the API directly could break older applications.

---

# Benefits of Route Versioning

## 1. Stable Migration Path

Old clients continue using:

```http
/api/v1/products
```

New clients migrate to:

```http
/api/v2/products
```

---

## 2. No Need to Change Entire Route Names

Instead of:

```http
/api/new-products
```

we simply version the API.

---

## 3. Supports Deprecation

You can mark:

```http
v1
```

as deprecated.

Frontend engineers then get time to migrate.

Eventually:

* V1 is removed
* V2 becomes the primary version

---

# API Deprecation Workflow

Typical workflow:

1. Release V2
2. Keep V1 active temporarily
3. Notify client teams
4. Allow migration time
5. Remove deprecated version later

This creates a stable API evolution process.

---

# Catch-All Routes

Now consider this request:

```http
GET /api/v3/products
```

Suppose the server does not support this route.

What should happen?

---

# Purpose of Catch-All Routes

Servers usually implement a fallback route.

Example:

```js
app.get('*', handler)
```

This route catches all unmatched requests.

---

# How Catch-All Routing Works

The server processes routes sequentially.

If no earlier route matches:

* Method mismatch
* Path mismatch
* Unsupported endpoint

then the request reaches the catch-all handler.

---

# Example Response

Instead of returning nothing, the server sends a user-friendly response.

Example:

```json
{
  "message": "Route not found"
}
```

This improves developer experience and debugging.

---

# Summary of Routing Concepts

In this discussion, we covered:

* HTTP methods and intent
* Routing fundamentals
* Static routes
* Dynamic routes
* Path parameters
* Query parameters
* Pagination using query parameters
* Nested routes
* Route versioning
* API deprecation
* Catch-all routes

These are the core routing concepts needed to:

* Understand backend codebases
* Work with REST APIs
* Modify routing logic
* Add new endpoints
* Maintain scalable APIs

Understanding these concepts provides a strong foundation before diving deeper into backend frameworks and server-side architecture.
