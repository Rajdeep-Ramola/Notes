# Validations and Transformations in Backend APIs

## Introduction

This topic is about keeping a set of rules and guidelines in mind while designing APIs. These concepts are mostly related to:

- **Data integrity**
- **Security**

That is where **validations** and **transformations** come into play.

To understand them properly, we first need to understand where they fit inside a typical backend architecture.

---

# Typical Backend Architecture

In a standard backend application, execution is usually divided into multiple layers.

## 1. Repository Layer

The bottom-most layer is generally called the **Repository Layer**.

### Responsibilities

This layer deals with:

- Database connections
- Query execution
- Insertions
- Deletions
- Data persistence

The database can be:

- Relational databases (PostgreSQL, MySQL, etc.)
- Redis
- Any other persistent storage system

### Example Responsibilities

- Running SQL queries
- Creating rows/documents
- Updating records
- Fetching data

---

## 2. Service Layer

Above the repository layer, we have the **Service Layer**.

### Responsibilities

This layer contains the **business logic** of the application.

A service method may:

- Call one or more repository methods
- Send notifications
- Send emails
- Trigger webhooks
- Process data
- Execute workflows

### Important Point

The service layer defines the actual functionality of an API.

A typical service method interacts with the repository layer to perform database operations.

---

## 3. Controller Layer

Above the service layer, we have the **Controller Layer**.

### Responsibilities

The controller:

- Receives the HTTP request
- Calls the appropriate service method
- Returns the response back to the client

### Why Separate Controller and Service Layers?

We separate them because we want to isolate:

- HTTP-related logic
- Business logic

### HTTP-Related Logic Includes

- Status codes
- Request validation
- Response formatting
- Error handling

### Flow of an API Request

```text
Client Request
      ↓
Controller Layer
      ↓
Service Layer
      ↓
Repository Layer
      ↓
Database
```

The response then travels back upward to the client.

---

# Where Do Validations and Transformations Happen?

Validations and transformations happen at the entry point of the server.

More specifically:

- After route matching
- Before business logic execution

## Execution Flow

```text
Incoming Request
      ↓
Route Matching
      ↓
Validation & Transformation Pipeline
      ↓
Controller
      ↓
Service
      ↓
Repository
```

This is the critical point where the server checks whether incoming data is valid.

---

# Why Do We Need Validations?

Imagine clients from all over the world sending API requests.

They can send:

- JSON payloads
- Query parameters
- Path parameters
- Headers

Before this data enters the application logic, we want to ensure that:

- The structure is correct
- The data types are correct
- The values satisfy our constraints

---

# Example of Validation

Suppose an API expects this payload:

```json
{
  "name": "Harry Potter"
}
```

### Requirements

- `name` field is required
- Must be a string
- Length must be between 5 and 100 characters

---

## Validation Process

The validation pipeline checks:

### 1. Does the field exist?

If `name` is missing:

```json
{
  "error": "name field is required"
}
```

---

### 2. Is the type correct?

If:

```json
{
  "name": 0
}
```

The server returns:

```json
{
  "error": "name must be a string"
}
```

---

### 3. Does it satisfy constraints?

If:

```json
{
  "name": "Hi"
}
```

The server returns:

```json
{
  "error": "name length must be between 5 and 100"
}
```

---

# Why Validation Is Important

Without validation, invalid data can reach the database layer.

---

## Example Without Validation

Suppose the client sends:

```json
{
  "name": 0
}
```

The repository tries to insert it into PostgreSQL.

### Database Schema

```sql
name TEXT NOT NULL
```

Since PostgreSQL expects a string (`TEXT`) but receives a number, the insertion fails.

The client may receive:

```text
500 Internal Server Error
```

This creates:

- Poor user experience
- Unexpected application states
- Potential security problems

---

# Proper Validation Response

Instead of a server error, the API should return:

```text
400 Bad Request
```

Meaning:

> "The data you sent does not satisfy the API constraints."

---

# Validation Demo Examples

---

# Syntactic Validation

## Example Request

POST request:

```text
/api/valid/syntactic
```

### Empty Payload

```json
{}
```

### Response

```json
[
  "email is required",
  "phone is required",
  "date is required"
]
```

---

## Invalid Values

```json
{
  "email": "randomString",
  "phone": 123456,
  "date": "2025-11-05"
}
```

### Response

```json
[
  "invalid email format",
  "phone must be a string"
]
```

---

## Correct Payload

```json
{
  "email": "test@gmail.com",
  "phone": "1234567890",
  "date": "2025-11-05"
}
```

### Response

```text
200 OK
```

---

# Types of Validations

There are many kinds of validations, but these are the most common.

---

# 1. Syntactic Validation

Syntactic validation checks whether data follows a particular structure or syntax.

---

## Examples

### Email Validation

Checks whether the value follows this structure:

```text
name@domain.com
```

---

### Phone Number Validation

Checks:

- Country code
- Number length
- Overall structure

---

### Date Validation

Checks whether the string matches the expected date format.

Example:

```text
YYYY-MM-DD
```

---

# 2. Semantic Validation

Semantic validation checks whether the data makes logical sense.

---

## Example: Date of Birth

Suppose today is:

```text
2025-11-01
```

If the user sends:

```text
2026-01-13
```

That is invalid because a birth date cannot be in the future.

---

## Example: Age

If the user sends:

```json
{
  "age": 365
}
```

It may pass type validation, but semantically it does not make sense.

---

# 3. Type Validation

Type validation checks the actual data types.

Examples:

- String
- Number
- Boolean
- Array
- Object

---

# Semantic Validation Demo

## Empty Payload

Response:

```json
[
  "dateOfBirth is required",
  "age is required"
]
```

---

## Valid Payload

```json
{
  "dateOfBirth": "1995-06-12",
  "age": 43
}
```

### Response

```text
200 OK
```

---

## Invalid Date of Birth

```json
{
  "dateOfBirth": "2026-06-12",
  "age": 43
}
```

### Response

```json
{
  "error": "date of birth cannot be in the future"
}
```

---

## Invalid Age

```json
{
  "dateOfBirth": "1995-06-12",
  "age": 430
}
```

### Response

```json
{
  "error": "age must be less than or equal to 120"
}
```

---

# Complex Validation Example

Suppose an API expects:

- `password`
- `passwordConfirmation`
- `married`

---

## Invalid Payload

```json
{
  "password": "random",
  "passwordConfirmation": "another",
  "married": false
}
```

### Errors

```json
[
  "passwords do not match",
  "password must contain at least 8 characters"
]
```

---

## Fixed Payload

```json
{
  "password": "random123",
  "passwordConfirmation": "random123",
  "married": false
}
```

### Response

```text
200 OK
```

---

## Conditional Validation

If:

```json
{
  "married": true
}
```

Then the API requires another field:

```json
{
  "partner": "John"
}
```

### Error If Missing

```json
{
  "error": "partner name is required when married is true"
}
```

---

# Transformations

Validation checks data.

Transformation modifies data.

---

# What Is Transformation?

Transformation means:

> Converting incoming data into a desired format before business logic execution.

---

# Query Parameter Example

Suppose the API is:

```text
/bookmarks?page=2&limit=20
```

---

## Validation Rules

```text
page:
- must be number
- > 0
- < 500

limit:
- must be number
- > 0
- < 10000
```

---

# Problem

Query parameters always arrive as strings.

The server receives:

```json
{
  "page": "2",
  "limit": "20"
}
```

Validation fails because:

```text
"2" !== 2
```

---

# Solution: Transformation

Before validation:

```text
"2" → 2
"20" → 20
```

This conversion process is called **transformation**.

---

# Validation + Transformation Pipeline

Usually both processes are combined together.

Benefits:

- Centralized input handling
- Cleaner architecture
- Easier debugging
- Easier maintenance

---

# Transformation Demo

## Input Payload

```json
{
  "email": "TeSt@GMAIL.Com",
  "phone": "1234567890",
  "date": "2025-11-05"
}
```

---

## Server Transformations

### Email

Converted to lowercase:

```text
test@gmail.com
```

---

### Phone

Adds `+` prefix:

```text
+1234567890
```

---

### Date

Converted into a standardized format.

---

# Type Validation Demo

The API expects:

- `stringField`
- `numberField`
- `arrayField`
- `booleanField`

---

## Invalid Payload

```json
{
  "stringField": "something",
  "numberField": "10",
  "arrayField": "hello",
  "booleanField": "false"
}
```

### Errors

```json
[
  "numberField must be number",
  "arrayField must be array",
  "booleanField must be boolean"
]
```

---

## Correct Payload

```json
{
  "stringField": "something",
  "numberField": 10,
  "arrayField": ["one", "two"],
  "booleanField": false
}
```

### Response

```text
200 OK
```

---

# Nested Array Validation

The server can even validate array elements.

Example:

```json
{
  "arrayField": [1, 2]
}
```

### Error

```json
{
  "error": "array elements must be strings"
}
```

---

# Frontend Validation vs Backend Validation

This is an extremely important concept.

Many developers mistakenly think frontend validation can replace backend validation.

It cannot.

---

# Purpose of Frontend Validation

Frontend validation exists for:

- User experience
- Immediate feedback

Example:

- Showing errors instantly in forms
- Preventing unnecessary API calls

---

# Purpose of Backend Validation

Backend validation exists for:

- Security
- Data integrity
- Stability

---

# Why Backend Validation Is Mandatory

A backend can have multiple clients:

- Web applications
- Mobile apps
- Postman
- Insomnia
- Third-party integrations

Not all clients perform frontend validation.

Some clients directly hit the API.

If backend validation is missing, invalid data can break the system.

---

# Important Principle

## Frontend validation is optional.

## Backend validation is mandatory.

---

# Frontend + Backend Working Together

## Frontend Validation

Suppose a form contains:

- Email
- Phone
- Date

If the email is invalid:

- The frontend immediately shows an error
- No API request is sent

This improves UX.

---

## Backend Validation

When the frontend sends valid data:

- The backend performs its own validation again
- Then processes the request

This ensures:

- Security
- Data integrity
- Reliability

---

# Final Takeaway

When designing APIs:

- Validate all incoming data
- Transform data into expected formats
- Keep validations strict and explicit
- Never rely solely on frontend validation

## Remember

### Frontend Validation

- UX-focused
- Optional
- Immediate feedback

### Backend Validation

- Security-focused
- Mandatory
- Protects system integrity