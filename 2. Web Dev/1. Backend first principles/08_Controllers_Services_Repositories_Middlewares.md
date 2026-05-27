# Handlers, Services, Repositories, Middleware, and Request Context

# Introduction

In this topic, we are covering three closely related backend concepts:

- Handlers / Controllers
- Services and Repository Pattern
- Middleware and Request Context

Initially, these were supposed to be separate topics, but since they are deeply connected in backend architecture, it makes more sense to explain them together in the same context.

---

# Client and Server Communication

We have:

```text
Client  ⇄  Server
```

The communication happens over HTTP.

When a client sends a request to the server, a lot of processing happens inside the server before the client receives a response.

This entire process is called the:

# Request Lifecycle

We have already discussed the request lifecycle outside the server:

```text
Client → Internet → Server → Response
```

Now we are focusing on:

> What happens inside the server after the request reaches it.

---

# Entry Point of the Request

The moment the operating system forwards an HTTP request to the port where the server is listening, the request enters the server.

Example ports:

- `3000`
- `4000`

The server continuously listens on a specific port.

---

# Routing

After the request enters the server, the first major step is:

# Routing

The routing mechanism matches the incoming request to a route.

Examples:

```text
/users
/users/123
/books
/books/42
```

Depending on the route, the request is forwarded to a particular:

- Handler
- Controller

---

# What Is a Handler or Controller?

A handler (or controller) is a function responsible for processing a specific API request.

---

# Why Separate Handlers, Services, and Repositories?

Technically, everything can be written inside one large handler function.

But that creates problems:

- Hard to maintain
- Hard to scale
- Hard to debug
- Hard to extend

So we separate responsibilities into layers.

This is called a:

# Design Pattern

It is not a strict requirement, but it is considered a best practice for scalable backend systems.

---

# Request Lifecycle Architecture

```text
Client Request
      ↓
Routing
      ↓
Handler / Controller
      ↓
Service Layer
      ↓
Repository Layer
      ↓
Database
```

---

# Handler / Controller Layer

## Responsibilities

The handler receives two important objects:

- Request object
- Response object

These are usually provided automatically by the framework/runtime.

Examples:

- Express.js
- Go HTTP server
- FastAPI
- Django
- Gin
- Fiber

---

# Responsibilities of the Handler

The handler:

- Extracts data from the request
- Validates the request
- Calls the service layer
- Sends the response

---

# Step 1: Extracting Data from Request

Depending on the request type:

## GET Request

Extract:

- Query parameters

Example:

```text
/books?page=1
```

---

## POST / PUT / PATCH Requests

Extract:

- Request body

Example:

```json
{
  "name": "Harry Potter"
}
```

---

## DELETE Request

May extract:

- Path parameters
- Request body

Depending on implementation.

---

# Serialization and Deserialization

When the client sends data, it is usually serialized into JSON.

Example:

```json
{
  "name": "Book"
}
```

JSON is used because it can travel over the internet easily.

---

# Deserialization

The server must convert JSON into native language objects.

Examples:

| Language | Native Format |
|---|---|
| Go | Struct |
| Python | Dictionary / Class |
| Rust | Struct |
| JavaScript | Object |

---

# Binding

This deserialization step is also commonly called:

# Binding

Example:

```text
JSON → Native Struct/Object
```

---

# If Deserialization Fails

The server returns:

```text
400 Bad Request
```

Because the request payload is invalid.

---

# Step 2: Validation and Transformation

After deserialization:

- Validate the data
- Transform the data if necessary

We already discussed validations and transformations in detail previously.

---

# Validation

Validation ensures:

- Required fields exist
- Data types are correct
- Constraints are satisfied
- Data is safe

Example validations:

- Email format
- Required password
- Number ranges
- String length

---

# Transformation

Transformation modifies data before processing.

Examples:

- Lowercasing emails
- Trimming strings
- Adding default values
- Type conversion

---

# Example: Query Parameter Transformation

Suppose the API is:

```text
/books?sort=name
```

Allowed values:

- `name`
- `date`

Suppose query parameters are optional.

If the client sends:

```text
/books
```

Then inside the transformation pipeline:

```text
sort = "date"
```

can be assigned as a default value.

This makes downstream processing simpler.

---

# Why Use Transformations?

Without transformations:

The service layer would need many conditional checks.

Example:

```text
if sort exists:
    use sort
else:
    use default
```

Transformations centralize these operations.

Benefits:

- Cleaner code
- Predictable execution flow
- Easier maintenance

---

# Calling the Service Layer

After:

- Binding
- Validation
- Transformation

The handler now has properly structured data.

The handler passes this data to the:

# Service Layer

---

# Service Layer

The service layer contains:

# Business Logic

The service layer should ideally know nothing about HTTP.

It should not care about:

- Status codes
- Request objects
- Response objects
- Query parameters
- JSON formatting

---

# Good Service Layer Principle

If someone reads a service method alone, they should not immediately know it belongs to an API.

The service layer should simply:

```text
Take data → Process data → Return data
```

---

# Responsibilities of the Service Layer

The service layer may:

- Call repositories
- Send emails
- Trigger notifications
- Call external APIs
- Execute workflows
- Merge multiple database results

---

# Example: Email Service

A service may:

```text
Receive email address
→ Send email
→ Return success
```

Without using a database at all.

---

# Repository Layer

The repository layer handles:

# Database Operations

---

# Responsibilities

The repository:

- Constructs queries
- Inserts data
- Fetches data
- Updates data
- Deletes data

---

# Important Principle

A repository method should have:

# Single Responsibility

---

# Bad Example

One repository method:

```text
getBooks(optionalId)
```

Returning:

- One book sometimes
- All books sometimes

This is bad design.

---

# Good Example

Separate methods:

```text
getBookById(id)
getAllBooks()
```

Each method should do one thing.

---

# Service Layer Orchestration

The service layer can combine multiple repository calls.

Example:

```text
Repository A → User Data
Repository B → Book Data
Repository C → Reviews
```

The service merges all results together.

---

# Response Flow

After processing:

```text
Repository → Service → Handler
```

The handler then:

- Decides response status code
- Formats response
- Sends response

---

# HTTP Response Codes

## Success Codes

Examples:

| Code | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No Content |

---

## Error Codes

Examples:

| Code | Meaning |
|---|---|
| 400 | Bad Request |
| 401 | Unauthorized |
| 500 | Internal Server Error |

---

# Final Request Lifecycle

```text
Client
   ↓
Routing
   ↓
Handler
   ↓
Validation & Transformation
   ↓
Service
   ↓
Repository
   ↓
Database
   ↓
Response
```

---

# Middleware

Now we introduce:

# Middleware

Middleware sits between different stages of the request lifecycle.

---

# Middleware Lifecycle

```text
Request
   ↓
Middleware
   ↓
Middleware
   ↓
Routing
   ↓
Middleware
   ↓
Handler
   ↓
Middleware
   ↓
Response
```

---

# What Is Middleware?

Middleware is a function executed in the middle of request processing.

---

# Middleware Receives

Typically:

- Request object
- Response object
- Next function

---

# What Is `next()`?

`next()` passes execution to the next middleware or next processing stage.

Example:

```text
Middleware A
    ↓ next()
Middleware B
    ↓ next()
Handler
```

---

# Middleware Capabilities

Middleware can:

- Read request data
- Modify request
- Modify response
- Send response directly
- Stop execution
- Forward execution

---

# Why Use Middleware?

Without middleware:

We would duplicate logic across all handlers.

Middleware centralizes common operations.

---

# Common Middleware Examples

---

# 1. CORS Middleware

CORS controls which frontend origins can access the backend.

Example:

```text
https://frontend.com
```

The middleware:

- Checks request origin
- Adds appropriate response headers
- Allows or blocks request

---

# Why Middleware?

Because every request needs CORS handling.

---

# 2. Security Headers Middleware

Adds security headers like:

- Content Security Policy
- X-Frame-Options
- X-Content-Type-Options

---

# 3. Authentication Middleware

Authentication middleware:

- Extracts token
- Verifies token
- Identifies user

---

# Failed Authentication

Returns:

```text
401 Unauthorized
```

Immediately.

---

# Successful Authentication

Stores:

- User ID
- Role
- Permissions

inside request context.

Then calls:

```text
next()
```

---

# 4. Rate Limiting Middleware

Prevents abuse.

Example rule:

```text
30 requests per 2 seconds
```

If exceeded:

```text
429 Too Many Requests
```

---

# 5. Logging Middleware

Logs request details:

- Path
- Method
- Query params
- Body
- IP address

Useful for:

- Debugging
- Auditing
- Monitoring

---

# 6. Global Error Handling Middleware

Very important in production systems.

---

# Purpose

Catch errors from anywhere:

- Middleware
- Handler
- Service
- Repository

And return properly structured responses.

---

# Typical Error Response

```json
{
  "message": "Something went wrong",
  "code": "INTERNAL_ERROR"
}
```

---

# Important Principle

Global error middleware is usually placed:

# At the End

Because errors can happen anywhere upstream.

---

# Middleware Ordering Matters

Correct ordering is extremely important.

Typical order:

```text
CORS
↓
Logging
↓
Authentication
↓
Handlers
↓
Error Handling
```

---

# Request Context

Now we move to:

# Request Context

---

# What Is Request Context?

Request context is:

> A request-scoped shared state.

Each request gets its own context.

---

# Purpose

Allows data sharing between:

- Middleware
- Handlers
- Services

without tightly coupling components.

---

# Context Is Usually

A key-value store.

Example:

```text
{
  userId: 123,
  role: "admin"
}
```

---

# Authentication Example

Authentication middleware verifies token.

Then stores:

```text
userId
role
permissions
```

inside context.

---

# Why Store User ID in Context?

Suppose a client sends:

```json
{
  "userId": 999
}
```

A malicious client could impersonate another user.

Instead:

- Ignore client-provided user ID
- Extract authenticated user ID from context

This is safer.

---

# Example Flow

```text
Authentication Middleware
    ↓
Stores userId in context
    ↓
Handler accesses context
    ↓
Uses authenticated userId
```

---

# Request ID Example

A middleware can generate:

```text
UUID
```

Store it in context.

Example:

```text
x-request-id
```

This helps:

- Trace requests
- Debug distributed systems
- Track microservice calls

---

# Additional Uses of Request Context

Request context can also store:

- Cancellation signals
- Timeouts
- Deadlines
- Correlation IDs

Useful for:

- Preventing hanging requests
- Managing distributed systems
- Microservice communication

---

# Final Summary

---

# Handler / Controller

Responsible for:

- Request parsing
- Validation
- Calling services
- Sending responses

---

# Service Layer

Responsible for:

- Business logic
- Workflow orchestration
- External integrations

---

# Repository Layer

Responsible for:

- Database operations only

---

# Middleware

Responsible for:

- Cross-cutting concerns
- Shared request processing
- Authentication
- Logging
- Security
- Error handling

---

# Request Context

Responsible for:

- Sharing request-scoped data
- Passing metadata safely
- Maintaining loosely coupled architecture

These concepts form the foundation of scalable and maintainable backend systems.