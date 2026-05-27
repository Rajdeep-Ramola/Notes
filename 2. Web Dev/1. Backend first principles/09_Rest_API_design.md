# REST API Design

## Introduction

API design is one of the most important responsibilities of a backend engineer. A significant amount of backend development time is spent:

* Designing APIs
* Thinking about API architecture
* Standardizing communication patterns
* Structuring payloads and responses
* Handling errors consistently
* Maintaining developer experience

This discussion focuses primarily on **REST APIs**.

Although other API paradigms exist, such as:

* RPC (Remote Procedure Calls)
* GraphQL

…the emphasis here is entirely on REST API design.

---

# Why API Design Is Confusing

Even with years of industry standards and widespread adoption of REST APIs, developers still struggle with common API design questions.

Examples include:

* Should URI path segments be singular or plural?
* When updating data, should we use `PATCH` or `PUT`?
* For custom actions, which HTTP method should be used?
* Which HTTP status code should be returned in different scenarios?
* How should request payloads and responses be structured?

These confusions exist because:

* The internet evolved significantly over time
* The original web architecture was designed in a different era
* Earlier systems were heavily based on **Multi-Page Applications (MPAs)**
* Modern applications are dominated by **Single Page Applications (SPAs)**

---

# Evolution From MPAs to SPAs

## Multi-Page Applications (MPAs)

Older web applications followed the MPA architecture.

Characteristics:

* Every interaction triggered a full page reload
* The server rendered complete HTML pages
* Navigation happened primarily on the server

---

## Single Page Applications (SPAs)

Modern frontend frameworks heavily rely on SPAs.

Characteristics:

* The browser downloads JavaScript once
* Routing happens mostly on the client side
* The frontend becomes heavily client-driven
* APIs are consumed continuously in the background

This architectural shift changed how APIs are designed and consumed.

---

# Purpose of Standardized API Design

The goal is **not** to invent new standards.

The goal is:

* Extract practical guidelines from existing REST standards
* Follow consistent patterns
* Eliminate ambiguity
* Reduce unnecessary decision-making
* Focus more on business logic

Benefits include:

* Easier API integration
* Better maintainability
* Predictable API behavior
* Fewer developer misunderstandings
* Better documentation quality

---

# Topics Covered in API Design

A complete REST API design process includes:

* Designing resources
* Designing routes
* Structuring payloads
* Returning success responses
* Returning error responses
* Choosing status codes
* Versioning APIs
* Naming conventions
* Request validation
* Documentation patterns

The overall objective is to establish a repeatable and scalable API design workflow.

---

# Historical Background of REST APIs

## The Origin of the Web

In 1990, **Tim Berners-Lee** started a project called the **World Wide Web**.

Initial motivation:

> Share knowledge and information globally.

Within a short period of time, several foundational technologies were invented.

---

# Technologies Invented for the Web

## 1. URI (Uniform Resource Identifier)

URIs identify resources on the internet.

Example:

```txt
https://example.com/books
```

---

## 2. HTTP Protocol

HTTP became the communication protocol between:

* Clients
* Servers

Modern versions include:

* HTTP/1.1
* HTTP/2
* HTTP/3

---

## 3. HTML

HTML became the markup language used to build webpages.

It acts as:

* The structure
* The skeleton of web pages

---

## 4. The First Web Server

A server capable of serving web content.

---

## 5. The First Web Browser

A client capable of accessing and rendering webpages.

---

## 6. The First WYSIWYG HTML Editor

An editor built directly into the browser.

WYSIWYG means:

> What You See Is What You Get

---

# The Scalability Problem of the Web

As the World Wide Web gained popularity:

* User growth became exponential
* The original architecture could not scale efficiently
* New standards and architectural approaches became necessary

The original assumptions about scale were no longer valid.

This led to major architectural reconsiderations.

---

# Roy Fielding and Web Scalability

Around 1993, **Roy Fielding**, co-founder of the Apache HTTP Server Project, became concerned about web scalability.

To solve scalability issues, he proposed several architectural constraints.

These constraints later became the foundation of:

# REST (Representational State Transfer)

---

# REST Architectural Constraints

## 1. Client-Server Constraint

The client and server should have separate responsibilities.

### Client Responsibilities

* User interface
* User experience
* Presentation layer

### Server Responsibilities

* Business logic
* Data storage
* Processing requests

This separation allows:

* Independent evolution
* Better scalability
* Easier maintenance

---

## 2. Uniform Interface Constraint

The system should follow a standardized communication pattern.

This simplifies architecture and makes APIs predictable.

### Sub-constraints of Uniform Interface

#### a. Resource Identification

Resources should be uniquely identifiable.

Example:

```txt
/books/123
```

---

#### b. Resource Manipulation Through Representations

Resources are manipulated using representations such as:

* JSON
* XML
* HTML

---

#### c. Self-Descriptive Messages

Each request/response should contain enough information to understand itself.

---

#### d. Hypermedia as the Engine of Application State (HATEOAS)

Responses can contain links describing possible next actions.

---

## 3. Layered System Constraint

The architecture should consist of multiple layers.

Each layer interacts only with adjacent layers.

This allows:

* Better scalability
* Better security
* Load balancers
* Proxy servers
* Caching systems

…without affecting core functionality.

---

## 4. Cache Constraint

Server responses should explicitly define whether they are cacheable.

Benefits:

* Reduced server load
* Faster responses
* Improved network efficiency
* Better user experience

---

## 5. Stateless Constraint

Every request must contain all information necessary to process itself.

The server should not remember previous requests.

### Important Characteristics

* No client context stored on the server
* Any server instance can process the request
* Improves scalability and reliability

### Example

Suppose:

* Multiple servers exist behind a load balancer
* Requests are distributed using algorithms like round-robin

Because requests are stateless:

* Any server can process any request
* No session dependency exists between requests

---

## 6. Code-on-Demand Constraint (Optional)

Servers may temporarily extend client functionality by sending executable code.

Example:

* JavaScript sent from server to browser

This constraint is optional.

---

# HTTP/1.1 and REST

Roy Fielding and Tim Berners-Lee worked together to improve web scalability and standardization.

This collaboration contributed to:

# HTTP/1.1

Later, in the year 2000, Roy Fielding formally described the web architectural style in his PhD dissertation.

The dissertation introduced:

# REST — Representational State Transfer

This became the conceptual foundation of modern REST APIs.

---

# Why Is It Called REST?

REST stands for:

# Representational State Transfer

Breaking it down:

---

# 1. Representation

Resources can be represented in different formats.

Common formats include:

* JSON
* XML
* HTML

The same resource may have different representations depending on the client.

---

## Example: User Resource

Suppose a user resource contains:

* ID
* Name
* CreatedAt

### Server-to-Server Communication

Representation may be:

```json
{
  "id": 1,
  "name": "John"
}
```

### Server-to-Browser Communication

Representation may be HTML:

```html
<div>
  <h1>John</h1>
</div>
```

Same resource, different representations.

---

# 2. State

State refers to:

> The current condition or attributes of a resource.

Every resource has a state.

---

## Example: Shopping Cart State

A shopping cart may contain:

* Items
* Quantities
* Total price

This collection of information represents the cart’s current state.

The state is transferred between client and server during API interactions.

---

# 3. Transfer

Transfer refers to:

> Movement of resource representations between client and server.

Communication occurs using HTTP.

Common HTTP methods include:

* GET
* POST
* PUT
* PATCH
* DELETE
* OPTIONS
* HEAD

---

## Example

When a browser requests a webpage using:

```http
GET /home
```

…the server transfers a representation of that resource back to the client.

---

# Combining the Three Concepts

REST describes an architectural style where:

1. Resources have representations
2. Resource state can be transferred
3. Clients and servers communicate through these representations
4. Communication follows scalable architectural constraints

---

# URL Structure

A typical URL structure looks like this:

```txt
https://example.com/books?id=1#reviews
```

---

# Components of a URL

## 1. Scheme

Example:

```txt
https
```

Defines the protocol.

Common values:

* http
* https

---

## 2. Authority / Domain

Example:

```txt
example.com
```

---

## 3. Path / Resource

Example:

```txt
/books
```

Represents the resource being accessed.

---

## 4. Query Parameters

Example:

```txt
?id=1
```

Used for:

* Filters
* Sorting
* Pagination
* Additional metadata

---

## 5. Fragment

Example:

```txt
#reviews
```

Used to navigate to a specific section of a webpage.

---

# REST API URL Structure

An industry-standard API route often looks like this:

```txt
https://api.example.com/v1/books
```

---

# API URL Components

## 1. HTTPS

Secure encrypted communication.

---

## 2. API Subdomain

Example:

```txt
api.example.com
```

A common convention for APIs.

---

## 3. Versioning

Example:

```txt
/v1
```

Used for maintaining backward compatibility.

---

## 4. Resource Path

Example:

```txt
/books
```

Represents the collection resource.

---

# Resource Naming Convention

## Always Use Plural Nouns

Correct:

```txt
/books
```

Incorrect:

```txt
/book
```

Even when accessing a single resource:

Correct:

```txt
/books/123
```

Still plural because the route represents the collection resource.

---

# Slugs in URLs

A slug is a human-readable identifier.

Example book title:

```txt
Harry Potter
```

Converted slug:

```txt
harry-potter
```

---

# Slug Rules

## 1. Convert to Lowercase

Reason:

* Avoid case-sensitivity issues across systems

---

## 2. Replace Spaces with Hyphens

Avoid:

* Spaces
* Underscores

Preferred:

```txt
harry-potter
```

---

# Hierarchical Relationship in URLs

The forward slash (`/`) represents hierarchy.

Example:

```txt
/books/123
```

Interpretation:

* `books` → collection resource
* `123` → specific resource inside the collection

---

# Idempotency in REST APIs

Idempotency means:

> Performing the same operation multiple times has the same effect as performing it once.

The focus is on:

* Server-side side effects
* State changes caused by requests

---

# HTTP Methods and Idempotency

Major HTTP methods:

* GET
* POST
* PUT
* PATCH
* DELETE

---

# GET Method

Purpose:

* Retrieve data

GET is idempotent.

Reason:

* Fetching data multiple times does not change server state

Example:

```http
GET /books
```

Calling this repeatedly only retrieves information.

---

# PATCH Method

Purpose:

* Partially update a resource

Example:

```json
{
  "name": "Updated Name"
}
```

PATCH is idempotent because:

* Applying the same update repeatedly results in the same final state

---

# PUT Method

Purpose:

* Completely replace a resource representation

Example:

Suppose a user object contains:

* ID
* Name
* CreatedAt

A PUT request generally sends the full object.

PUT is idempotent because:

* Replacing a resource with identical data repeatedly produces the same result

---

# PATCH vs PUT

## PATCH

Use when:

* Updating only part of a resource

---

## PUT

Use when:

* Replacing the entire resource representation

---

# Practical Reality

In many real-world APIs:

* PUT and PATCH are often used interchangeably

However, for public APIs:

* Following semantic standards is important
* Consumers expect standard behavior

Incorrect semantics create confusion.

---

# DELETE Method

DELETE is idempotent.

Example:

### First Request

```http
DELETE /users/1
```

Result:

* User gets deleted

---

### Second Request

```http
DELETE /users/1
```

Result:

* Server returns `404 Not Found`
* No additional side effect occurs

Because repeated requests do not continue changing state:

DELETE remains idempotent.

---

# POST Method

POST is non-idempotent.

Purpose:

* Create new resources
* Execute custom actions

---

# Example: Creating Books

Request payload:

```json
{
  "name": "Book Name",
  "description": "Some description"
}
```

### First Request

Creates:

* Book #1

### Second Request

Creates:

* Book #2

Each request causes a new side effect.

Therefore:

# POST is non-idempotent.

---

# Why Duplicate Names May Still Work

Book names are usually not unique.

Databases differentiate records using:

* IDs
* UUIDs
* Auto-increment values

Therefore:

* Multiple books may have identical names
* IDs remain unique

---

# POST for Custom Actions

POST is intentionally open-ended.

It can be used for actions that do not fit standard CRUD semantics.

---

# Example: Send Email API

Example route:

```txt
/send-email
```

Payload:

```json
{
  "target": "user@example.com"
}
```

Question:

Which HTTP method should be used?

This is:

* Not a fetch operation
* Not a resource update
* Not a delete operation

Therefore:

* It becomes a custom action
* POST is appropriate

---

# Designing APIs in Practice

Before writing business logic:

# Design the API interface first.

The API should be:

* Intuitive
* Predictable
* Easy to integrate
* Standardized
* Well-documented

---

# Why Standards Matter

Without standards:

* Consumers must guess API behavior
* Developers may need to read implementation code
* Integration becomes difficult
* Assumptions increase
* Bugs become more likely

With standards:

* Behaviors become predictable
* Documentation becomes easier
* Integration becomes faster
* Confusion decreases

---

# Starting Point for API Design

The best place to start API design is:

# UI/Wireframe Designs

Examples:

* Figma
* Product mockups
* User flows
* User stories

---

# Why Start From the UI?

UI designs reveal:

* How users interact with data
* What entities exist in the system
* Which workflows require APIs

This creates a direct connection between:

* User interactions
* Frontend behavior
* Backend resources
* Database entities

---

# Identifying Resources

Resources are typically:

# Nouns extracted from requirements.

This is a practical rule of thumb.

---

# Example: Project Management Platform

Suppose you are building a system like:

* Jira
* Linear

Possible resources include:

* Projects
* Users
* Organizations
* Tasks
* Tags

These become foundational backend entities.

---

# Typical Backend Workflow

A common workflow:

1. Analyze requirements
2. Identify resources
3. Design database schema
4. Design REST API interface
5. Implement business logic

---

# Transition Toward Database Schema Design

Once resources are identified:

* Database schema design usually comes next
* API design follows schema design

# API Design Workflow and REST API Design Patterns

## Introduction

The API design workflow starts after the database schema design is completed.

Imagine a project management platform with the following database tables:

* **Organization** → stores organizations in the platform
* **Project** → stores projects within organizations
* **Task** → stores tasks inside projects

The goal is to design APIs around these resources.

The workflow generally looks like this:

1. Understand requirements from Figma or product designs
2. Identify resources/nouns in the system
3. Design database schemas and tables
4. Identify client actions (CRUD operations)
5. Design the API interface
6. Implement the API later in the programming language/framework of choice

This section focuses only on **API interface design**, not implementation.

---

# CRUD Operations and Resource Actions

For every resource, clients typically need these operations:

* Create
* Read
* Update
* Delete
* Fetch single resource
* Fetch list of resources

These are commonly called **CRUD operations**.

Example resource:

* Organization

Example actions:

* Create organization
* Get all organizations
* Get single organization
* Update organization
* Delete organization

---

# API Design Tool

The transcript uses **Insomnia**, an alternative to Postman.

Characteristics:

* Lighter than Postman
* Useful for API interface design
* Helps visualize requests and responses

The focus is on:

* API routes
* Request payloads
* Responses
* Query parameters
* API consistency

---

# Designing List Organization API

## Endpoint

```http
GET /organizations
```

Example local URL:

```http
http://localhost:3000/organizations
```

### Notes

* Resource names should be plural
* Path segments should use lowercase
* Versioning is optional

Example with versioning:

```http
/v1/organizations
```

---

# Designing Create Organization API

## Endpoint

```http
POST /organizations
```

## Request Payload

```json
{
  "name": "Org One",
  "status": "active",
  "description": "Some description"
}
```

### Excluded Fields

These are handled by the server/database:

* id
* createdAt
* updatedAt

---

## Successful Response

Typical response:

* Status code: `201 Created`
* Returns newly created entity

Example:

```json
{
  "id": "123",
  "name": "Org One",
  "status": "active",
  "description": "Some description",
  "createdAt": "..."
}
```

---

# List API Response Structure

Example response:

```json
{
  "data": [],
  "total": 0,
  "page": 1,
  "totalPages": 0
}
```

---

# Pagination

## Why Pagination Exists

Returning huge datasets is expensive.

Problems:

* Heavy JSON serialization/deserialization
* Increased server load
* Higher network latency
* Slower frontend rendering
* Poor user experience

Instead of returning everything:

* Return small chunks/pages
* Fetch more data incrementally

---

## Pagination Parameters

### Limit

Controls number of records per response.

Example:

```http
GET /organizations?limit=2
```

### Page

Controls which chunk/page to fetch.

Example:

```http
GET /organizations?page=2&limit=2
```

---

## Typical Paginated Response

```json
{
  "data": [],
  "total": 5,
  "page": 1,
  "totalPages": 3
}
```

### Meaning of Fields

| Field      | Meaning                   |
| ---------- | ------------------------- |
| data       | Current page data         |
| total      | Total records in database |
| page       | Current page number       |
| totalPages | Total pages available     |

---

## Default Pagination Values

If client does not send values:

* Default page → `1`
* Default limit → `10` or `20`

Good APIs provide sensible defaults.

---

# Sorting

Typical list APIs support sorting.

## Query Parameters

```http
GET /organizations?sortBy=name&sortOrder=ascending
```

---

## Default Sorting

If client does not specify sorting:

* Sort by `createdAt`
* Sort order → descending

Why?

Without sorting:

* Databases do not guarantee row order
* Responses become inconsistent

Default sorting ensures stable responses.

---

# Filtering

Filtering narrows results.

## Example

```http
GET /organizations?status=active
```

Another example:

```http
GET /organizations?name=OrgOne
```

---

# Update Organization API

## Endpoint

```http
PATCH /organizations/:id
```

Example:

```http
PATCH /organizations/123
```

---

## Why PATCH Instead of PUT

PATCH is preferred because:

* Usually only partial fields are updated
* Entire entity replacement is uncommon
* PATCH semantically represents partial updates better

PUT and PATCH are often used interchangeably, but PATCH is generally more accurate.

---

## Example Request

```json
{
  "status": "active"
}
```

## Successful Response

* Status code: `200 OK`
* Returns updated entity

---

# Get Single Organization API

## Endpoint

```http
GET /organizations/:id
```

Example:

```http
GET /organizations/123
```

---

# Delete Organization API

## Endpoint

```http
DELETE /organizations/:id
```

---

## Successful Response

* Status code: `204 No Content`
* Empty response body

Meaning:

* Operation succeeded
* No content to return

---

# 404 Rules

## When to Return 404

Return `404 Not Found` when:

* Client requests a specific resource
* Resource does not exist

Example:

```http
GET /organizations/999
```

Response:

```json
{
  "message": "Organization not found"
}
```

---

## List APIs Should NOT Return 404

If list API has no results:

Return:

```json
{
  "data": [],
  "total": 0,
  "page": 1,
  "totalPages": 0
}
```

Status should still be:

```http
200 OK
```

Reason:

* Client requested a list
* Not a specific entity

---

# Custom Actions

Sometimes actions do not fit CRUD semantics.

Example:

* Archive organization

At first glance it seems like an update:

```json
{
  "status": "archived"
}
```

But archiving may also:

* Delete projects
* Send emails
* Notify users
* Trigger workflows
* Archive tasks

Therefore it is treated as a **custom action**.

---

# Designing Custom Action APIs

## Endpoint Structure

```http
POST /organizations/:id/archive
```

### Pattern

```text
/resource/:id/action
```

---

## Why POST?

POST is used because:

* Action is not CRUD
* POST is open-ended in REST semantics

---

## Response Codes for Custom Actions

Custom actions may return:

* `200 OK`
* `201 Created`

Depending on behavior.

Example:

* Clone project creates new entity → `201`
* Archive organization updates entity → `200`

Do not blindly assume every POST returns `201`.

---

# Designing Project APIs

The same patterns apply to projects.

## Create Project

```http
POST /projects
```

### Payload

```json
{
  "name": "Project One",
  "organizationId": "org-id",
  "status": "planned",
  "description": "Some description"
}
```

---

# JSON Naming Conventions

JSON fields should use:

* camelCase

Example:

```json
{
  "organizationId": "123"
}
```

Avoid:

```json
{
  "organization_id": "123"
}
```

if the rest of the API uses camelCase.

Consistency matters.

---

# API Consistency Principles

Across all APIs:

* Use consistent field names
* Use consistent route styles
* Use consistent query parameters
* Use consistent response structures

---

## Bad Example

```json
{
  "description": "..."
}
```

in one API and:

```json
{
  "dsc": "..."
}
```

in another.

This creates confusion for API consumers.

---

# Project CRUD Endpoints

## Create

```http
POST /projects
```

## List

```http
GET /projects
```

## Get Single

```http
GET /projects/:id
```

## Update

```http
PATCH /projects/:id
```

## Delete

```http
DELETE /projects/:id
```

---

# Clone Project Custom Action

## Endpoint

```http
POST /projects/:id/clone
```

Reason it is custom action:

* May clone tasks
* May send emails
* May trigger workflows
* More than simple create operation

---

# Important API Design Principles

## 1. Provide Interactive Documentation

Use tools like:

* Swagger
* OpenAPI

Benefits:

* API testing
* Interactive playground
* Better developer experience
* Easier integrations

---

## 2. Keep APIs Intuitive and Consistent

Consistency should exist across:

* Routes
* Dynamic parameters
* Payloads
* Responses
* Query parameters
* Naming conventions

---

## 3. Provide Sensible Defaults

Examples:

* Default page → 1
* Default limit → 10
* Default sortBy → createdAt
* Default sortOrder → descending

Also:

If organization status is not provided:

```json
{
  "status": "active"
}
```

can be set automatically by the server.

---

## 4. Avoid Abbreviations

Avoid:

```json
{
  "dsc": "..."
}
```

Prefer:

```json
{
  "description": "..."
}
```

Readable APIs reduce confusion.

---

# Final Thoughts

Good API design is not just about implementation.

Before coding:

* Design the interface
* Think about consumers
* Define standards
* Maintain consistency
* Improve developer experience

API design should be treated as a dedicated phase.

Useful tools:

* Swagger/OpenAPI
* Insomnia
* Postman

These tools help visualize and improve API usability before implementation begins.

The principles remain the same regardless of programming language:

* Go
* Node.js
* Java
* Python
* Any backend stack

Strong API design leads to:

* Better maintainability
* Easier integrations
* Better developer experience
* More intuitive systems
